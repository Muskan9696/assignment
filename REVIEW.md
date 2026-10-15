# Starter code review

| # | Where | Defect | How to notice | Fix |
|---:|---|---|---|---|
| 1 | `compute_late_minutes` | Negative differences are possible and overnight shifts are not anchored to the correct shift start. | Test a 02:00 punch-in for a 22:00–06:00 shift. | Compute the relevant IST shift start, including the prior day for overnight shifts. |
| 2 | `compute_late_minutes` | Grace is implemented as `minutes > 10`, which incorrectly treats 10:00:59 as not late only because seconds are discarded inconsistently; the rule is strictly more than 10 minutes after the shift start. | Test 09:40:01 for a 09:30 shift. | Compare elapsed seconds against 600, then floor whole minutes. |
| 3 | `compute_work_hours` | Python `round` uses ties-to-even and binary floating-point, not decimal half-up rounding. | Test durations exactly halfway between hundredths. | Use decimal half-up rounding. |
| 4 | `compute_overtime` | Shift end is always on the attendance date, so overnight shifts calculate overtime against the wrong instant. | Punch out at 06:40 after a 22:00–06:00 shift. | Build the shift-end instant on the next day for overnight shifts. |
| 5 | `create_employee` | Duplicate checking is check-then-insert and is not safe under concurrent requests. | Send two simultaneous creates with the same `emp_code`. | Enforce uniqueness with a MongoDB unique index and handle duplicate-key errors. |
| 6 | `create_employee` | `shift_start == shift_end` is accepted despite being invalid for this contract. | POST an employee with both values `09:30`. | Validate that the times differ. |
| 7 | `create_employee` | `created_at` uses a naive local datetime rather than a normalized UTC instant and is returned without epoch-millisecond conversion. | Compare returned values with the API's epoch-ms convention. | Store UTC BSON datetime and convert instants at the API boundary. |
| 8 | `list_employees` | `skip` uses `page * page_size`, skipping the first page. | Request page 1 and compare against sorted records. | Use `(page - 1) * page_size`; sort by `emp_code`; count the filtered query. |
| 9 | `list_employees` | `total` ignores the department filter. | Filter to a department with fewer employees than the whole company. | Count with the same filter used to fetch items. |
| 10 | `punch_in` | An unknown employee causes a `None` dereference (500). | Punch in using an unknown code. | Return 404 before using employee shift fields. |
| 11 | `punch_in` | Timestamp seconds, floats, strings and out-of-range milliseconds are not validated; timestamps are not truncated to whole seconds. | Send seconds or a fractional millisecond timestamp. | Validate epoch milliseconds and truncate to seconds. |
| 12 | `punch_in` | Uses host-local time and `date()` instead of the IST attendance date, including the overnight-shift exception. | Use a UTC timestamp near midnight or an overnight shift. | Derive attendance date in IST from the employee shift. |
| 13 | `punch_in` | Check-then-insert is not concurrency-safe, and no unique index is created. | Fire parallel punch-ins for the same attendance date. | Unique compound index on `(emp_code, date)` and map duplicate-key to 409. |
| 14 | `punch_in` | Does not reject non-presence statuses or consistently return all required response fields. | Try `ABSENT` as a punch-in status. | Restrict to `PRESENT`, `WFH`, and `ON_DUTY`; normalize response fields. |
| 15 | `list_attendance` | Loads every matching record into Python, then sorts and slices in memory. | Run against a large dataset or inspect query plan. | Count separately and sort/skip/limit in MongoDB with supporting indexes. |
| 16 | `list_attendance` | Sorting omits the required `emp_code` tie-breaker. | Several records share the same date. | Sort by date descending, then employee code ascending. |
| 17 | `list_attendance` | Does not validate date ranges, page bounds, or status values; `total` is only the loaded list length. | Supply reversed dates, page 0, or an invalid status. | Validate parameters and count all filtered matches. |
| 18 | Missing `/attendance/punch-out` | No punch-out behaviour exists. | Call the documented route. | Implement an atomic conditional update, elapsed-time validation and derived fields. |
| 19 | Missing `/attendance/{emp_code}/{date}` | No correction endpoint or audit history exists. | PATCH a record. | Validate corrections, recompute derived values, append one history entry atomically. |
| 20 | Missing analytics endpoints | Required summaries, rankings and trend are absent. | Call any documented analytics route. | Implement aggregation pipelines in MongoDB. |
| 21 | Missing explain endpoint | There is no way to return execution-statistics explain plans. | Call `/admin/explain/...`. | Expose explain output for the named operation and its parameters. |
| 22 | Startup and health | No indexes are created and `/health` always returns success without checking MongoDB. | Stop MongoDB and call health; inspect indexes. | Create idempotent indexes at startup and ping MongoDB for readiness. |

## Checked and retained

- The starter's `compute_work_hours` formula correctly measures elapsed duration in hours conceptually; its rounding method needs replacement.
- Using `emp_code` as the business identifier is correct. MongoDB `_id` should remain internal.
- `history` should be written only by manual corrections, not ordinary punch-in or punch-out events.
