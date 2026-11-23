# MCP vs CLI for data analysis: follow-along guide

This guide goes with the video comparing two ways to connect Claude Code to Metabase: the **MCP server** and the **CLI**. It has every command and prompt from the demo, in order, so you can repeat it yourself.

You will ask the same question of the same data through both routes, save the result as a Metabase question each time, and compare what happened.

## The idea in one picture

- **The MCP is a menu.** It gives Claude Code a fixed list of Metabase actions (21 in the version we tested), such as `search`, `execute_query`, `run_saved_question`, `question_write` and `dashboard_write`. Claude can do those things and nothing else.
- **The CLI is a keyboard and a manual.** The `mb` command is a program Claude runs in your terminal the same way you would. Its help screen lists nearly 40 command groups. The "manual" is a short skill (`metabase-cli`, about 310 tokens in the skills list) that teaches Claude how to use it.
- The MCP needs no separate manual, because the server hands Claude its own instructions when it connects.

Note that 21 counts actions in a menu and 40 counts command groups. They are different kinds of count, so a bigger number does not mean a better tool.

## What you need

- Claude Code installed
- A Metabase instance you can sign in to (this guide uses the Sample Database that comes with Metabase)
- For the MCP: an admin to turn the server on
- For the CLI: Node.js 20.6 or later, Metabase 58 or later, and Metabase 63 or later for the browser login

Replace `https://your-metabase-url` in every command below with your own Metabase address.

## Part 1: Setup

### Route A: the MCP

1. In Metabase, an admin goes to **Admin, then AI, then MCP** and turns the server on.
2. Register it with Claude Code:

```bash
claude mcp add --transport http metabase https://your-metabase-url/api/metabase-mcp
```

3. Start Claude Code and sign in through your browser when prompted. There is no API key and nothing else to install.
4. Check the connection inside Claude Code:

```
/mcp
```

You should see the `metabase` server connected, with its list of tools.

### Route B: the CLI

1. Install the CLI (needs Node.js 20.6 or later):

```bash
npm install -g @metabase/cli
```

2. Log in through your browser:

```bash
mb auth login --url https://your-metabase-url
```

3. Check that it worked:

```bash
mb auth status
```

