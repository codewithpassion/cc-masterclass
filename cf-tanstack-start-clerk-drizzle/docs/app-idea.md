## Worklog Tracking System

We are building a simple system to track users' worked hours for a company with a 10K user-base.
Data from the last 10 years will be migrated.

Tech Stack & Setup

- Frontend: TanStack Router, Tailwind, shadcn
- Backend: Bun, Hono, Cloudflare Workers
- Database: Cloudflare D1 with Drizzle ORM
- Authentication: Clerk

Access Control

- Anonymous users
  - can only view the login and a help page (use lorem ipsum).
- Registered users
  - Log in
  - Create worklogs (multiple logs per day)
  - List their worklogs
    - data should be grouped by days
  - we display total hours of a user for a given day
  - beside the hours we should have a UT (undertime) sign if worked less than minimum and an OT (overtime) sign if worked more than maximum!
  - add filtering by OT or UT
    - add paging (entries)
    - use default sorting (by day) - most recent first
- Admin
  - List all users
  - Edit users' min/max daily hours
  - List each user's worklog (same as user's personal worklog, with one major detail)
    - we need to show user's name too
  - we sort by day, then by user name

Data Model

- User: name, dailyMinHours, dailyMaxHours
- Worklog: date, workedHours, taskId (optional), description (optional)

