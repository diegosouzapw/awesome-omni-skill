---
name: webshop-query-parser
description: This skill extracts key constraints and product requirements from a user's shopping query. It should be triggered when the agent receives a new shopping instruction, to identify product attributes (e.g., 'gluten free'), price limits (e.g., 'lower than 140.00 dollars'), and other specifications. The skill outputs a structured set of search criteria to guide subsequent product discovery.
---
# Instructions

## When to Use
Activate this skill immediately when a new shopping instruction is received from the user, before any search or click actions are performed.

## Core Task
Parse the user's natural language instruction to extract structured search criteria. Your goal is to identify:
1.  **Product Type/Name:** The primary item the user wants (e.g., "popcorn").
2.  **Key Attributes:** Descriptive features like "gluten free", "organic", "vegan", etc.
3.  **Price Constraints:** Any upper or lower price limits (e.g., "lower than 140.00 dollars").
4.  **Other Specifications:** Brand, size, quantity, or other qualifying terms.

## Procedure
1.  **Run the Parser:** Execute the bundled script `parse_query.py` on the user's instruction.
2.  **Review & Refine:** Examine the script's output. If the instruction is complex or ambiguous, use your judgment to refine the criteria. For example, ensure price limits are correctly interpreted as numeric ranges.
3.  **Formulate Search Strategy:** Use the extracted criteria to plan the initial web search. Combine the **Product Type** with the most critical **Key Attributes** to form effective initial search keywords.
    *   **Example:** For "i need gluten free popcorn, and price lower than 140.00 dollars", the script will output `{'product': 'popcorn', 'attributes': ['gluten free'], 'price_max': 140.0}`. Your initial search should be `search[gluten free popcorn]`.

## Output
After parsing, hold the structured criteria in memory. Use it to:
*   Guide the formulation of `search[keywords]` actions.
*   Evaluate product listings and details during `click` actions to check for constraint compliance (especially price).
*   Inform your reasoning in the `Thought:` part of your response.

## Notes
*   Keep the initial search query concise but precise. Prioritize must-have attributes from the user's instruction.
*   The price filter is often not available as a direct web action; you must manually check prices in the search results and product details.
*   If the initial search yields no results, consider broadening the search by removing less critical attributes one at a time, but always respect hard constraints like "gluten free" if specified.