4. Install the skill that teaches Claude Code how to use `mb`. Follow the install instructions in the [Metabase CLI docs](https://www.metabase.com/docs/latest/installation-and-operation/metabase-cli). If the one-line install fails for you, cloning the skills repo by hand first is a route that worked in our test:

```bash
git clone https://github.com/metabase/agent-skills.git /tmp/agent-skills-test
npx skills add /tmp/agent-skills-test --skill metabase-cli -a claude-code
```

### Setup compared

| | MCP | CLI |
| --- | --- | --- |
| Admin step in Metabase | Yes, turn the server on | No |
| Install something | No | Yes, Node.js and an npm package |
| Teach Claude how to use it | Automatic, the server sends instructions | Install the `metabase-cli` skill |
| Sign in | Browser | Browser |

## Part 2: The demo

The question for both routes:

> How many orders are there in each product category in the Sample Database, highest first?

Paste every prompt below **inside Claude Code**, not at the plain Terminal prompt. If you see a `quote>` prompt, you pasted into Terminal by mistake.

Claude Code may run shell commands and tool calls without asking you first (auto mode). Watch what it is doing, and turn that setting off if you want to approve each step.

For a fair comparison, start a fresh Claude Code session for each round.

### Round 1: the MCP

**1. Check the starting context.** This shows what Claude Code already carries before you ask Metabase anything.

```
/context
```

**2. Look at the menu.** Open the `metabase` server and look at the list of tools, then press Escape.

```
/mcp
```

**3. Read.** Name the tool in the prompt so Claude does not choose for you.

```
Use the Metabase MCP (the "metabase" server, not the CLI) to tell me how many orders there are in each product category in the Sample Database, highest first. Don't save anything yet.
```

**4. Write.** This creates a collection and saves the question inside it.

```
Now use the Metabase MCP to create a new collection called "Video Demo", then save this as a question inside it named "Demo - orders by category": order count by product category, sorted highest first inside the query itself. Tell me the link when it's done.
```

**5. Check it in Metabase.** Open the link Claude gives you, open the saved question, and confirm it runs and returns the same rows. Always verify an answer outside the chat.

**6. Check the context after the round.**

```
/context all
```

### Round 2: the CLI

Do this after Round 1, because the write prompt saves into the "Video Demo" collection that Round 1 creates.

**1. Start a fresh session.**

```
/exit
```

```bash
claude
```

**2. Check the starting context.**

```
/context
```

**3. Look at the manual.** Find `metabase-cli` in the skills list, then press Escape.

```
/skills
```

**4. Look at the keyboard.** This prints the CLI's command groups.

```
mb --help
```

**5. Read.**

```
Use the Metabase CLI (the mb command and the metabase-cli skill), not the Metabase MCP, to tell me how many orders there are in each product category in the Sample Database, highest first. Don't save anything yet.
```

**6. Write.**

```
Now use the Metabase CLI (not the MCP) to save this as a question named "Demo CLI - orders by category" in the existing "Video Demo" collection, sorted highest first inside the query itself. Tell me the link when it's done.
```

**7. Check it in Metabase.** Open the "Video Demo" collection and compare both saved questions side by side.

**8. Check the context after the round.**

```
/context all
```

## Part 3: What to expect

Your exact output will differ, because Claude makes its own choices. In our test run:

**The answer** was the same from both routes. In the Sample Database, order counts by product category came out as Widget 5,061, Gadget 4,939, Gizmo 4,784 and Doohickey 3,976, for 18,760 orders in total.

**The routine** was the same too: read the rules, find the IDs, run the query. The MCP read Metabase's glossary, browsed the databases, tables and fields, then ran the query. The CLI read its own manual, listed the databases, looked up the table fields, then ran the query.

**What got saved** differed. The MCP saved a table with a description. The CLI saved a bar chart with no description. Claude chose the chart type and description on its own, and you can ask for either one in your prompt. The CLI also left a small query file in a hidden folder inside the working directory, because it works with files on your computer.

**Context use** (one run each, so treat it as an example and not a benchmark):

| Moment | MCP session | CLI session |
| --- | --- | --- |
| Starting context | 35.7k tokens | 35.6k tokens |
| After the round | 78.4k tokens | 76.0k tokens |
| Where the growth went | Tool definitions (8.6k for the six tools used), messages 34.1k | No MCP tools loaded, messages 40.3k |

The two ended 2.4k tokens apart, which is too close to call from one run each. The shape is different, though. The MCP pays for the definitions of the tools it uses. The CLI pays inside the conversation, because it prints its own instruction manuals and command output into the messages.

## Part 4: Access and permissions

Both routes sign you in through the browser as you, so both act with your Metabase permissions.

- **CLI:** the approval screen warns that it gets full access to Metabase as your user account. Anything you can do, the CLI can do.
- **MCP:** according to the Metabase docs, its tools are gated by five scopes: read content, write content, run queries, run raw SQL, and create alerts and subscriptions. A scope never grants more than your own Metabase permissions. (We read this in the docs and did not test the scopes in the demo.)

## Part 5: Which one should you use?

- The **MCP** is easier to set up: an admin toggle, one command and a browser sign-in. It is a little less flexible in what it lets you do. For example, it has no export tool among its 21 tools, though you can still ask Claude Code to write returned rows to a file.
- The **CLI** gives you more control, but takes more to set up: Node.js, an npm install, a login and a skill.
- Both did both jobs in this demo, so we did not find something one can do and the other cannot. The CLI's command list does include groups the MCP has no tool for, such as git-sync and transforms. We did not test those, and the Metabase docs say some CLI command groups need a Pro or Enterprise plan.

The right choice follows the job and the person. If you want the quickest route to asking and saving, start with the MCP. If you are comfortable in a terminal and want more control, try the CLI.

## Troubleshooting

- **`mb: command not found` or npm install fails:** check that Node.js 20.6 or later is installed, and that your npm global folder is writable by your user.
- **Browser says login is complete but `mb auth status` says you are not authenticated:** run `mb auth login --url https://your-metabase-url` again and check the status command before moving on.
- **The CLI round fails because "Video Demo" does not exist:** run Round 1 first, or create a collection with that name in Metabase.
- **Rerunning the demo and the MCP round behaves oddly:** delete or rename the leftover "Video Demo" collection in Metabase first, since the MCP prompt creates it.

## Docs

- [Metabase MCP server docs](https://www.metabase.com/docs/latest/ai/mcp)
- [Metabase CLI docs](https://www.metabase.com/docs/latest/installation-and-operation/metabase-cli)
