# Changelog

All notable changes to the Paycor WISE Assistant plugin for Claude.

## 0.4.0 — first public release

- **Ask WISE from Claude.** Ask HR, payroll, benefits, time and employee-data questions in plain
  language, and Claude routes them to Paycor WISE (the `ask-wise` skill and the `ask_wise` tool).
  WISE answers only from data your own Paycor account is allowed to see.
- **Follow-up questions.** Claude writes each request so it stands on its own, for example by
  restating which employees or dates "they" or "last month" refers to. WISE receives only your
  current request, never your chat history.
- **Company selection.** If your account covers more than one company, Claude asks which one you
  mean before answering.
- **Changes only with your approval.** WISE makes a change only when you ask for it. Before any
  delete, or any other change that cannot be undone, WISE pauses for your approval: on WISE's
  approval card or, in apps that cannot show the card (such as Claude in Excel), in Claude's own
  confirmation prompt. Claude cannot approve on your behalf.
- **File documents into your Paycor records** (the `upload-to-wise` skill), for example "attach this
  safety certificate to employee Z". Accepts `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.txt` and
  `.csv` up to 10 MB. Asking Claude to read or summarise a file you attach to the chat uploads
  nothing.
- **Send your workbook to WISE from Claude for Excel** (the `send-excel-data-to-wise` skill). Claude
  confirms the file name once, sends every sheet in one step, and then tells WISE what to do with it.
- **Sign in with your own Paycor account.** Claude never sees your Paycor password.
