# Functional Requirements

## Authentication
- The system shall allow users to sign in using Microsoft Azure AD B2C.
- The system shall allow users to sign out and clear the authenticated session.
- The system shall reflect the current authentication state in the UI (e.g., show/hide features requiring sign-in).

## Shifts Management
- The system shall allow users to create, view, edit, and delete shifts.
- The system shall display shifts in daily and weekly schedule views.
- The system shall validate and normalize shift data before saving.
- The system shall calculate and store shift durations.

## Shift Templates
- The system shall allow users to create, view, edit, and delete shift templates.
- The system shall allow users to apply a template to prefill a new shift.

## Work Information
- The system shall group work profiles under a workplace, with an optional position and location for each profile.
- Each profile shall support named fixed hourly rates, such as Standard, Overnight, Weekend, and Holiday. Different names may have the same amount.
- Users shall select a profile and rate when entering a shift, or enter a custom hourly amount. Rate names do not automatically apply date, holiday, or overnight rules.
- The Work details editor shall support creating, editing, and removing profiles and rates without changing the workplace name.
- Saved shifts and templates shall retain their selected position, location, rate name, and hourly amount. Editing or deleting work information shall not recalculate existing records.
- Existing numeric rates shall remain available without assigning an inferred rate category. Local storage, backups, import, and account sync shall preserve complete profiles, including profiles without rates.

## Shift Recurrence
- The system shall allow users to create shifts that recur on a schedule (e.g., daily or weekly).
- The system shall expand recurrence into concrete shift instances within a selected date range.
- The system shall allow editing or skipping a single occurrence without changing the entire series.

## Persistence and Sync
- When the user is not authenticated, the system shall persist data locally using browser storage.
- When the user is authenticated, the system shall synchronize data with the backend API.
