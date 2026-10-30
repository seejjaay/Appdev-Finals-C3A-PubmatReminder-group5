# PubmatReminder

**An Automated Publication Material Deadline Reminder System Using UiPath**

A project submitted to the College of Information Technology and Engineering in partial fulfillment of the requirements for the course **APPDEVT (Applications Development and Emerging Technologies)**.

**Submitted to:** Ms. Arabelle Balita

**Submitted by:**

- Lantin, Marcus Ireneo P.
- Rosales, CJ Nicko B.
- Ruiz, Zachary Scott D.
- Pecho, Clark Alexander Y.
- Pilay, Justine M.

---

## Overview

Publication materials ("pubmats") such as brochures, banners and videos each have an owner and a deadline. Tracking them in a spreadsheet works only if someone checks it. PubmatReminder is a UiPath process that does the checking: it reads the pubmat tracker, finds items that are due soon and not yet marked **Done**, and emails the assigned person a reminder.

## How it works

The whole flow lives in [Main.xaml](Main.xaml) ("Pubmat Deadline Reminder"):

1. **Read the tracker.** Downloads the Google Sheet as CSV (it never edits the sheet) and finds the *Deadline* column.
2. **Check each row.** For every pubmat it parses the deadline (for example `Oct. 22 (Thu)`) and skips the row if the deadline is missing or the status is already Done. Bad rows are logged and skipped instead of stopping the run.
3. **Decide.** If the deadline is within 7 days, using the current date in Manila time, the row needs a reminder.
4. **Send.** It emails the assignee through Gmail SMTP (port 587, SSL). The message greets them by first name, names the event, says when it is due (and how many days are left), and includes any notes from the sheet.
5. **Log.** Each sent or skipped row is logged, and a summary with the number of reminders sent is written at the end.

The run fails with a clear message if no sheet ID is provided or the Deadline column can't be found.

### Tracker format

The sheet is expected to have these columns (see [pubmats.csv](pubmats.csv) for a sample):

| Event | Notes | Assigned to | Deadline | Pub Mat Status |
| --- | --- | --- | --- | --- |
| Brochure | | assignee@example.com | Oct. 23 (Fri) | Not done |

> The emails in the sample files are placeholders.

### Arguments

| Argument | Purpose |
| --- | --- |
| `in_SheetId` | ID of the Google Sheet that holds the tracker |
| `in_SheetName` | Name of the tab to read |
| `in_CredentialTarget` | Name of the Orchestrator credential asset holding the Gmail login |

## Setup

1. **Share the sheet** so the process can read it (view access is enough).
2. **Create a Gmail App Password** for the sending account. A normal account password won't work with SMTP.
3. **Store it as a credential asset** in Orchestrator under the name passed as `in_CredentialTarget`. The default is `PubmatReminder.Gmail`. No password is kept in this repository.
4. **Set the arguments** `in_SheetId` and `in_SheetName` for your own tracker.
5. Open the project in UiPath Studio and run `Main.xaml`, or publish it and schedule it in Orchestrator to remind people automatically each day.

## Tech stack

- UiPath Studio, Windows target, VB.NET expressions
- Activity packages: `UiPath.System.Activities`, `UiPath.Mail.Activities`, `UiPath.Excel.Activities`, `UiPath.Credentials.Activities`
- Google Sheets (as the data source) and Gmail SMTP (for delivery)

## Project layout

| Path | What it is |
| --- | --- |
| `Main.xaml` | The workflow (entry point) |
| `project.json` | UiPath project manifest |
| `pubmats.csv`, `pubmats.xlsx` | Sample pubmat trackers |
| `.objects/`, `.settings/`, `.project/` | Studio project metadata |
