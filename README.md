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

Follow-up questions work — "and for the month before?" stays in the same conversation.

## What you need before you start

You need to have an active Paycor account, an assigned seat to access Paycor WISE Assistant, and the appropriate permissions for the Paycor data, workflows, and services you want to use through Claude. Available responses and actions depend on your Paycor role, company configuration, enabled services, and data-access permissions. If Claude tells you that you lack access, your administrator needs to grant it in Paycor's role and permission settings — signing in again will not help. Contact your Paycor administrator or Paycor Services and Support for help enabling this permission.

## Installing

The plugin appears in Claude as **Paycor WISE Assistant**.

### From Claude's plugin directory

1. In Claude Desktop or on claude.ai, open **Settings** and, under **Customize**, select
   **Plugins**.
2. Open the **Discover** tab and find **Paycor WISE Assistant**.
3. Click **Add**. The first time Claude uses WISE, you are asked to sign in (see below).

In Claude Cowork, open the **Cowork** tab first, then open **Customize > Plugins**.

### From a plugin file

If you were given the plugin as a file (for example `Paycor WISE Assistant.zip`), open
**Settings > Customize > Plugins** and upload it there. On Team and Enterprise plans, your
organization's Claude owner can also share the plugin with you directly.

### As a connector only

To connect WISE without the plugin's skills, open **Settings > Customize > Connectors**, click
**+ Add**, then **Add custom connector**, and enter the URL from the `url` field of this
package's `.mcp.json` file. Without the skills, Claude is less likely to route HR and payroll
questions to WISE on its own.

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

| Tool | What it does |
|---|---|
| `ask_wise` | Sends your question to WISE and returns the answer. WISE may also make changes on your behalf (e.g. updating employee information) when you ask it to. If your account has access to more than one company, `ask_wise` asks which one you mean before answering. |

`ask_wise` can read and write data depending on what you ask, so Claude will confirm before
calling it. If you would rather not approve each use individually, your administrator can
pre-approve it in your organization's Claude settings.

## If something goes wrong

| What you see | What to do |
|---|---|
| Asked to sign in repeatedly | Sign in once more; if it keeps recurring, contact support with the details below |
| "You do not have access" | Your Paycor administrator needs to grant the required permission |
| Sign-in page never opens | Another program may be using the port Claude needs for sign-in; restart Claude and try again |
| "Cannot connect" | Check that your network allows the host in the `url` field of this package's `.mcp.json`; your IT team may need to allow it |

When contacting support, include: this plugin's version (below), the error text you saw, roughly when
it happened, and whether sign-in had succeeded.

**Support:** [Paycor Services and Support](https://www.paycor.com/services-support/)

## Privacy & Terms

- [Paycor Privacy Policy](https://www.paycor.com/privacy-policy/)
- [Paycor Terms of Use](https://www.paycor.com/terms-of-use/)

## Version

0.3.0 — see [CHANGELOG.md](CHANGELOG.md).
