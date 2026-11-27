# Paycor WISE for Claude

Paycor WISE Assistant connects Claude to Paycor’s workforce intelligence experience, giving HR administrators a faster, more intuitive way to work with HR, payroll, time, attendance, and reporting information. Using natural language, users can ask questions about their workforce, find answers to questions about HR functions, and receive guidance on supported administrative tasks, all without leaving Claude or switching between applications.

Save time by reducing context switching:

- Access workforce information and Paycor process guidance directly in Claude.
- Get answers and insights faster: Ask questions in plain language, retrieve authorized information, and summarize workforce trends or reporting insights.
- Simplify HR and payroll work: Receive guidance for supported processes and complete common administrative workflows more efficiently.
- Turn information into understanding: Use Claude to summarize and explain available Paycor information, such as reports and process guidance, and help draft or transform HR-related content based on accessible information.
- Maintain appropriate controls: Responses and available actions reflect the user’s Paycor account and role permissions, company configuration, and enabled services.


## What you can ask

Once connected, ask naturally and Claude will route the question to WISE:

- "Which employees were hired in the last 30 days?"
- "How many employees are in the Sales department?"
- "How do I add a new hire in Paycor?"

Follow-up questions work — ask "and for the month before?" and Claude restates what it refers to
in the request it sends. WISE receives only your current request, never your chat history.

## What you need before you start

You need to have an active Paycor account, an assigned seat to access Paycor WISE Assistant, and the appropriate permissions for the Paycor data, workflows, and services you want to use through Claude. Available responses and actions depend on your Paycor role, company configuration, enabled services, and data-access permissions. If Claude tells you that you lack access, your administrator needs to grant it in Paycor's role and permission settings — signing in again will not help. Contact your Paycor administrator or Paycor Services and Support for help enabling this permission.

## Installing

The plugin appears in Claude as **Paycor WISE Assistant**.

### From Claude's plugin directory

1. In Claude Desktop or on claude.ai, open **Settings** and, under **Customize**, select
   **Plugins**.
2. Open the **Discover** tab and find **Paycor WISE Assistant**.
3. Click **Add**. The first time Claude uses WISE, you are asked to sign in (see below).

In Claude Cowork, open the **Cowork** tab first, then open **Customize > Plugins**. On Team and
Enterprise plans, your organization's Claude owner can also share the plugin with you directly.

### In Claude Code

Add this repository as a plugin marketplace, then install the plugin:

```
/plugin marketplace add PaycorInc/paycor-wise-claude-plugin
/plugin install paycor-wise@paycor-wise-marketplace
```

### As a connector only

To connect WISE without the plugin's skills, open **Settings > Customize > Connectors**, click
**+ Add**, then **Add custom connector**, and enter `https://apis.paycor.com/mcp/wise-assistant/mcp`.
Without the skills, Claude is less likely to route HR and payroll questions to WISE on its own.

See [Use plugins in Claude](https://support.claude.com/en/articles/13837440-use-plugins-in-claude)
and [Get started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
for details.

## Signing in

The first time Claude uses WISE, a browser window opens for you to sign in to Paycor. Approve the
request, and the browser shows a short confirmation page you can close.

You are signing in to Paycor's own sign-in page — Claude never sees your Paycor password. Your sign-in
stays valid for a while afterwards and refreshes silently, so you should rarely need to repeat it.

## What Claude can and cannot see

- Claude can only reach the companies **your own Paycor account already has access to**. It cannot see
  other organizations' data, and it cannot see companies you personally lack access to.
- If your account covers more than one company, Claude asks which one you mean before answering.
- Your question and WISE's answer pass through Claude in order to be understood and presented —
  the same as anything else you type in a conversation.
- For Paycor's full data handling and privacy practices, see the
  [Paycor Privacy Policy](https://www.paycor.com/privacy-policy/).

## Tools this plugin adds

| Tool | Visible to Claude | What it does |
|---|---|---|
| `ask_wise` | Yes | Sends your current request to WISE and returns the answer. WISE makes a change (e.g. updating employee information) only when you ask for it. Before any delete, or any other change that cannot be undone, WISE always pauses for your approval — on WISE's approval card or, in apps that cannot show the card, in Claude's own confirmation prompt — and never makes such a change without it. If your account has access to more than one company, `ask_wise` asks which one you mean before answering. |
| `create_wise_upload_ticket`, `upload_wise_file`, `get_wise_file_status` | Yes | Upload a document you want filed in your Paycor records. |
| `upload_wise_workbook`, `get_wise_workbook_status` | Yes (Claude for Excel) | Send the open workbook to WISE. |
| `get_wise_decision_result` | Yes | Reads WISE's reply after you decided on the approval card. |
| `submit_wise_decision` | Yes (only where the card can't be shown) | Submits the answer you gave in Claude's confirmation prompt. |
| `resolve_wise_decision` | No (used by the approval card only) | Supports the decision card: submits your Approve/Deny or answer click. |
| `get_wise_decision_status` | No (used by the approval card only) | Supports the decision card: shows whether a card was already decided when it is shown again. |

`ask_wise` can read and write data depending on what you ask, so Claude will confirm before
calling it. If you would rather not approve each use individually, your administrator can
pre-approve it in your organization's Claude settings.

## If something goes wrong

| What you see | What to do |
|---|---|
| Asked to sign in repeatedly | Sign in once more; if it keeps recurring, contact support with the details below |
| "You do not have access" | Your Paycor administrator needs to grant the required permission |
| Sign-in page never opens | Restart Claude and try again. In Claude Code, another program may be using the local port needed for sign-in |
| "Cannot connect" | Check that your network allows `apis.paycor.com`; your IT team may need to allow it |

When contacting support, include: this plugin's version (below), the error text you saw, roughly when
it happened, and whether sign-in had succeeded.

**Support:** [Paycor Services and Support](https://www.paycor.com/services-support/)

## Privacy & Terms

- [Paycor Privacy Policy](https://www.paycor.com/privacy-policy/)
- [Paycor Terms of Use](https://www.paycor.com/terms-of-use/)

## Version

0.5.0 — see [CHANGELOG.md](CHANGELOG.md).
