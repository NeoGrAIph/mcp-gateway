# Permission model pointers

Status: active | Audience: developer | Last reviewed: 2026-02-09

## Goal

Know where the source-of-truth is for Entra roles and gateway access checks.

## Preconditions

- You have an Entra ID application registration used by the gateway.

## Procedure

- Configure Entra App Roles and assign them to identities.
- When creating adapters/tools, set `requiredRoles` (if supported by the API version you deploy).

## Verification

- A caller without required roles cannot write/update resources (expected `401/403` depending on deployment).

## References

- Entra roles how-to: [`docs/entra-app-roles.md`](../../entra-app-roles.md)
