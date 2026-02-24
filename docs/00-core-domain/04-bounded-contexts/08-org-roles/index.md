---
title: "08. Org & Roles — Teams, Staff & Delegation"
description: "P0 bounded context: team structure, staff roles, delegation, and permission model for B2B flows (clinics, schools, businesses)."
---

# 08. Org & Roles — Teams, Staff & Delegation

> **Status: P0 — Required for B2B onboarding (clinics, schools, businesses with staff).**

---

## Responsibility

The Org & Roles bounded context defines how organizations and their staff operate in VyteMerge.

It answers:
> **Who can do what on behalf of this organization or provider?**

Org & Roles does NOT:
- Manage the social identity of staff (that is `Social/Profiles`)
- Enforce permissions at query time (that is `Access`)
- Manage formal cross-organization Agreements (that is `Foundation/Agreements`)

It defines the **internal permission structure** of a Provider organization.

---

## Context: Why this exists

A solo provider (Dr. X) owns their own Listings and Timeline directly. But a business (Clinic ABC) needs:
- **Staff**: employees who perform services but don't own the commercial configuration.
- **Roles**: different staff have different capabilities (receptionist vs. doctor vs. owner).
- **Delegation**: the business owner delegates specific actions to specific people.
- **Assignment**: bookings are auto-assigned to the right staff member.

---

## Key Entities

### Team
A group of Profiles operating under an organizational Profile.

| Field | Description |
|---|---|
| `teamId` | Unique identifier |
| `organizationProfileId` | The organizational Profile that owns this team |
| `name` | Team display name |
| `description` | Optional description |
| `createdAt` | ISO8601 |

### TeamMember
A Profile's membership in a Team with a specific role.

| Field | Description |
|---|---|
| `teamMemberId` | Unique identifier |
| `teamId` | The team |
| `profileId` | The member's Profile |
| `role` | `owner \| admin \| staff \| viewer` |
| `permissions` | Fine-grained permissions (see Permission model below) |
| `status` | `active \| invited \| suspended` |
| `joinedAt` | ISO8601 |

### Permission Model

Roles carry default permissions. Admins can grant additional permissions to individual staff.

| Role | Can manage Listings | Can view calendar | Can manage bookings | Can view financials | Can invite staff |
|---|---|---|---|---|---|
| `owner` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `admin` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `staff` | ❌ | own only | own only | ❌ | ❌ |
| `viewer` | ❌ | read-only | ❌ | ❌ | ❌ |

Custom permissions can be granted per-staff member on top of role defaults.

---

## Staff Assignment

When a Booking is created against an organizational Timeline, it needs to be assigned to a staff member.

Two models (see also `core-model.md` Section 12):

**Model A (recommended):** Booking is created against the org's Timeline. An auto-assignment rule selects and assigns a staff member. The Event appears in the staff member's Timeline (via timeline share/delegation).

**Model B:** Booking is created directly against the staff member's Timeline. The org acts as a broker.

The assignment rule can be:
- **Round-robin**: distribute evenly
- **First-available**: assign to whoever has the next open slot
- **Manual**: require a manager to assign explicitly
- **Skills-based**: assign to staff with the matching skill tag

---

## Commands

| Command | Description |
|---|---|
| `CreateTeam` | Creates a Team under an organizational Profile. |
| `InviteTeamMember` | Sends an invitation to a Profile to join a Team with a role. |
| `AcceptTeamInvitation` | Invited Profile joins the Team. |
| `UpdateTeamMemberRole` | Changes a member's role or custom permissions. |
| `RemoveTeamMember` | Removes a member from the Team. |
| `ConfigureAssignmentRule` | Sets the auto-assignment rule for a Timeline. |
| `AssignBooking` | Manually assigns a Booking to a staff member. |

## Events

| Event | Description |
|---|---|
| `TeamCreated` | A Team was created. |
| `TeamMemberInvited` | An invitation was sent. |
| `TeamMemberJoined` | A Profile accepted an invitation. |
| `TeamMemberRoleUpdated` | Role or permissions changed. |
| `TeamMemberRemoved` | A member left or was removed. |
| `BookingAssigned` | A Booking was assigned to a staff Profile. |

---

## Integration with Access

The Access module reads team membership and roles when evaluating whether an actor can perform an action on an organizational resource.

```
AccessRequest(actorId: staffProfileId, resource: orgTimeline, action: Manage)
  → Access checks: is actorId a TeamMember of the org with role ≥ admin?
  → returns: Allowed or Denied
```

---

## Open Questions (P0 items to resolve)

- [ ] Can a staff member belong to multiple teams across different organizations?
- [ ] What happens to staff bookings when a team member is removed? (reassign vs orphan)
- [ ] Does staff have a separate login or share the same Keycloak user?
- [ ] What is the maximum team size? (any platform limits?)
- [ ] Who can configure assignment rules? (owner only? admin?)

---

## References

- Access module: `backend/vt-core/src/Modules/Access/readme.md`
- Agreements (cross-org): `01.Foundation & Governance/3.agreements/index.md`
- Core model (Assignment): `03-core-model/core-model.md` (Section 12)
- Teams module: `backend/vt-core/src/Modules/Teams/`
