# DECISIONS.md

1. **Indexes.** I use a unique index on `employees.emp_code` to enforce identity, and a unique compound index on `attendance_logs.emp_code, date` to guarantee one attendance record per employee per attendance day. Department and join-date indexes support headcount queries; date-first and employee/date indexes support attendance filtering, sorting and monthly analytics. I would consider a dedicated status/date index if query plans show the current compound indexes are insufficient, but avoid redundant indexes because every write must maintain them.

2. **Punch-in race.** Both requests may reach the insert at once, but the unique `(emp_code, date)` index allows only one document. The winning request returns 201; the losing insert raises duplicate-key and becomes 409. A prior existence check alone cannot guarantee this.

3. **Ties.** The leaderboard uses competition ranking: equal totals share the same rank and the next rank is skipped. The limit is applied to rank, so a tie at the cutoff returns every employee at that rank. Employee code breaks display-order ties.

4. **Headcount.** Department reporting starts from employees and joins attendance, so people without logs remain represented. The joined-on date is compared with the period end, and headcount is based on distinct employees rather than log count.

5. **100x data.** I would validate plans and latency with production-like data, then consider pre-aggregated daily summaries, a background refresh strategy and operational metrics before adding more indexes blindly.
