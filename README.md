# ContractClarity

An agent-ready contract reader. Load a contract and the page pulls out the key terms and flags the risky clauses on its own. Your AI agent reaches those findings through WebMCP and explains, compares, and advises in plain English. Everything runs in your browser, so the contract never leaves your machine.

Built for the WebMCP Challenge. All work in this repository is new, created during the submission period.

## Why it fits WebMCP

The page owns real work. It parses PDF, DOCX, and TXT locally, extracts the parties, dates, amounts, term length, notice period, payment terms, and governing law, and runs a clause rulebook that flags auto-renewal, termination for convenience, indemnification, non-compete, arbitration, unilateral-change, IP-assignment, and missing liability caps, each with a severity and the matching snippet. That structured result is deterministic and the agent cannot produce it reliably from raw text on its own.

WebMCP then exposes those findings as tools. The agent calls `analyze_contract`, `list_risk_flags`, `extract_terms`, `get_clause`, and `compare_contracts`, reads the exact structured output, and turns it into advice. The split is the point: the page structures the document, the agent reasons over it. Remove WebMCP and the agent has no clean way in.

Privacy is the other reason this shape matters. Contracts are sensitive. The page keeps everything client-side, so the only thing that reads the text is the user's own agent, which they already trust and configure.

## The tools

All registered on `document.modelContext.registerTool` in the single-object form the spec uses.

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

```js
document.modelContext.registerTool({
  name: "analyze_contract",
  description: "Run the on-page analysis and return terms, risk flags, and readability.",
  inputSchema: {
    type: "object",
    properties: { which: { type: "string", enum: ["primary", "secondary"] } },
    required: ["which"],
  },
  execute: async ({ which }) => ({ ok: true, which, analysis: analyze(textOf(which)) }),
});
```

## Running it

It is a single static file. Open `index.html` over HTTPS or localhost (WebMCP needs a secure context).

- In ChatGPT's desktop in-app browser, WebMCP is on by default.
- In Chrome 149 or later, enable `chrome://flags/#enable-webmcp-testing` and restart.
- In any other browser the page still works. It loads a WebMCP polyfill for development and ships a built-in Tool tester so you can run every tool by hand without an agent.

To try it: click "Load sample" on the primary box, then use the Tool tester (or your agent) to run `analyze_contract` with `{ "which": "primary" }`.

## Deploying

Any static host works. Drop the folder on Cloudflare Pages, Netlify, Vercel, or GitHub Pages. There is no build step.

## What changed from the first draft

The original registered against a WebMCP API that does not match the spec (`registerResource`, and `registerTool(name, { parameters, handler })`), so it registered nothing in a real WebMCP browser. This version uses the correct `registerTool({ name, inputSchema, execute })` form, replaces the three-regex parser with a real term extractor and clause risk rulebook, fixes the share link that silently truncated and corrupted the contract, and adds a built-in tester so the tools are verifiable without an agent.

## License

MIT. See [LICENSE](LICENSE).
