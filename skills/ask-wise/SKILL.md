---
name: "ask-wise"
description: "This skill should be used whenever the user asks an HR, payroll, benefits, time, or employee-data question that should be answered from their Paycor data — for example \"ask WISE\", \"check my PTO balance\", \"when is our next payroll deadline\", \"look up an employee's benefits\", \"how much overtime did my team log\", \"list the client ids I have access to\", or any follow-up to a previous WISE answer."
---

# Asking Paycor WISE

Route HR, payroll, benefits, time, and employee-data questions to the WISE agent via the `ask_wise`
tool. Follow the history and company rules below exactly — getting them wrong silently produces
answers about the wrong company or drops all conversational context.

If the user wants to **store a document in Paycor** (attach a certificate to an employee, file a
signed form), that is the `upload-to-wise` skill, not this one — it is a multi-step flow that ends in
an `ask_wise` call carrying a `messageMarker`. Reading or summarising a file the user attached to the
conversation needs no upload and no tool at all.

## Companies vs. clients

Companies and clients are **different** levels in Paycor, and their ids are **different** identifiers:

- A **company** (`companyId`, the ids in `structuredContent.companies`) is the top-level container
  that `ask_wise` is scoped to.
- A company **contains one or more clients**. A **client id** is a separate identifier that lives
  inside a company and is only known to WISE — it never appears in `structuredContent.companies`.

Never present company ids as client ids, or vice versa. When reporting, always label which is which
(e.g. "Company: Sales Demo (984202) → Client ID 323318").

For questions about clients (e.g. "list the client ids I have access to", "which clients can I see"):

1. Resolve the company first using the normal company-selection flow below. The company list is
   **not** the answer to a client question — it only determines which company to ask WISE about.
2. Call `ask_wise` with the chosen `companyId` and the user's client question, and report the client
   id(s) WISE returns, grouped under the company they belong to.
3. If the user has access to multiple companies and wants all their clients, ask WISE once per
   company (or offer to check the remaining companies) and present the results grouped per company.
   Do not assume clients in one company apply to another.

For questions about companies (e.g. "which companies can I see"), the `structuredContent.companies`
list is the answer — present it as companies, never as clients.

## Chat history

`ask_wise` is stateless: WISE retains nothing between calls, so `messages` is the only way it sees
prior turns.

1. Every call passes `messages`: an ordered array of `{ role, content }` entries, oldest first,
   ending with the current question (`role: "user"`). There is no `conversationId` — nothing is
   reused across calls.
2. Include only the prior turns needed to resolve a back-reference in the current question (e.g.
   "what about last month?" or "and for my other employees"). Omit turns unrelated to the current
   question rather than replaying the entire session.
3. Roles must alternate meaningfully: use `"user"` for the end user's own turns and `"assistant"`
   for WISE's own prior replies (its `content` text, not files/views). Never fabricate a turn that
   didn't happen.
4. If no prior turns are relevant, `messages` may be a single `user` entry with just the current
   question.

## Company selection

