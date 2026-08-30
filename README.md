# ContractClarity

ContractClarity is a contract reader that runs in your browser. You load a contract as a PDF, a Word file, or plain text, and the page pulls out the important terms and marks the clauses worth a second look. An AI agent can then read those findings through WebMCP and talk you through them. The text stays on your machine the whole time.

## What the page does on its own

Open a file and the page does the mechanical reading for you. It parses the document locally, then extracts the parties, the effective date, the term length, the notice period, the payment terms, the dollar amounts, and the governing law. On top of that it runs a small rulebook that looks for clauses people tend to skim past: auto-renewal, termination for convenience, indemnification, a non-compete, an arbitration or jury waiver, penalties and late fees, one-sided change rights, IP assignment, and a liability section with no cap. Each hit comes with a severity, the sentence it matched, and a short note on why it matters. The page rolls the hits into one risk score from 0 to 100 so you get a read at a glance.

None of that needs an agent. The page is useful the moment a contract is loaded.

## How the WebMCP part works

WebMCP is a browser API that lets a page hand structured tools to an AI agent. Instead of the agent scraping the screen or guessing at the raw text, the page tells it plainly what it can ask for, and the agent calls those tools directly.

ContractClarity registers its tools on `document.modelContext` when the page loads. Each tool is a plain object with four parts. A `name` the agent refers to. A `description` written for the agent to read. An `inputSchema` that spells out the arguments as JSON Schema. And an `execute` function that runs when the agent calls the tool and returns a plain object.

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

When an agent is connected it sees the whole set and can call any of them. Ask it to explain the contract and it calls `analyze_contract`, reads the exact structure the page built, and puts it in plain words. The page did the parsing and the flagging, the agent does the explaining, and each side handles the part it is good at.

A couple of details about the API itself. WebMCP moved its entry point from `navigator.modelContext` to `document.modelContext`, and the old name still works as an alias, so the page reads whichever one the browser exposes. Registration takes a single object with `name`, `inputSchema`, and `execute`. Some older examples pass the name as a separate first argument, or use `parameters` and `handler`. Those do not match the current API and register nothing, so the code here uses the single-object form.

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

It is a single static file. Open `index.html` over HTTPS or on localhost, since the API only exists in a secure context.

- ChatGPT's desktop app has an in-app browser with WebMCP on by default.
- Chrome 149 and later works after you turn on `chrome://flags/#enable-webmcp-testing` and restart.
- Any other browser still runs the page. It loads a small polyfill so the tools register, and it ships a tool tester so you can call each tool by hand without an agent.

To try it, click "Load sample" on the primary box, then run `analyze_contract` with `{ "which": "primary" }` from the tool tester or your agent.

## Privacy

The contract never leaves the browser. Parsing, analysis, and the optional share link all happen on your machine. The only thing that reads the plain text is the agent you chose to connect.

## License

MIT. See [LICENSE](LICENSE).
