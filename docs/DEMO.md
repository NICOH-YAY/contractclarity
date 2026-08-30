# Demo video script

Target 2:30, hard cap 3:00. Narration in your own voice or text-to-speech. Show the product working in the first 15 seconds.

## 0:00 to 0:15 — Hook

On screen: the page. Click "Load sample" on the primary contract. The on-page analysis fills in at once: key terms, a list of risk flags colored by severity, and a "Risk High" badge.

Narration: "This is ContractClarity. I dropped in a contract, and the page already pulled out the terms and flagged the risky clauses, before any AI is involved. Here is how it works with your agent."

## 0:15 to 0:45 — The idea

On screen: point at the risk flags and the extracted terms, then at the "WebMCP connected" badge.

Narration: "The page does the structured part. It parses the file in your browser, extracts the parties, dates, and payment terms, and runs a rulebook that flags things like auto-renewal, non-compete, and a missing liability cap, each with a severity and a score. Nothing leaves your machine. Then it exposes all of that to your agent through WebMCP."

## 0:45 to 1:45 — The agent reads it

On screen: open ChatGPT's desktop in-app browser on this page. Type: "Call analyze_contract for the primary contract and explain the top risks in plain English." The agent calls the tool and replies with a plain-English rundown that matches the flags on the page.

Narration: "I ask my own agent to read the contract. It calls the page's analyze_contract tool, gets the exact findings the page computed, and explains them for a non-lawyer. It is not guessing from raw text. It is reading structure the page owns."

## 1:45 to 2:20 — Compare two contracts

On screen: load a second contract into the secondary box. Type: "Call compare_contracts and tell me which agreement is more favorable to me, clause by clause." The agent returns a comparison.

Narration: "Load a second contract and the agent compares them. The page builds the clause matrix and the term comparison exactly, and the agent turns it into a recommendation. Comparing two agreements by hand is exactly the tedious, error-prone job worth handing off."

## 2:20 to 2:45 — Close

On screen: show the code briefly, `document.modelContext.registerTool`, then the Tool tester running a tool without an agent.

Narration: "Under the hood, the page registers nine tools on document.modelContext. The page structures the document, your agent reasons over it, and the contract stays in your browser. That is ContractClarity."

## Recording notes

Record early. The WebMCP browsers are new and can be flaky. If the live agent stumbles, the built-in Tool tester can stand in for any single tool call without breaking the story.
