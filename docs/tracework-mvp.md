# Tracework — MVP Specification

Tracework helps people manage job applications in one place. Users can enter applications manually or bring over an existing spreadsheet, view the same pipeline as a table or a Kanban board, and optionally connect Gmail to track replies and update stages.

## Pages and navigation

| Route                                  | Purpose                                                                                                         |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `/login`, `/signup`, `/reset-password` | Sign in, create an account, or reset a password                                                                 |
| `/applications`                        | Browse, search, filter, add, and import applications. Switch between Table and Board                            |
| `/applications/:id`                    | View or edit application details and history. Open as a side panel from the list or as a page via a direct link |
| `/profile`                             | View account name and email                                                                                     |
| `/profile/settings`                    | Manage pipeline statuses, the No Answer threshold, theme, and Gmail connection                                  |

The Applications page provides a **Review emails** entry when messages need attention. It opens a review panel for uncertain matches and suggested status changes.

Application and account routes require sign-in. Supabase Auth handles email and password authentication. Express verifies the session and checks ownership. Supabase Postgres stores the data, with row-level security for queries made on the user's behalf.

Resend is the provider for outgoing application emails. Password reset is the first use case in the MVP. Supabase Auth generates the password reset link and handles recovery. Resend delivers the reset email through Supabase Auth's custom SMTP configuration. The link returns the user to `/reset-password` in the same environment to set a new password.

## Application fields

- **Company** — required.
- **Position** — required.
- **Applied date** — required. A new form starts with today's date in the user's time zone. The user can edit it. Imports preserve the date in the spreadsheet.
- **Status** — defaults to Applied for new applications.
- **Job posting URL** — optional.
- **Source** — optional, such as LinkedIn, Djinni, or a company website.
- **Missing Requirements** — optional free text for job requirements the applicant does not meet or wants to work on.
- **Notes** — optional free text for other details about the application.

Users can search by company or position and filter by status or No Answer. The details view shows application fields and a chronological history of activity, including replies and manual or automatic status changes.

## Optional Excel import

Signed-in users can import existing applications from an `.xlsx` workbook, including one exported from Google Sheets. Manual entry is available as soon as the account is created.

1. Upload a workbook and select a sheet if it contains more than one.
2. Review suggested column mappings for Company, Position, Applied date, and Status. Map optional fields, including Missing Requirements, when available. Resolve ambiguous dates and map unfamiliar statuses to the account's configured stages.
3. Review the preview. Show the number of rows with possible duplicates at the top. Flag rows where company and position match an existing application in the current account, allowing for differences in case and extra whitespace. The date helps the user decide whether the records refer to different applications.
4. For each flagged row, show the matching record or records with company, position, applied date, and current status. Offer **Add as new** or **Skip**, with an **Apply to all similar rows** option for the selected action. Adding a row creates a separate record and preserves existing applications.
5. Confirm the import and show counts for added, skipped, and invalid rows. Evaluate No Answer immediately using the imported date and the account's threshold. A legacy No Answer or Ignore value that means “no reply” maps to Applied and the badge is recalculated.

## Pipeline and status settings

| Default active stage | Meaning                                                  |
| -------------------- | -------------------------------------------------------- |
| Applied              | Application sent                                         |
| In Review            | The employer confirmed the application is under review   |
| Screen Interview     | An initial recruiter or screening interview was arranged |
| Test Task            | A take-home or other test assignment was received        |
| Tech Interview       | A technical interview was arranged                       |
| Offer                | An offer was received                                    |

**Rejected** means the employer declined. **Withdrawn** means the applicant chose to leave the process. Both are closing statuses.

In profile settings, users can add active stages, edit status labels, and reorder active stages. The underlying status IDs remain stable when labels change, so existing applications and Gmail rules retain their meaning. Rejected and Withdrawn retain their closing behavior and dedicated drop zones if their labels are edited. An account can move an application between stages in the order its hiring process actually follows.

### No Answer

Each user controls the number of calendar days before the badge appears. The initial threshold is **14 days**, with a minimum of one. The badge appears for applications still at Applied without a substantive reply, using the user's time zone. Changing the threshold recalculates it for existing applications. A substantive reply detected in Gmail or recorded manually clears the badge. A routine acknowledgment can appear in the history while the badge remains eligible. Both views share the same No Answer filter.

### Table and Kanban

The table and board share the same application records and statuses. The table is optimized for scanning, with columns for company, position, applied date, status, and latest reply. On the board, active statuses are columns in the order chosen in settings. Closed applications are accessible through table filters.

The bottom **Rejected** and **Withdrawn** drop zones appear only while a card is being dragged. They stay visible while the board scrolls. When the card enters a zone, the zone highlights and visually expands while its drop area stays stable. The zones disappear when the drag ends or is canceled.