1. If the user has already said which company they mean, pass its id as `companyId`.
2. Otherwise call `ask_wise` without `companyId` and inspect the result:
   - `structuredContent.status == "needs_company_selection"` → the user has access to more than one
     company. **This status is a request to present an interactive company picker — never a plain
     chat message, never an error, and never something to just report as text.** You MUST call the
     `AskUserQuestion` tool for this — never type the question or the company list as plain chat
     text, and never surface it to the user as a failure. This is not optional and not a stylistic
     choice: writing it as prose forces the user to type a reply back manually, which defeats the
     point of this rule. Before writing anything else in this turn, call `AskUserQuestion` built from
     `structuredContent.companies`:
     - `question`: "Which company would you like to use?"
     - `header`: something short like "Company"
     - `options`: one per company, `description` set to its id (e.g. "Id: 929462").
       **`label` must be unique across all options** — `AskUserQuestion` rejects the call outright
       if two options share a label, and that rejection is what can look like a generic "failed to
       show" error to the user instead of the picker. Companies frequently share a display name
       (e.g. two entries both named "Sales Demo"), so always disambiguate by appending the id to the
       label, e.g. `"Sales Demo (929462)"`, rather than using the raw company name as-is. Do this
       unconditionally, even when the names look unique at a glance, so a fresh set of companies
       never trips the same validation error.
     Do this exactly the same way every single time this status appears, with no exceptions, no
     paraphrasing the question in prose instead, and no summarizing the options in a sentence first.
     If `AskUserQuestion` is not currently listed among your available tools, that does NOT mean it
     is unavailable — many hosts (including Claude Cowork) load some tools lazily. Before falling
     back to plain text, call `ToolSearch` (or whatever tool-discovery mechanism your host exposes)
     with a query like `"select:AskUserQuestion"` or `"AskUserQuestion"` to load it, then call it
     normally. Only fall back to a plain text list (one line per company, `Name (Id)`, followed by
     asking exactly "Which company would you like to use?") if, after actually attempting this
     discovery step, `AskUserQuestion` is confirmed genuinely unavailable in this environment — never
     skip straight to plain text just because it seems faster or more natural to just ask, and never
     show the user a raw tool error instead of one of these two presentations.

     This applies regardless of how the question was phrased, including requests to *enumerate*
     companies or clients (e.g. "list the client ids I have access to", "which companies can I
     see"). Do not treat `structuredContent.companies` as the final answer and read it back as text.
     The picker shows **companies**, not clients (see "Companies vs. clients"): for a company
     question it is how the company ids are shown; for a client question it only selects the
     company, and the client ids come from the follow-up `ask_wise` call. Never relabel company ids
     as "client ids".
   - Any other status → the company was resolved automatically; continue.
3. If the `companyId` passed turns out not to be one of the user's companies, `ask_wise` returns
   `structuredContent.status == "invalid_company_selection"` — this is not an error. Present it the
   same way as `needs_company_selection` above (via `AskUserQuestion` when available, with the same
   unique-label rule).
4. Once the user picks a company, call `ask_wise` again with the chosen `companyId` and the full
   `messages` history.
5. `structuredContent.status == "no_active_seat"` means the user has no active WISE seat in any of
   their companies. Tell them a WISE seat must be assigned to them and stop; do not retry.
6. Never guess a `companyId`, and never carry one over from a different user or session. Ids outside
   the caller's own companies are rejected by the server. Never pass a client id as `companyId`.

## Interpreting results

- `content` holds the human-readable answer. Report it as WISE's answer, not as your own analysis,
  and do not re-derive or recompute numbers from it.
- Attached files and views arrive as extra text blocks prefixed `[Attached file: …]`. Surface them as
  attachments from WISE rather than folding them into prose.
- `structuredContent` carries a generated `conversationId` (for support/log correlation only — do
  not reuse it on later calls, except as `pendingConversationId` below), `status`, and company data.
  Read those from it; do not parse the text blocks for them.
- Answers concern real employee and payroll data. Repeat what WISE returned; do not speculate beyond
  it, and do not fill gaps with assumptions about pay, benefits, or employment status.

## Citations (`sourceLink`)

Source links are rendered only by the `ui://` view's "Sources" widget — never by you.

1. If `structuredContent.sourceLink` contains a link, the view renders it. Do nothing.
2. If it is empty, there is no source. Say nothing about sources.
3. In either case, never write `sourceLink` (or any source link, "Sources:" line, or citation) in
   your reply — not as markdown, not as a plain URL, not paraphrased.

## Don't re-render data the host already displayed

`structuredContent.table` (title/headers/rows) drives its own widget on hosts that support MCP Apps
`ui://` views — Claude Cowork renders one automatically for every `ask_wise` result that includes it,
with no action needed from you. Some hosts also carry the same data as a markdown table inside
`content` (a fallback for hosts that don't render `structuredContent`/`ui://` at all) — that fallback
copy is for the host to display, not an invitation to output the same rows a second time.

1. If a table widget was rendered for this result (i.e. `structuredContent.table` is non-empty), do
   not also type the same rows out as a markdown table or bulleted list in your reply. That's a
   second, redundant copy of data the user can already see.
2. Your text reply should reference the table conversationally ("Here are the 20 employees — see
   the table above") and add only what it doesn't already show: a one-line summary, a caveat, or
   suggested follow-ups (WISE's `suggestions`, if present).
3. This applies per-result, not globally — if the host doesn't render `structuredContent` at all (rare;
   check whether earlier results in this session showed a widget), falling back to a markdown table in
   your own reply is reasonable so the data is visible somewhere.

## Decisions (`decision_required`)

Some requests change data (deleting an asset, editing report columns) or need an answer only the user
can give. For those, WISE pauses and `structuredContent.status` is `"decision_required"`. The note in the
result says which of the two flows below applies.

### With the decision card (the note says a decision card is shown)

The WISE view shows the user a decision card (Approve/Deny, a column list, or questions), and **only the
user can answer it there**.

1. End your turn without writing anything about the request. The card already tells the user what WISE
   needs, and WISE's reply to the decision appears on the card, so any text you add would sit between
   the card and that reply and read out of order. Do not call
   `ask_wise` again for the same request, and never say the change was made — it has not been yet.
2. You have no tool that approves anything, and you must not ask the user to approve in chat. The
   card is the only place a decision is made.
3. After the user decides on the card, WISE's reply usually appears on the card itself and you are told
   the outcome and the reply. Acknowledge it briefly without repeating what the card shows, and do not
   re-run the request. If instead the card posts the outcome in chat as the user's message (for example
   "I approved: Update Date Of Birth"), that message is not a new question: call
   `get_wise_decision_result` with `conversationId` set to the paused result's
   `structuredContent.conversationId`, then treat its result exactly like an `ask_wise` result (the
   same rules above apply). If it is `decision_required` again, follow these steps for the new card.
4. If the user replies in chat instead of using the card (for example they change their mind or ask
   something else), call `ask_wise` with their reply as the current question and
   `pendingConversationId` set to the paused result's `structuredContent.conversationId`. That
   declines the pending request — it is never approved this way — and WISE answers the reply.

### Without the card (the note says this app cannot show WISE's decision card)

The user decides in chat instead, and you relay their answer.

1. Before writing anything else, ask the user with `AskUserQuestion` built from the note: its question
   and its options exactly as listed. If `AskUserQuestion` is not loaded, discover it first (`ToolSearch`
   with `"select:AskUserQuestion"`), as in company selection. Ask in plain text only if it is confirmed
   unavailable.
2. Once the user has answered in this turn, call `submit_wise_decision` with the ticket from the note and
   the decision the note maps their answer to. Never pick an option for the user, never submit before
   they answered, and never reuse an earlier answer for a new decision.
3. If the user instead changes the request, abandons it or asks something else, call `ask_wise` with
   their reply and `pendingConversationId` set to the conversationId the note gives, instead of
   submitting. That declines the pending request and WISE answers the reply.
4. Treat the `submit_wise_decision` result exactly like an `ask_wise` result. If it is
   `decision_required` again, start over at step 1 for the new decision.
5. Never say the change was made until a result says so.

## Errors

- `isError: true` with an authentication message → tell the user to re-authenticate the WISE
  connection. Do not retry in a loop.
- `isError: true` about privileges → the user's Paycor account lacks the access this tool requires.
  Re-authenticating will not fix it; their Paycor administrator needs to grant it. Say so plainly.
- A timeout → a WISE run is bounded server-side. Suggest a narrower or more specific question rather
  than retrying the same one unchanged.
