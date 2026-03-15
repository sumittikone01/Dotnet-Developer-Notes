
# 06 — Kendo TimePicker

---

## 🎯 One-Line Definition

> **The Kendo TimePicker is a styled time input that shows a scrollable dropdown of time slots — it enforces valid time entry, supports 12h/24h formats, configurable step intervals, and integrates with your model's DateTime or TimeSpan properties.**

---

## 🔑 Plain Input vs Kendo TimePicker

```
Plain <input type="time">:          Kendo TimePicker:
────────────────────────────────     ────────────────────────────────────
[09:30                     ]         [09:30 AM                      🕐]
                                       ↓ click 🕐
                                      ┌─────────────────────────────┐
Browser-dependent UI                  │ 12:00 AM                    │
No step control                       │ 12:30 AM                    │
No 12h/24h toggle                     │ 01:00 AM                    │
No min/max time enforcement           │ ...                         │
No Kendo styling                      │ ▶ 09:30 AM  ← current      │
                                      │ ...                         │
                                      │ 09:00 PM                    │
                                      └─────────────────────────────┘
                                     Consistent UI, configurable steps
```

---

## 🔑 Basic Setup

```cshtml
{{!-- Minimal --}}
<kendo-timepicker name="MeetingTime" />


{{!-- With common options --}}
<kendo-timepicker name="MeetingTime"
                  value="@Model.MeetingTime"
                  format="HH:mm"
                  min="new DateTime(2000,1,1,8,0,0)"
                  max="new DateTime(2000,1,1,18,0,0)"
                  interval="30"
                  placeholder="Select time..." />
```

---

## 🔑 Key Attributes

```cshtml
<kendo-timepicker name="WorkStart"
                  value="@Model.WorkStart"
                  format="hh:mm tt"
                  min="new DateTime(2000,1,1,6,0,0)"
                  max="new DateTime(2000,1,1,22,0,0)"
                  interval="15"
                  placeholder="Pick start time..."
                  enabled="true"
                  readonly="false" />
```

| Attribute       | What It Does                 | Example                           |
| --------------- | ---------------------------- | --------------------------------- |
| `name`        | Field name — binds to model | `name="WorkStart"`              |
| `value`       | Pre-selected time            | `value="@Model.WorkStart"`      |
| `format`      | Display format               | `"HH:mm"`,`"hh:mm tt"`        |
| `min`         | Earliest selectable time     | `new DateTime(2000,1,1,8,0,0)`  |
| `max`         | Latest selectable time       | `new DateTime(2000,1,1,18,0,0)` |
| `interval`    | Minutes between time slots   | `30`(every 30 min)              |
| `placeholder` | Hint text when empty         | `"Select time"`                 |
| `enabled`     | Enable or disable            | `false`= greyed out             |
| `readonly`    | Show value, no edit          | `true`                          |

---

## 🔑 Format Strings — 12h vs 24h

```cshtml
{{!-- 24-HOUR (most used in business apps) --}}
<kendo-timepicker format="HH:mm" />
{{!-- Shows: 09:30, 13:00, 17:45 --}}

<kendo-timepicker format="HH:mm:ss" />
{{!-- Shows: 09:30:00 (with seconds) --}}


{{!-- 12-HOUR (AM/PM) --}}
<kendo-timepicker format="hh:mm tt" />
{{!-- Shows: 09:30 AM, 01:00 PM --}}

<kendo-timepicker format="h:mm tt" />
{{!-- Shows: 9:30 AM, 1:00 PM (no leading zero) --}}


{{!-- Combined date + time (use DateTimePicker instead) --}}
{{!-- TimePicker is time only --}}
```

```
Format tokens:
  H    → hour 24h (0-23, no leading zero)
  HH   → hour 24h (00-23, with leading zero)
  h    → hour 12h (1-12, no leading zero)
  hh   → hour 12h (01-12, with leading zero)
  m    → minute (0-59, no leading zero)
  mm   → minute (00-59, with leading zero)
  s    → second (0-59)
  ss   → second (00-59)
  tt   → AM/PM designator
```

---

## 🔑 `interval` — Time Slot Gaps

Controls how many minutes apart each option in the dropdown is:

```cshtml
{{!-- 15-minute slots --}}
<kendo-timepicker name="AppointmentTime"
                  interval="15"
                  format="hh:mm tt" />
{{!-- Shows: 8:00 AM, 8:15 AM, 8:30 AM, 8:45 AM, 9:00 AM... --}}


{{!-- 30-minute slots (most common) --}}
<kendo-timepicker name="MeetingTime"
                  interval="30" />
{{!-- Shows: 8:00, 8:30, 9:00, 9:30... --}}


{{!-- 60-minute slots (hourly) --}}
<kendo-timepicker name="ShiftStart"
                  interval="60"
                  format="HH:mm" />
{{!-- Shows: 00:00, 01:00, 02:00... --}}


{{!-- Note: user CAN still type any time manually --}}
{{!-- interval only affects the dropdown slots --}}
```

