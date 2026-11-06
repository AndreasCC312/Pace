[README.md](https://github.com/user-attachments/files/34504441/README.md)
# Pace

Pace is a single-page to-do list that estimates how long each task will take and corrects itself when you run over. Tasks appear as speech bubbles pinned to a cork bulletin board.

It is one self-contained file (`task-estimator.html`) with no build step and no server.

## What Pace does

- **Estimates task duration** from the wording of the task, adjusted by how long your past tasks actually took.
- **Collects tasks from your calendar** by importing events from an `.ics` file, using each event's length as its first estimate.
- **Times your work** with a clock that starts when you press Start.
- **Re-estimates when you run over.** If you pass your estimate without finishing, Pace asks how far along you are and projects a new finish time from your actual pace.
- **Learns from history.** Finished tasks feed a correction factor that scales future estimates.
- **Saves everything in your browser**, so your board is still there when you return.

## Using Pace

1. **Connect your calendar (optional).** On first load, Pace asks which calendar you use (Google Calendar, Outlook, or Apple Calendar). You can reopen this with the **Connect calendar** button. See [Calendar import](#calendar-import) for how it works.
2. **Add a task.** Type a task name in the yellow note. The estimate fills in as you type; edit it if you disagree. Press **Add task** or Enter.
3. **Start it.** Press **Start** on the bubble. The progress bar and the "time spent / time left" line begin counting.
4. **Finish or report.**
   - Press **Mark done** when you finish.
   - If you are running late, Pace opens a re-estimate prompt automatically once you pass the estimate. You can also open it early with **Not finished?**.
5. **Review.** Finished tasks move to the Finished section of the board, faded.

The **+10 min** link on a running task adds ten minutes to its clock. It exists for testing the re-estimate flow without waiting. **Remove** deletes a task.

## How it operates

### Task lifecycle

Each task is in one of three states:

| State | Meaning | Available actions |
| --- | --- | --- |
| `todo` | Added but not started | Start, Remove |
| `running` | Clock is running | Mark done, Not finished?, +10 min, Remove |
| `done` | Finished | Remove |

Elapsed time is computed from the timestamp when you pressed Start, not from a counter. The clock therefore keeps correct time if the tab is in the background or closed and reopened. There is no pause.

### Estimating duration

When you type a task name, Pace picks a base duration from keywords. If several keyword groups match, the group lower in this table wins.

| Keywords | Base |
| --- | --- |
| email, reply, text, message, call back | 10 min |
| quick, short, brief, pay, book, schedule | 10 min |
| call, meeting, meet, sync, interview | 30 min |
| read, review, proofread, edit, plan, organize | 35 min |
| clean, laundry, dishes, cook, shop, errand, groceries, buy | 40 min |
| study, practice, homework, assignment, flashcards, vocab | 60 min |
| workout, gym, run, exercise | 60 min |
| write, essay, report, draft, paper, presentation, slides | 90 min |
| code, build, develop, design, research, project, exam, midterm | 120 min |

If nothing matches, the base is 30 minutes, or 45 minutes when the name is longer than six words.

The base is then multiplied by a **learned factor** and rounded to the nearest 5 minutes (minimum 5):

- Each time you mark a task done, Pace records the original estimate and the actual time taken.
- The factor is the **median** of actual ÷ estimate over your last 10 finished tasks.
- It is used only once you have at least 2 finished tasks (otherwise it is 1.0) and is limited to the range 0.5 to 3.0.
- The hint under the new-task form tells you when an adjustment is being applied.

The comparison uses the original estimate, so a task you had to re-estimate still counts as an overrun in your history.

### Re-estimating a late task

A re-estimate prompt appears when a running task's elapsed time exceeds its estimate, or when you press **Not finished?**. You drag a slider to report how much of the task is done (5% to 95%).

Pace assumes you will keep working at the pace you have shown so far:

```
elapsed        = time spent so far (minutes)
p              = reported progress (clamped to 5%–95%)
projected      = max(elapsed + 1, elapsed / p)
remaining      = max(1, ceil(projected − elapsed))
new estimate   = ceil(elapsed + remaining)
```

For example, at 30 minutes in and 40% done, the projected total is 75 minutes, so 45 minutes remain and the new estimate is 75 minutes.

After a re-estimate, the bubble shows the new total and a note recording what you reported and how the estimate changed. If you run past the new estimate, the prompt appears again.

### Calendar import

Pace does not sign in to calendar services itself. A page like this cannot authenticate with Google, Microsoft, or Apple. Choosing a calendar shows how to export an `.ics` file instead, and **Choose .ics file** imports it.

For each event, Pace:

- Uses the title as the task name.
- Uses the gap between start and end as the estimate. If an event has no end, a zero length, or is longer than 12 hours, it falls back to the keyword estimate.
- Skips events that ended more than a day ago.
- Skips a task if an unfinished task with the same title already exists.

Imported tasks are ordinary tasks. You can edit, start, or remove them like any other.

### Storage

Tasks, finished-task history, and the connected-calendar label are stored in the browser's `localStorage` under the key `pace.v1`. Data stays on your device, is not shared between devices or browsers, and is lost if you clear site data. If storage is unavailable, Pace still works for the current session.

## Interface

- **Board:** a cork bulletin board with a wooden frame. The new-task form is a sticky note at the top.
- **Bubbles:** each task is a speech bubble with its own pushpin color and slight tilt. The name is at the top, with the estimated duration shown large beneath it.
- **Progress bar:** fills as time passes, turns amber once you pass the estimate, and shows a marker for how the estimate was revised.
- **Themes:** follows your system light or dark setting. The cork board looks the same in both.
- **Accessibility:** controls have visible keyboard focus, and motion is reduced when your system asks for it.

## Limitations

- No live calendar connection; import is manual via `.ics`.
- No pause or resume. A running task keeps counting.
- Estimates come from keywords and your history, not from understanding the task. A vague name gets a generic estimate.
- Re-estimates assume a constant pace, so they are less accurate for tasks whose later stages are harder or easier than the early ones.
- Recurring calendar events are imported only once, as written in the file, and are not expanded into each occurrence.

## Files

| File | Purpose |
| --- | --- |
| `task-estimator.html` | The whole app: markup, styles, and logic |
| `README.md` | This document |
