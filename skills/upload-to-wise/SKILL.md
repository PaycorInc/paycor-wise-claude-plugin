---
name: upload-to-wise
description: >
  This skill should be used whenever the user wants to put a document INTO their Paycor
  records — for example "attach this certificate to employee Z", "upload this signed form
  to Paycor", "file this W-4 against my profile", "add this spreadsheet to the employee's
  documents", or any follow-up about a file that is being or has been uploaded. Do NOT use
  this skill to read, summarise, or answer questions about a file the user attached to the
  conversation — that needs no upload at all.
---

# Uploading a document to Paycor WISE

Filing a document is a **four-step flow across three tools**, not a single call. Each step depends on
a value produced by the previous one, and the most common failure is skipping step 2's response body.

## First: is this actually an upload?

Only upload when the user wants the document **stored in Paycor**: attached to an employee, filed
against a profile, added to their records.

If they want you to read, summarise, extract from, or answer questions about a file they attached to
the conversation, **do that yourself from the attachment**. Uploading is slower, changes their
records, and is not what they asked for.

## Pick the right tool

| Tool | Use when |
|---|---|
| `create_wise_upload_ticket` | **Default.** You can run a shell command. |
| `upload_wise_file` | Only when you cannot run shell commands. |

Prefer the ticket whenever you have a shell. `upload_wise_file` carries the file inline as base64,
which you have to emit character by character — roughly two minutes for a 10 KB file, and it becomes
unusable well before 1 MB. The ticket costs the same handful of characters no matter how big the file
is.

Allowed types: `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.txt`, `.csv`. Limit: 10 MB.

## The flow

### 1. Get a ticket

Call `create_wise_upload_ticket` with `fileName` (including extension) and `sizeBytes`. You get back
an `uploadUrl`.

**It does not return a `fileId`.** That is deliberate and it is the step people get wrong — see
below.

### 2. PUT the bytes, then read the response

```bash
curl --fail-with-body -X PUT --data-binary @/path/to/file "<uploadUrl>"
```

Send the **raw body**, not a multipart form.

> **The `fileId` comes back in this response, not from step 1.** So does the `messageMarker`. Read
> them out of the curl output. If you skip this, you have nothing to poll with and the upload is
> effectively lost — the file is stored but you cannot refer to it.

The `uploadUrl` works **once** and expires within minutes. Never reuse it, never save it, never print
it to the user. If it expires, go back to step 1 for a fresh one.

### 3. Poll until ready

Call `get_wise_file_status` with that `fileId` until it reports one of:

- `processing` — not usable yet. Keep polling, a couple of seconds apart. Expect this on the first
  poll or two; it is normal.
- `ready` — usable. Carries the `messageMarker`.
- `rejected` — terminal. Carries a `rejectionReason`. Tell the user why and stop. **Do not reference
  the file.**
- `not_found` — terminal. Unknown or expired id. Do not reference the file.

Paycor validates the document's contents after it arrives, so a file whose contents do not match its
extension (a renamed executable, a corrupt PDF) is accepted at upload and comes back here as
`rejected`. That is expected — relay the `rejectionReason` rather than retrying.

### 4. Tell WISE what to do with it

Only once `ready`, call `ask_wise` with the `messageMarker` included **verbatim** in your `messages`,
alongside the instruction — e.g. "attach this to employee Z".

`ask_wise` is stateless, so **re-include the same `messageMarker` in every later call that refers to
the same file.** Drop it and WISE no longer knows which document you mean. See the `ask-wise` skill
for the history and company rules that apply to this call.

## Company selection

Both upload tools take an optional `companyId` and follow exactly the same disambiguation contract as
`ask_wise` — including presenting the choice via `AskUserQuestion`. See the `ask-wise` skill's
"Company selection" section and follow it identically.

One difference worth noting: the result names **the tool you were part-way through**, not `ask_wise`.
Retry that tool with `companyId` set, then carry on from where you were. Do not switch to `ask_wise`.

## Errors are final

`unsupported_type`, `file_too_large`, `invalid_file_name`, `invalid_content` and `ticket_expired` are
all terminal. Report the reason to the user and stop.

Do **not** retry the same file. Nothing about it will have changed, and on the inline path a retry
costs the entire emission budget again to receive the identical answer. `ticket_expired` is the one
exception in spirit: the file is fine, the ticket is not — mint a new ticket and start from step 1.

A `rejected` status from step 3 is terminal in the same way.