---

## 🔑 Min and Max Time — Business Hours Pattern

```cshtml
{{!-- Business hours only (8 AM to 6 PM) --}}
<kendo-timepicker name="AppointmentTime"
                  min="new DateTime(2000,1,1,8,0,0)"
                  max="new DateTime(2000,1,1,18,0,0)"
                  interval="30"
                  format="hh:mm tt" />
{{!-- Only 8:00 AM through 6:00 PM shown in dropdown --}}
{{!-- Times outside range are greyed out / not shown --}}


{{!-- Morning only --}}
<kendo-timepicker name="MorningSlot"
                  min="new DateTime(2000,1,1,6,0,0)"
                  max="new DateTime(2000,1,1,12,0,0)"
                  interval="15" />


{{!-- No restriction (full 24 hours) --}}
<kendo-timepicker name="AnyTime"
                  interval="30" />
```

> **Why `2000,1,1`?** The `min`/`max` need a full `DateTime` object — Kendo only reads the time part. The date `2000,1,1` is just a placeholder — any date works.

---

## 🔑 JavaScript API

```javascript
var tp = $("#MeetingTime").data("kendoTimePicker");

// ── Reading the value ──────────────────────────────────────────
tp.value()
// Returns a JavaScript Date object  (or null if empty)
// The DATE part is today's date — only the TIME is meaningful
// e.g.: Tue Jan 01 2000 09:30:00

// Extract just the time:
var date = tp.value();
if (date) {
    var hours   = date.getHours();    // 9
    var minutes = date.getMinutes();  // 30
    var formatted = kendo.toString(date, "HH:mm");  // "09:30"
    console.log("Time:", formatted);
}

// ── Setting the value ──────────────────────────────────────────
// Set using a Date object
tp.value(new Date(2000, 0, 1, 9, 30));  // sets to 09:30

// Set using a string (must match the format)
tp.value("09:30");   // works with format="HH:mm"
tp.value("9:30 AM"); // works with format="h:mm tt"

// Clear the time
tp.value(null);

// Set to current time
tp.value(new Date());

// ── Enable / Disable ──────────────────────────────────────────
tp.enable(false)
tp.enable(true)

// ── Read-only ─────────────────────────────────────────────────
tp.readonly(true)
tp.readonly(false)

// ── Update min/max dynamically ────────────────────────────────
tp.min(new Date(2000, 0, 1, 9, 0));    // min is now 9:00 AM
tp.max(new Date(2000, 0, 1, 17, 0));   // max is now 5:00 PM

// ── Open / Close dropdown ─────────────────────────────────────
tp.open()
tp.close()
```

---

## 🔑 Events

```javascript
var tp = $("#MeetingTime").data("kendoTimePicker");

// ── Change: fires when time is picked or typed ────────────────
tp.bind("change", function() {
    var date = this.value();

    if (!date) {
        console.log("Time cleared");
        return;
    }

    var timeStr = kendo.toString(date, "HH:mm");
    console.log("Time selected:", timeStr);

    // Real-world: calculate meeting end time
    var endTime = new Date(date.getTime() + 60 * 60 * 1000);  // +1 hour
    $("#MeetingEnd").data("kendoTimePicker").value(endTime);
});

// ── Open: fires when dropdown opens ──────────────────────────
tp.bind("open", function() {
    console.log("Time picker opened");
});

// ── Close: fires when dropdown closes ────────────────────────
tp.bind("close", function() {
    // Validate start < end
    var start = $("#StartTime").data("kendoTimePicker").value();
    var end   = $("#EndTime").data("kendoTimePicker").value();
    if (start && end && start >= end) {
        showNotification("End time must be after start time", "error");
        this.value(null);
    }
});
```

---

## 🔑 Start/End Time Pair — Most Common Real Pattern

```cshtml
<div class="row">
    <div class="col-md-3">
        <label>Start Time *</label>
        <kendo-timepicker name="StartTime"
                          value="@Model.StartTime"
                          format="hh:mm tt"
                          min="new DateTime(2000,1,1,8,0,0)"
                          max="new DateTime(2000,1,1,21,0,0)"
                          interval="30" />
    </div>
    <div class="col-md-3">
        <label>End Time *</label>
        <kendo-timepicker name="EndTime"
                          value="@Model.EndTime"
                          format="hh:mm tt"
                          min="new DateTime(2000,1,1,8,0,0)"
                          max="new DateTime(2000,1,1,22,0,0)"
                          interval="30" />
    </div>
</div>
```

