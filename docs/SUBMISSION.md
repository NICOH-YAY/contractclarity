# ContractClarity — submission writeup

## What it is

ContractClarity is a contract reader that runs entirely in your browser. You load a contract as PDF, DOCX, or plain text, and the page extracts the key terms and flags the risky clauses on its own. Your AI agent then reaches those findings through WebMCP and explains them, compares two contracts, or drafts questions to ask before you sign. The contract text never leaves your machine.

## Why this use case is a strong fit for WebMCP

The work splits along a clean seam. The page does the structured, deterministic part: parse the file, pull out parties, dates, amounts, term length, notice period, payment terms, and governing law, and run a clause rulebook that flags auto-renewal, termination for convenience, indemnification, non-compete, arbitration, unilateral-change, IP-assignment, penalties, and a missing liability cap, each with a severity, a snippet, and an overall 0 to 100 risk score. The agent does the open-ended part: reading that structure and turning it into plain-English advice for a specific person.

WebMCP is what connects the two. The page registers `analyze_contract`, `list_risk_flags`, `extract_terms`, `get_clause`, `assess_risk`, and `compare_contracts` as tools. The agent calls them, reads the exact structured output, and reasons over it. Without WebMCP the agent has no clean way to reach the page's analysis, and the page is a document viewer with no voice. The two need each other.

Privacy is the second reason the shape fits. Contracts are sensitive. Because everything is client-side, the only thing that ever reads the text is the user's own agent, which they already trust and configure. There is no server to leak it and no API key to manage.

## How it creates a better experience

A person who wants a contract explained today either pays a lawyer or pastes it into a chatbot and hopes. Here the page shows its work first: the extracted terms, the flagged clauses colored by severity with the exact snippet, and a risk score, all before any agent is involved. Then the agent adds judgment on top of a structured base it can trust, rather than eyeballing raw text. The one-click prompts call the tools by name, so the handoff from page to agent is a single paste.

## What people and agents can do together that was hard before

Comparing two agreements clause by clause is tedious and error-prone by hand, and a model reading two long documents from scratch drops details. Here the person loads both, and the agent calls `compare_contracts` to get a clause matrix and a term comparison the page computed exactly, then explains which agreement is more favorable and why. A person can also share a contract by link, so a second person opens the same document and their own agent reads it, without re-uploading and without either contract touching a server.

## How WebMCP was implemented

Nine tools are registered on `document.modelContext.registerTool` when the page loads, each in the single-object form the spec uses: a `name`, a `description` written for the agent, a JSON Schema `inputSchema`, and an `execute` handler that returns a plain object. Read tools carry a `readOnlyHint`. The handlers call the page's analysis functions and return their structured results. A polyfill loads in non-WebMCP browsers so the page is testable everywhere, and a built-in Tool tester runs any tool by hand.

## Testing instructions for judges

No login. Open the URL in ChatGPT's in-app browser or Chrome 149+ with `chrome://flags/#enable-webmcp-testing` enabled. Click "Load sample" on the primary box, then ask your agent: "Call analyze_contract for the primary contract and explain the top risks in plain English." To exercise the tools without an agent, use the Tool tester in the lower right: pick a tool, adjust the prefilled arguments, and run it.
