# Listing Domain Model — vt-core Backend

## Entity Relationship Diagram

```mermaid
erDiagram
    Listing {
        UUID id PK
        UUID profile_id FK "NOT NULL"
        UUID service_id FK "NULLABLE"
        UUID place_id FK "NULLABLE"
        UUID shared_capacity_group_id FK "NULLABLE"
        VARCHAR_200 title "NOT NULL"
        TEXT description "NOT NULL"
        TEXT location "NULLABLE"
        INT status "NOT NULL (enum)"
        INT meeting_method_type "NOT NULL (enum)"
        TEXT time_zone_id "NOT NULL"
        INT capacity "NOT NULL DEFAULT 1"
        TIMESTAMP starts_at_utc "NULLABLE"
        TIMESTAMP ends_at_utc "NULLABLE"
        TIMESTAMP updated_at "NOT NULL"
        NUMERIC_18_4 price "NOT NULL"
        VARCHAR_10 currency "NOT NULL DEFAULT USD"
        INT confirmation_policy "NOT NULL (enum)"
        INT visibility "NOT NULL (enum)"
        INT slot_config_duration_min "NOT NULL"
        INT slot_config_pre_buffer_min "NULLABLE"
        INT slot_config_post_buffer_min "NULLABLE"
        INT slot_config_bw_min_notice_hours "NOT NULL"
        INT slot_config_bw_max_advance_days "NOT NULL"
        JSONB tags "NOT NULL DEFAULT []"
        JSONB intake_form "NOT NULL DEFAULT []"
        JSONB add_ons "NOT NULL DEFAULT []"
        JSONB recurrence "NOT NULL"
    }

    ListingMedia {
        UUID id PK
        UUID listing_id FK "NOT NULL"
        TEXT url "NOT NULL (Uri)"
        TEXT caption "NULLABLE"
        BOOL is_cover "NOT NULL DEFAULT false"
        INT order "NOT NULL 1-based"
    }

    SharedCapacityGroup {
        UUID id PK
        UUID owner_id FK "NOT NULL"
        INT capacity_effective "NOT NULL"
        TIMESTAMP created_at_utc "NOT NULL"
        TIMESTAMP updated_at_utc "NOT NULL"
    }

    SharedCapacityGroupMember {
        UUID group_id PK "NOT NULL"
        UUID listing_id PK "NOT NULL UNIQUE"
        TIMESTAMP added_at_utc "NOT NULL"
    }

    Profile {
        UUID id PK
    }

    Service {
        UUID id PK
        STRING status "must be Active to publish"
    }

    Place {
        UUID id PK
        STRING name "must be set to publish"
    }

    Listing ||--o{ ListingMedia : "1:N max 5"
    Listing }o--o| SharedCapacityGroup : "N:1 optional"
    SharedCapacityGroup ||--o{ SharedCapacityGroupMember : "1:N"
    SharedCapacityGroupMember }o--|| Listing : "FK listing_id"
    Listing }o--|| Profile : "FK profile_id"
    Listing }o--o| Service : "FK service_id"
    Listing }o--o| Place : "FK place_id"
```

## JSONB Structures

```mermaid
erDiagram
    WeeklyRecurrence {
        STRING type "Weekly or WeeklyEveryNWeeks"
        INT interval_in_weeks "DEFAULT 1"
    }

    WeeklyDaySchedule {
        ENUM day_of_week "0-6 Mon-Sun"
    }

    TimeRange {
        TimeOnly start "e.g. 09:00"
        TimeOnly end "e.g. 13:00"
    }

    WeeklyExceptionDate {
        DateOnly date "specific date"
        BOOL is_day_disabled "override or disable"
    }

    SpecificDatesRecurrence {
        STRING type "SpecificDates"
    }

    SpecificDateSchedule {
        DateOnly date "specific date"
    }

    FieldDefinition {
        STRING label "NOT NULL"
        ENUM type "Text or Select or Bool"
        BOOL required "NOT NULL"
        STRING_ARRAY options "NULLABLE only for Select"
    }

    AddOn {
        STRING name "NOT NULL"
        DECIMAL price "NOT NULL"
        BOOL is_optional "NOT NULL"
    }

    WeeklyRecurrence ||--o{ WeeklyDaySchedule : "days[]"
    WeeklyDaySchedule ||--o{ TimeRange : "time_ranges[]"
    WeeklyRecurrence ||--o{ WeeklyExceptionDate : "exceptions[]"
    WeeklyExceptionDate ||--o{ TimeRange : "override_ranges[]"
    SpecificDatesRecurrence ||--o{ SpecificDateSchedule : "dates[]"
    SpecificDateSchedule ||--o{ TimeRange : "time_ranges[]"
```

## Enums

```mermaid
graph LR
    subgraph ListingStatus
        LS0["0 = Draft"]
        LS1["1 = Published"]
        LS2["2 = Unpublished"]
        LS3["3 = Archived"]
    end

    subgraph MeetingMethod
        MM0["0 = None"]
        MM1["1 = GoogleCall"]
        MM2["2 = PhoneCall"]
        MM3["3 = OnOwnerLocation"]
        MM4["4 = OnAttendeeLocation"]
    end

    subgraph ConfirmationPolicy
        CP0["0 = AutoConfirm"]
        CP1["1 = ManualConfirm"]
        CP2["2 = RequestOnly"]
    end

    subgraph Visibility
        V0["0 = Public"]
        V1["1 = Private"]
    end
```

## Status Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Draft : CreateDraft
    Draft --> Published : Publish (validates all fields)
    Published --> Unpublished : Unpublish
    Unpublished --> Published : Publish
    Draft --> Archived : Archive
    Published --> Archived : Archive
    Unpublished --> Archived : Archive
    Archived --> Draft : Unarchive
```

## Publish Validation Rules

To transition from Draft/Unpublished → Published, ALL must be true:
- `title` non-empty
- `description` non-empty
- `place_id` is set
- `price` >= 0
- `slot_config.duration_min` > 0
- at least one `ListingOption` exists and validates for publish
- linked `Service` (if service_id set) must have status = Active

> Channels (audiencias comunidad-style) fueron diferidos post-v1 — ver [ADR-0007](../../../private/decisions/ADR-0007-channels-deferred-from-v1.md).

## Default Weekly Schedule (on creation)

Monday–Friday, two blocks per day:
- 09:00 – 13:00
- 16:00 – 20:00
