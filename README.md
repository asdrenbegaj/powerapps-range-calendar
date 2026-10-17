# Range Calendar for Power Apps

A date-range calendar **component** for Power Apps canvas apps. It renders any month, marks
today, and lets people pick a single day or a range. It respects booking rules too: earliest
and latest dates, blocked dates, weekends and a maximum length.

It's the booking calendar from our Asset Booking app, rebuilt as a reusable component so
other teams can use it.

**[Live demo and walkthrough →](https://asdrenebagj.github.io/powerapps-range-calendar/)**

## Get it

| I want to… | Download | How |
|---|---|---|
| Use it in my apps | [`RangeCalendar_solution.zip`](../../releases/latest) | **Solutions → Import solution**, then in your app **Insert → Get more components** |
| Try it first | [`RangeCalendarDemo.msapp`](../../releases/latest) | **Apps → Import app → From file (.msapp)** |
| Read or rebuild the source | [`src/component/`](src/component/RangeCalendar.component.pa.yaml) | Follow [docs/BUILD.md](docs/BUILD.md) |
| Learn the technique only | [`src/paste-in/`](src/paste-in/RangeCalendar.pa.yaml) | Paste onto any screen. This simpler version uses app variables. |

Importing a solution needs an environment with Dataverse and permission to import
solutions. If you can't do that, use the demo app or build the component from the YAML.

## Properties

**Inputs** (your app sets these; all optional)

| Property | Type | Default | What it does |
|---|---|---|---|
| `MinDate` | Date and time | `Today()` | Days before this are disabled. `Blank()` allows past dates. |
| `MaxDate` | Date and time | `Blank()` | Days after this are disabled. Blank means no limit. |
| `BlockedDates` | Table | `Table({BlockedDate: Date(1900, 1, 1)})` | Days that can't be picked, in a column called `BlockedDate`. A range can't cross one. |
| `DisableWeekends` | Boolean | `false` | Saturdays and Sundays can't be a start or end day. A range can still span a weekend. |
| `AllowRange` | Boolean | `true` | Turn off for single-day picking. |
| `MaxDays` | Number | `0` | Longest range allowed, counting both ends. `0` means no limit. |
| `AccentColor` | Color | `RGBA(0, 94, 184, 1)` | Selection, today ring and buttons. |

**Outputs** (your app reads these)

| Property | Type | Value |
|---|---|---|
| `StartDate` | Date and time | First selected day, or blank |
| `EndDate` | Date and time | Last selected day. Equals `StartDate` until an end is picked |
| `DayCount` | Number | Days selected, counting both ends. `0` when empty |
| `IsComplete` | Boolean | `true` once an end day has been picked |

**Reset:** `Reset(RangeCalendar1)` clears the selection and goes back to this month.

## Example

```powerfx
// btnBook.DisplayMode
If(RangeCalendar1.IsComplete, DisplayMode.Edit, DisplayMode.Disabled)

// btnBook.OnSelect
Patch(Bookings, Defaults(Bookings),
    { StartDate: RangeCalendar1.StartDate, EndDate: RangeCalendar1.EndDate });
Reset(RangeCalendar1)
```

Blocking days that are already booked (bookings stored as start/end dates):

```powerfx
Clear(colBlocked);
ForAll(
    Filter(Bookings, AssetID = varAsset.ID, EndDate >= Today()) As b,
    Collect(colBlocked,
        ForAll(Sequence(DateDiff(b.StartDate, b.EndDate, TimeUnit.Days) + 1, 0) As i,
            { BlockedDate: DateAdd(b.StartDate, i.Value, TimeUnit.Days) }))
)
// then set RangeCalendar1.BlockedDates to colBlocked
```

Use dates without times (SharePoint "Date only" columns, `Today()`, `Date()`), or blocked
dates won't match.

## How it works

- **State is private.** Inside a component, `Set()` variables belong to that one copy, so
  `_month`, `_start` and `_end` never clash with your app or with another calendar on the same screen.
- **42 cells from `Sequence(42, 0)`.** These cover six weeks, offset by
  `Weekday(first, StartOfWeek.Monday) - 1` so each row starts on Monday.
- **One `IsDisabled` flag per cell** combines every rule from the input properties. Buttons
  use `DisplayMode`, so disabled days can't be clicked or tabbed to.
- **One selection rule:** if a click can't finish a valid range, it starts a new one.
- **No experimental features.** Output properties recalculate by themselves, so there's no need for a custom
  `OnChange` event property (still experimental in Power Apps). Reset uses the built-in `OnReset`.

The [demo page](https://asdrenbegaj.github.io/powerapps-range-calendar/) explains each formula
and runs the same logic in the browser.

## Repository layout

```
index.html                                  the GitHub Pages site
src/component/RangeCalendar.component.pa.yaml   controls inside the component
src/paste-in/RangeCalendar.pa.yaml           simple paste-in version (app variables)
docs/BUILD.md                               building the component yourself
```

Release assets (`RangeCalendar_solution.zip`, `RangeCalendarDemo.msapp`) are attached to each
[release](../../releases).

## Licence

MIT. See [LICENSE](LICENSE). Made by the Digital Services Team at University Hospitals of
Morecambe Bay NHS Foundation Trust.
