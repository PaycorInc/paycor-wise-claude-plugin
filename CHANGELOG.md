# Changelog

All notable changes to the Paycor WISE plugin for Claude.

## Unreleased

- **Send your workbook to WISE from Claude for Excel.** Claude asks once, showing the file name it
  will be stored under: Send, Rename or Cancel. It then sends every sheet of the workbook in one step
  (the "Data Sources" sheet Claude adds is always left out), and Paycor stores them as one Excel file
  under your own account, without leaving Excel or saving a file. Claude then tells WISE what to do
  with it (for example "import the Timesheet sheet as time punches"); the company is chosen there, as
  for any other WISE request. Available in the Claude for Microsoft 365 add-ins only; Claude Desktop,
  Cowork, claude.ai and Claude Code are unchanged.
- Adds the `send-excel-data-to-wise` skill covering that flow: the whole workbook, one confirmation
  with the file name and Send / Rename / Cancel, one upload with all sheets, at most 3 retries, and the
  hand-over to `ask_wise`. No other skill changes.
- Version moves to `0.3.0`.

- **Approvals now work in Claude apps that cannot show the WISE decision card**, such as Claude in
  Excel. There, Claude asks you the same question in chat (Approve or
  Deny, which columns, or WISE's questions) and passes your answer back to WISE. Claude Desktop,
  Cowork and claude.ai keep the card exactly as before.
- `ask-wise` skill: adds the rules for deciding without the card — ask with `AskUserQuestion` first,
  submit only your answer through `submit_wise_decision`, and send any other reply to `ask_wise` with
  `pendingConversationId`. The card rules are unchanged.
- **Changes WISE makes on your behalf now ask for your approval first.** When a request would change
  data — or WISE needs you to choose report columns or answer a question — the WISE view shows a
  card with Approve/Deny (or the choices), and nothing happens until you decide there. Claude cannot
  approve on your behalf. Replying in chat instead declines the pending request.
- `ask-wise` skill: adds the `decision_required` rules — end the turn without writing about the request
  while the card is pending (so WISE's reply follows the card directly), never claim the change was
  made, and pass `pendingConversationId` when the user replies in chat instead of using the card.
- After you decide on the card, WISE's reply appears right under it. The connector can instead be set
  to show the reply in the chat below your decision; the `ask-wise` skill reads it with
  `get_wise_decision_result` in that case.
- `ask-wise` skill: source links are now shown only by the WISE view's "Sources" widget. Claude
  no longer writes a source link or "Sources:" line in its own reply.
- **You can now file documents into your Paycor records from Claude** — for example "attach this
  safety certificate to employee Z". Claude validates the file's name, type and size before anything
  is sent, uploads it directly to Paycor without the document's contents passing through the
  conversation, waits for Paycor to accept it, and then tells WISE what to do with it. Accepts
  `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.txt` and `.csv` up to 10 MB. Asking Claude to read or
  summarise a file you attach to the chat is unchanged and still uploads nothing.
- Adds the `upload-to-wise` skill covering that flow, so Claude follows the same upload, wait and
  reference steps every time instead of improvising them. Kept separate from `ask-wise` so it
  activates on document requests rather than on questions.
- `ask-wise` skill: points at `upload-to-wise` when the request is to store a document, so a file
  request is not silently handled as a question.
- Version moves to `0.2.0` to distinguish installed copies that include document filing from earlier
  builds that only answer questions.
- The connection now points at the production WISE environment by default, rather than the
  Quarterly test environment every installed copy previously pointed at regardless of which
  environment you actually use. Also removes a separate sign-in configuration value that could
  disagree with the connection address — sign-in now always matches the server you're connected to.
- `ask-wise` skill: adds guidance not to re-render `structuredContent.table` as a second markdown
  table in the reply when the host already renders it as a widget (e.g. Claude Cowork's MCP Apps
  `ui://` view) — was causing every table result to appear twice on-screen.
- `ask-wise` skill: when a company you specify no longer matches one you have access to, Claude now
  asks you to pick again from your available companies instead of treating it as a connection or
  permission error.
- `ask-wise` skill: when you have access to more than one company, Claude now always presents the
  choice the same way — your companies listed by name with one consistent question — instead of
  sometimes varying the format.
- Adds `license`, `homepage`, and `repository` fields to `plugin.json`, and a top-level
  `LICENSE` file — required metadata for submitting the plugin through Anthropic's Connectors
  Directory portal.

## 0.1.0 — unreleased

Initial version. Not yet published to customers.

- Adds the WISE MCP server (`/mcp/wise-assistant/mcp`, environment-specific host) over streamable HTTP, authenticated with
  the user's own Paycor account via OAuth 2.1 with PKCE.
- Adds the `ask-wise` skill, which handles caller-supplied chat history (`messages`, stateless — no
  `conversationId` input) and company selection (`needs_company_selection`).
- Exposes one tool: `ask_wise` (read/write — can query and update HR/payroll data), including
  multi-company disambiguation (`needs_company_selection`) when the user has access to more than
  one company.