A completed drop updates the application and writes a history entry. Closing an application removes the card from the active board. If saving fails, the card returns to its previous stage and the app shows an error. The card menu supports the same actions by keyboard, and reduced-motion mode uses a static highlight.

## Optional Gmail monitoring

Show a **Connect Gmail to track replies** banner in Applications while Gmail is disconnected. The banner explains that Gmail can surface recruiter replies, build an application-specific email history, and update stages automatically. Users can also connect or disconnect their account in profile settings. Gmail authorization is separate from app sign-in and requests read-only access.

After connection, the backend periodically syncs messages. Known Gmail threads are linked to their application. For new threads, the app considers the company, position, and sender. AI classifies relevant replies, for example as a screening invitation, test task, technical interview, offer, or rejection. A clearly linked message with a clear classification can move the application to the corresponding status. Ambiguous matches or classifications go to the **Review emails** panel. Conflicts with a later manual change or a closed application also require review.

The application history combines relevant Gmail messages, user actions, and automatic changes in chronological order. It shows the sender, subject, timestamp, a snippet, and a link to the original email. An automatic status change records the source message and previous status, and the user can correct it. Sync is idempotent per Gmail account and message ID. Before a public launch, verify that Gmail access and processing by the chosen AI provider meet Google's restricted-scope and Limited Use requirements. [Google Workspace API user data policy](https://developers.google.com/workspace/workspace-api-user-data-developer-policy)

## Design and themes

The main workspace gives the table or board most of the screen. A shared toolbar holds search, filters, the Table / Board switch, and the Add application action. Keep the Gmail banner compact so users can get to their applications quickly. Selecting an application opens a side panel on wider screens, with company, position, and current status at the top, followed by application details and a timeline. Narrow screens can use the dedicated details page.

Use **shadcn/ui** for shared controls such as buttons, form fields, dialogs, menus, and the details panel. Set up shared theme tokens for page and card surfaces, borders, text, accent colors, and focus states. Light, dark, and monochrome are proposed options. Save the selected theme in profile settings and apply it across the app, including the table, Kanban, and Gmail review panel. The exact typography, palette, and density can be decided when designing the screens. [shadcn/ui theming](https://ui.shadcn.com/docs/theming)

Use short transitions for opening the details panel and signaling the active drop zone. dnd-kit handles drag behavior. During design, check that the active stage and recent replies are easy to spot in every theme, and that loading, empty, and error states stay readable.

## Technical foundation

| Area                        | Choice                                                                                                                                                                              |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend                    | React, TypeScript, Vite, React Router                                                                                                                                               |
| Rendering                   | Client-side rendering. The browser renders each routed screen and fetches data from Express. Configure the frontend host to serve the app shell on direct visits to frontend routes |
| UI and theming              | shadcn/ui, Tailwind CSS, shadcn/ui theme tokens                                                                                                                                     |
| Drag and drop               | dnd-kit                                                                                                                                                                             |
| Backend                     | Node.js, Express, TypeScript                                                                                                                                                        |
| Database and authentication | Supabase Postgres and Supabase Auth                                                                                                                                                 |
| Transactional email         | Resend for outgoing application emails. Supabase Auth uses Resend via custom SMTP for password recovery                                                                             |
| Tooling                     | Vite, Vitest, Oxlint. Oxfmt is optional                                                                                                                                             |

### Environments

| Environment | Purpose                                                                        |
| ----------- | ------------------------------------------------------------------------------ |
| Staging     | Test changes and verify releases with test accounts and data before deployment |
| Production  | Run the live application for users and their real data                         |

Production runs on the `tracework.surf` domain. Each environment has its own frontend and API deployment, Supabase project, and environment variables. Validate releases in staging before deploying them to production.

### Data entities

- `user` — the application user, identified by their Supabase Auth user ID.
- `user_setting` — the user's theme, time zone, and No Answer threshold.
- `user_pipeline_status` — a stage in the user's pipeline, with a stable ID, a configurable label, and a display order. Applications reference this stage by ID.
- `application` — a job application owned by a user.
- `application_event` — an entry in the application's history.
- `gmail_connection` — a user's Gmail connection.
- `gmail_message` — a synced Gmail message.

Test duplicate decisions and imported dates, status edits with existing applications, No Answer recalculation, and Gmail retries and uncertain classifications.

## Implementation order

1. Set up auth, password reset emails with Resend, routes, profile settings, application fields, and the table.
2. Add account-level status settings, the Kanban board, application history, No Answer, and themes.
3. Add optional Excel import with column mapping, a preview, and duplicate decisions.
4. Add Gmail connection, message history, the review panel, and AI-assisted automatic status changes.

## After the MVP

- Export applications.
- Build a mobile app.
- Automatically discover job openings.
- Add reports, pipeline statistics, and activity tracking.
- Build a browser extension to fill out job application forms.
- Scan résumés and assess how well a job matches the applicant's experience.
- Add follow-up reminders.
- Consider paid plans or a paywall later.
