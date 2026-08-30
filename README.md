# ContractClarity

ContractClarity reads a contract in your browser. You give it a PDF, a Word file, or pasted text, and it finds the terms that matter and points out the clauses you probably want to look at twice. If you have an AI agent, it can pick up those findings through WebMCP and walk you through them. The contract stays on your computer.

## What the page does without an agent

When you load a file, the page reads it for you. It parses the document on your machine and finds the parties, the effective date, how long the term runs, the notice period, the payment terms, any dollar amounts, and the governing law. It also runs a set of rules that look for the clauses people miss: auto-renewal, termination for convenience, indemnification, a non-compete, an arbitration or jury waiver, penalties and late fees, one party's right to change the terms alone, IP assignment, and a liability section that never sets a cap. When a rule matches, you see how serious it is, the sentence that triggered it, and a line on why it is worth noting. The page adds those up into a single risk score out of 100.

You do not need an agent for any of this. Load a contract and the page already has something to show you.

## How WebMCP fits in

WebMCP is a browser feature that lets a page offer an AI agent a set of tools it can call. The agent does not have to read the screen or guess at the text. The page tells it what it can ask for, and the agent asks.

The page sets up its tools on `document.modelContext` as soon as it loads. A tool is a plain object. It has a name the agent uses, a description the agent reads to understand what the tool does, an `inputSchema` written in JSON Schema that lists the arguments, and an `execute` function that runs when the agent calls the tool and hands back a plain object.

```js
document.modelContext.registerTool({
  name: "analyze_contract",
  description: "Return the extracted terms, the risk flags, and a readability read for a contract.",
  inputSchema: {
    type: "object",
    properties: { which: { type: "string", enum: ["primary", "secondary"] } },
    required: ["which"],
  },
  execute: async ({ which }) => ({ ok: true, which, analysis: analyze(textOf(which)) }),
});
```

Once an agent is connected it can see every tool and call any of them. Ask it about the contract and it calls `analyze_contract`, gets back the structure the page already worked out, and explains it in normal language. The page does the reading and the flagging. What the agent adds is the judgment about what those flags mean for your situation.

Two things are worth knowing about the API. WebMCP used to live at `navigator.modelContext` and now lives at `document.modelContext`, with the old name kept as an alias, so the page uses whichever one the browser gives it. Registration wants one object that holds the name, the schema, and the execute function together. You will find older snippets that pass the name on its own, or that call the fields `parameters` and `handler`. Those register nothing on a current browser, so the code here uses the one-object form.

### The tools

| Tool | What it returns |
| --- | --- |
| `get_contract` | The full text of one contract |
| `set_contract` | Sets a contract's text on the page |
| `analyze_contract` | Terms, risk flags, and readability for one contract |
| `list_risk_flags` | Just the detected risky clauses, with severity |
| `extract_terms` | Parties, dates, amounts, term, notice, payment, governing law |
| `get_clause` | The passage covering a topic (termination, liability, ...) |
| `assess_risk` | A 0 to 100 risk score, a level, and severity counts |
| `compare_contracts` | A clause matrix and term comparison across both |
| `get_both_contracts` | Both texts at once |

## Running it

It is one static HTML file. Open it over HTTPS or on localhost, because the API only shows up in a secure context.

- ChatGPT's desktop app comes with an in-app browser that has WebMCP turned on.
- Chrome 149 and up works once you switch on `chrome://flags/#enable-webmcp-testing` and restart.
- Every other browser still runs the page. It loads a polyfill so the tools register, and it has a tool tester so you can call any tool by hand without an agent.

Click "Load sample" on the primary box, then run `analyze_contract` with `{ "which": "primary" }` from the tester or from your agent.

## Privacy

The contract stays in the browser. The page reads the file, runs the analysis, and builds any share link on your own machine, and none of it is sent anywhere. The one thing that ever sees the plain text is the agent you decide to connect.

## License

MIT. See [LICENSE](LICENSE).
