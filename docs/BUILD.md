# Building the component yourself

Use this if you can't import the solution, or you'd like to see how the component is put
together. It takes about 20 minutes in Power Apps Studio.

The order matters. Input properties must exist **before** you paste the controls, because
the controls' formulas refer to them.

## 1. Create the component library

1. Go to [make.powerapps.com](https://make.powerapps.com) and pick your environment.
2. Select **Component libraries** in the left menu (it may be under **… More**), then
   **New component library**. Name it `Range Calendar` and select **Create**.
3. Studio opens with one empty component called `Component1`. In the **Tree view**, select
   **…** next to it → **Rename**, and call it exactly **`cmpRangeCalendar`**.
4. With `cmpRangeCalendar` selected, set **Width** `364` and **Height** `430` in the
   Properties pane on the right.

## 2. Add the input properties

With `cmpRangeCalendar` selected, select **+ New custom property** in the Properties pane
for each row below. Use the **same text for Display name and Property name**, and set
**Property type** to **Input**. Leave **Raise OnReset when value changes** unticked.

| Property name | Data type |
|---|---|
| `MinDate` | Date and time |
| `MaxDate` | Date and time |
| `BlockedDates` | Table |
| `DisableWeekends` | Boolean |
| `AllowRange` | Boolean |
| `MaxDays` | Number |
| `AccentColor` | Color |

Then set each one's default. Select the component, pick the property in the formula-bar
drop-down (top left), and type:

| Property | Default formula |
|---|---|
| `MinDate` | `Today()` |
| `MaxDate` | `Blank()` |
| `BlockedDates` | `Table({BlockedDate: Date(1900, 1, 1)})` |
| `DisableWeekends` | `false` |
| `AllowRange` | `true` |
| `MaxDays` | `0` |
| `AccentColor` | `RGBA(0, 94, 184, 1)` |

## 3. Paste the controls

1. Open [`src/component/RangeCalendar.component.pa.yaml`](../src/component/RangeCalendar.component.pa.yaml),
   select all of it and copy.
2. In Studio, click the empty component canvas and press **Ctrl+V**.
3. `rcFrame` appears with the arrows, month label, two galleries, summary and Clear button inside.
4. If Studio says a control version isn't recognised, change the `@x.y.z` after that control
   type in the YAML to the version your Studio uses, and paste again.

## 4. Add the output properties and OnReset

Add four more custom properties, this time with **Property type: Output**:

| Property name | Data type | Formula |
|---|---|---|
| `StartDate` | Date and time | `_start` |
| `EndDate` | Date and time | `If(IsBlank(_start), Blank(), Coalesce(_end, _start))` |
| `DayCount` | Number | `If(IsBlank(_start), 0, DateDiff(_start, Coalesce(_end, _start), TimeUnit.Days) + 1)` |
| `IsComplete` | Boolean | `!IsBlank(_end)` |

Then select the component, choose **OnReset** in the formula-bar drop-down and enter:

```powerfx
Set(_start, Blank()); Set(_end, Blank()); Set(_month, Blank())
```

## 5. Save, publish, use

1. **Save** (add a version note), then **Publish**. Only published versions reach apps.
2. In any app in the same environment: **Insert → Get more components**, then tick
   **cmpRangeCalendar** and select **Import**. It appears under **Library components** in the Insert pane.

## Checking it works

- The month shows today's month, with today ringed and past days greyed out.
- Click one day, then a later one: the range fills in.
- Add two copies to one screen. Selecting in one doesn't change the other.
- Add a label with `RangeCalendar1.DayCount`: it updates as you click.
- A button with `Reset(RangeCalendar1)` clears the selection.
