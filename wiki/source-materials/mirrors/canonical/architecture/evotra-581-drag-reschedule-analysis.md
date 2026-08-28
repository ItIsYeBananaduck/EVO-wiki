---
title: "EVOTRA-581 Analysis: Drag-and-drop rescheduling in CalendarWeekView"
type: source-material
tags: ['lsctech', 'source-material', 'canonical', 'evo']
updated: 2026-06-18
---

# EVOTRA-581 Analysis: Drag-and-drop rescheduling in CalendarWeekView

**Analyst session:** 2026-06-18  
**Confidence:** 91%  
**Status:** prerequisites met — implementation-ready

---

## Prerequisites

| Issue | Title | Status |
|-------|-------|--------|
| EVOTRA-560 | Coach Calendar — Cluster C: Core Calendar UI | ✅ Done |
| EVOTRA-572 | Coach Calendar — Cluster E: Polish, Recurrence | ✅ Done |

No blockers. Safe to proceed.

---

## Affected Files

| File | Role | Change |
|------|------|--------|
| `flutter_app/lib/features/coach/presentation/calendar/calendar_week_view.dart` | Week grid UI | Primary — StatefulWidget conversion, Draggable/DragTarget, visual feedback |
| `flutter_app/lib/features/coach/presentation/calendar/calendar_tab.dart` | Parent host | Add `onWindowReschedule` wiring → CalendarBloc |
| `flutter_app/lib/features/coach/domain/availability_service.dart` | Domain | No changes — `updateWindow`, `editThisOccurrenceOnly`, `editAllFuture` already exist |
| `flutter_app/lib/features/coach/presentation/calendar/calendar_bloc.dart` | BLoC | No changes — `updateAvailability` already exists |
| `flutter_app/test/features/coach/calendar/calendar_drag_reschedule_test.dart` | Tests | New file |

---

## Architecture Findings

### CalendarWeekView is StatelessWidget
`calendar_week_view.dart` is a `StatelessWidget`. Drag state (active drag window, hovered drop target time) is ephemeral UI state that must live inside the widget tree. **Must convert to `StatefulWidget`** before adding `Draggable`/`DragTarget`.

### Callback wiring
`CalendarTab._buildCalendarView()` constructs `CalendarWeekView` with tap callbacks. A new `onWindowReschedule(AvailabilityWindow, DateTime newStart, DateTime newEnd)` callback routes through `_bloc.updateAvailability(window.copyWith(...))`. No BLoC changes needed.

### Slot grid geometry
- Each cell is 48px tall, 30-minute duration (`timeSlotDuration`)
- Grid column width is `Expanded` within `Row` — no fixed pixel width
- Drop target time can be read directly from the `slotStart` DateTime passed to each cell

### 15-minute snap
Grid cells are 30-minute slots. Snapping to 15 minutes means rounding `slotStart.minute` to `{0, 15, 30, 45}`. Logic: `minute - (minute % 15)` if remainder < 8, else `minute + (15 - minute % 15)`.

### Booked-slot drag constraint
Issue requires "coach cannot orphan a booked slot." The clean implementation: **booked availability windows (those with an overlapping `AppointmentSlot`) are not wrapped with `Draggable`**. The `TimeSlot.isBooked` flag already captures this. No `AppointmentService` changes needed.

### Recurring window drag
`AvailabilityService.editThisOccurrenceOnly()` and `editAllFuture()` already exist. On drop of a recurring window (`recurrenceRule != null`), show a bottom sheet: "Reschedule this occurrence only / All future occurrences." The appropriate service method is called based on user choice.

### Flutter drag primitives
Use `Draggable<AvailabilityWindow>` (data=window) on available/blocked cells and `DragTarget<AvailabilityWindow>` on every grid cell. `onWillAcceptWithDetails` returns false if the target cell already contains a window (≠ source) or any appointment.

---

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| StatefulWidget conversion breaks existing tap behavior | Low | Callbacks are unchanged; only `build` method extracted to State |
| DragTarget covers GestureDetector — tap conflicts | Medium | Replace `GestureDetector` with `DragTarget` (DragTarget passes through taps via `onTap` in builder) |
| Recurring dialog blocks async flow | Low | `showModalBottomSheet` returns a Future; await before calling reschedule |
| Window-duration overflow across day boundary | Low | Validate `newEndsAt` ≤ midnight of same day; show snackbar if invalid |

---

## Execution Plan Summary

4 children in strict serial order (each blocks the next):

1. **C1** — StatefulWidget conversion + `onWindowReschedule` callback scaffolding
2. **C2** — `Draggable<AvailabilityWindow>` on available/blocked cells, ghost feedback
3. **C3** — `DragTarget` grid cells, drop validation, 15-min snap, recurrence dialog
4. **C4** — Unit + widget tests for drag-to-reschedule