```javascript
$(function() {
    var startPicker = $("#StartTime").data("kendoTimePicker");
    var endPicker   = $("#EndTime").data("kendoTimePicker");

    // When start changes → update end's minimum
    startPicker.bind("change", function() {
        var start = this.value();
        if (!start) return;

        // End must be after start
        endPicker.min(start);

        // If current end is now invalid, clear it
        var currentEnd = endPicker.value();
        if (currentEnd && currentEnd <= start) {
            endPicker.value(null);
        }

        // Auto-suggest end time (+1 hour)
        if (!endPicker.value()) {
            var suggested = new Date(start.getTime() + 60 * 60 * 1000);
            endPicker.value(suggested);
        }
    });
});
```

---

## 🔑 TimePicker with DatePicker — Date + Time Together

When you need both date AND time, pair a DatePicker with a TimePicker:

```cshtml
<div class="row">
    <div class="col-md-3">
        <label>Date</label>
        <kendo-datepicker name="MeetingDate"
                          value="@Model.MeetingDate"
                          format="MM/dd/yyyy" />
    </div>
    <div class="col-md-3">
        <label>Time</label>
        <kendo-timepicker name="MeetingTime"
                          value="@Model.MeetingTime"
                          format="hh:mm tt"
                          interval="30" />
    </div>
</div>
```

```javascript
// Combine date + time into one DateTime before sending
function getMeetingDateTime() {
    var date = $("#MeetingDate").data("kendoDatePicker").value();
    var time = $("#MeetingTime").data("kendoTimePicker").value();

    if (!date || !time) return null;

    // Combine: take date parts from date, time parts from time
    return new Date(
        date.getFullYear(),
        date.getMonth(),
        date.getDate(),
        time.getHours(),
        time.getMinutes(),
        0   // seconds
    );
}

// Usage:
var meeting = getMeetingDateTime();
if (meeting) {
    console.log("Meeting at:", kendo.toString(meeting, "MM/dd/yyyy hh:mm tt"));
}
```

---

## 🔑 Sending Time to the Server

```javascript
var tp = $("#MeetingTime").data("kendoTimePicker");
var time = tp.value();

// Option A: send as "HH:mm" string
var payload = {
    title:     $("#Title").val(),
    timeString: time ? kendo.toString(time, "HH:mm") : null  // "09:30"
};

// Option B: send as full ISO DateTime (server reads time part)
var payload2 = {
    title:    $("#Title").val(),
    meetTime: time ? time.toISOString() : null  // "2000-01-01T09:30:00.000Z"
};

$.post("/Meeting/Save", payload, function(r) { ... });
```

```csharp
// Controller — receive the time string
public JsonResult Save(string title, string timeString)
{
    // Parse "09:30" → TimeSpan
    TimeSpan meetTime = TimeSpan.Parse(timeString);
}

// OR receive as DateTime — use the time part
public JsonResult Save(string title, DateTime meetTime)
{
    int hour   = meetTime.Hour;    // 9
    int minute = meetTime.Minute;  // 30
}
```

---

## ⚠️ Common Mistakes

| Mistake                                      | Symptom                                        | Fix                                                              |
| -------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| Reading `.val()`instead of `.value()`    | Gets formatted string `"09:30 AM"`not a Date | Use `.data("kendoTimePicker").value()`                         |
| Using `Date()`constructor month confusion  | Wrong time set                                 | For time only:`new Date(2000, 0, 1, 9, 30)`(month = 0 for Jan) |
| Comparing times with `===`                 | Always false for same time                     | Use `.getTime()`:`t1.getTime() === t2.getTime()`             |
| `min`and `max`use same date as value     | Works fine — date part is ignored             | Any date works:`new DateTime(2000,1,1,8,0,0)`                  |
| `interval`set but user types between slots | Valid — user can still type any time          | This is by design; add server-side validation if needed          |

---

## ❓ Interview Questions

**Q: What does `interval` do in the Kendo TimePicker?**

> It sets the gap in minutes between each time slot shown in the dropdown. `interval="30"` shows 8:00, 8:30, 9:00 etc. The user can still type any time manually — `interval` only affects the dropdown list.

**Q: How do you get the selected time as a formatted string?**

> `var date = tp.value()` returns a JS Date object. Then `kendo.toString(date, "HH:mm")` formats it. Always check `if (date)` first since `.value()` returns null when empty.

**Q: What does the date part of the TimePicker's value represent?**

> Nothing meaningful. The TimePicker returns a full Date object but only the time portion matters. The date part defaults to the current date or a placeholder date. Always use `.getHours()` and `.getMinutes()` or `kendo.toString(date, "HH:mm")` — never rely on the date part.

**Q: How do you pair a DatePicker and TimePicker to create a combined DateTime?**

> Read both values, then construct a new Date: `new Date(date.getFullYear(), date.getMonth(), date.getDate(), time.getHours(), time.getMinutes(), 0)`. This combines the date parts from the DatePicker with the time parts from the TimePicker.
>
