---
title: Classes
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/classes/page.tsx", "components/classes/ClassGrid.tsx", "components/calendar/ClassCalendar.tsx", "components/calendar/StripCalendar.tsx", "components/calendar/MonthCalendar.tsx", "lib/data/mock_data.ts"]
updated: 2026-06-01
---

> **Status:** 🟡 Mock data — class list from `lib/data/mock_data.ts`; calendar dates are hardcoded

# Classes

**Route:** `/classes`  
**File:** `app/(dashboard)/classes/page.tsx`  
Server component. Class scheduling and management page.

## Layout

```
Header: "Classes" / "Manage class schedules and bookings"
"Schedule Class" button (non-functional)
ClassCalendar (collapsed week strip by default)
ClassGrid (list of class cards)
```

## ClassCalendar

**File:** `components/calendar/ClassCalendar.tsx`  
Client component. Toggles between two views via a `ChevronDown` arrow button.

### Collapsed — StripCalendar

**File:** `components/calendar/StripCalendar.tsx`  
A horizontal week strip showing Mon–Sun with dates 20–26. **Fully hardcoded.** Wednesday (index 2) is highlighted with `bg-gym-green text-black` and a green glow. The header shows "December 2025" — hardcoded, not date-driven. See [[reference/Conventions & Gotchas]].

### Expanded — MonthCalendar

**File:** `components/calendar/MonthCalendar.tsx`  
Uses the `react-calendar` (v6) library. Local `value` state defaults to `new Date()`. Custom nav labels (`‹`/`›`), short weekday names via `Intl.DateTimeFormat`. Styled by `components/styles/FullCalendar.css` (see [[ui/Styling & Theme]]).

## ClassGrid

**File:** `components/classes/ClassGrid.tsx`  
Client component. Renders class cards from `lib/data/mock_data.ts`.

### Class Card Layout

```
Colored accent bar (cls.color Tailwind class)
Class name + status pill
Instructor name
Clock icon + time + duration
Users icon + enrolled/capacity progress bar
DropdownMenu (Edit, Delete — non-functional)
```

### Status Pills

| Status | Color |
|--------|-------|
| Full | red-500 |
| Almost Full | yellow-500 |
| Open | gym-green |

### Mock Classes (from `lib/data/mock_data.ts`)

| Name | Instructor | Status | Enrolled/Cap |
|------|-----------|--------|--------------|
| Morning Yoga Flow | Elena Fisher | Open | 12/20 |
| CrossFit WOD | Kratos | Almost Full | 18/20 |
| HIIT Blast | Marcus | Full | 25/25 |
| Power Lifting | Sarah | Open | 8/15 |
| Strength Training | James | Open | 5/20 |

### Enrollment Progress Bar

Width = `(enrolled / capacity) * 100%`. Uses `cls.color` as the bar color.

### `children` prop

`ClassGrid` declares a `children?: ReactNode` prop that is **never rendered** in the component body. See [[reference/Conventions & Gotchas]].

## Real Data Path

When Firestore is wired up, classes would read from `Firestore /gyms/{gymId}/classes`, the calendar would reflect real scheduled sessions, and the "Schedule Class" button would open a creation form.

## Related pages

- [[ui/Component Library]] — ClassCalendar, ClassGrid, DropdownMenu
- [[data/Mock Data Layer]] — mock_data.ts class type definition
- [[overview/Domain Model]] — Class entity
