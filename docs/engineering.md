# Engineering Navora

Navora combines trip planning, changing schedules, and shared travel data in a native Swift app. These selected decisions explain the engineering work behind the product without exposing its source code.

[Back to the showcase](../README.md)

## Keeping travel dates tied to the destination

A travel day should follow the destination's calendar even when the traveler plans it from another timezone. Using the device's timezone everywhere can shift day boundaries and make the wrong day appear current.

Navora stores a destination timezone with each trip and uses its calendar for trip dates and current-day calculations. Older records or invalid timezone values fall back to the device timezone. That fallback keeps those records usable, but it is less precise than a valid destination timezone.

## Changing a plan without silently overwriting later edits

Replacing a closed attraction can affect several activities or even later days. Meanwhile, another traveler may edit the shared itinerary.

For the closed-attraction flow, Rescue My Day builds a proposal from a snapshot of the schedule. The traveler can inspect and adjust the proposed changes. Before applying them, the app checks editing permissions and whether the affected schedule still matches the proposal's starting state.

Changes are saved together, with rollback if saving fails. A local undo record supports reversal while checking for conflicting later edits. The undo record stays on the device; it is not synchronized between travelers.

The **Free time** flow is intentionally smaller: the traveler selects a suggested place and adds it directly. It does not use the same proposal and undo workflow.

## Separating personal and shared trip data

Sharing introduces ownership, invitations, editing permissions, and remote changes. A shared trip must be written to the correct store and respect the participant's access.

Navora uses SwiftData for app-facing models, with Core Data and CloudKit handling personal and shared storage underneath. Shared-trip editing uses a separate data context, and a synchronization bridge brings remote persistence changes back into the app's contexts.

This preserves SwiftData's integration with the interface while supporting CloudKit sharing. The tradeoff is additional coordination between persistence layers and careful handling of remote changes.

## Connecting generated plans to places

AI planning combines Apple Foundation Models with MapKit place discovery. It uses travel pace, interests, notes, and language to produce a day-by-day plan associated with discovered places.

Generated data is assembled before being mapped into the persisted itinerary. Successful generation currently replaces the activities on matching generated days; it is a different workflow from Rescue's explicit review-and-apply step.

Manual planning remains available when AI is unavailable. Model availability is a capability the interface must account for, rather than a requirement for creating a trip.

## Keeping shared expenses understandable

Trip expenses support equal splits, exact amounts, percentages, and reimbursement cases. Participant selection also needs to account for travelers and collaborators without counting the same person twice.

Navora keeps totals and balances separate by currency. It does not convert currencies or process payments. This avoids presenting a combined total whose exchange-rate assumptions would be unclear.

## Testing focus

The private project includes test suites for trip timezones, Rescue proposals, expense calculations, collaboration identity, and notification routing. These areas cover the boundaries where travel dates, shared state, and changing plans interact.
