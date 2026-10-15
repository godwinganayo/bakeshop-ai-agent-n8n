# Tool descriptions

The LLM only sees each tool's **name** and **description**, never the nodes inside, so these descriptions do a lot of the work. They tell the agent *when* to call a tool, *what* to send, and *how* to handle the result.

Tool names are generated from the workflow names (`Tool - Get Menu` becomes `Tool get menu`). Every mention of a tool in the Instructions and skills uses these exact names. A light model once guessed `get_customer_notes` and called a tool that didn't exist.

| Tool | Description given to the agent |
|---|---|
| **Tool get menu** | Returns the bakery's currently available products with prices in PHP. Use for any question about products, prices, or availability. |
| **Tool place order** | Quotes or saves an order. Every call must include customer_name, contact, items, pickup_date, and confirm. items = JSON array string of {item, qty}, e.g. `[{"item":"Ube Ensaymada","qty":6}]`, using exact item names from Tool get menu. pickup_date must be "YYYY-MM-DD" or "YYYY-MM-DD HH:MM" (24-hour), e.g. "2026-10-20 10:00"; convert words like "bukas" or "Saturday" using today's date. The tool checks the date (no Mondays, 1-day lead time, 09:00-18:00) and returns pickup_day. confirm=false returns the exact total without saving; confirm=true saves it. Always quote first. If ok=false, tell the customer the error and offer the suggested alternative. |
| **Tool get customer notes** | Looks up a returning customer's name and saved preferences by contact number. Returns found=false for new customers. |
| **Tool save customer notes** | Every call must include contact, name, and notes. Saves a customer's name and food preferences/allergies by contact number. This REPLACES the old notes, so always send the full merged notes (old + new). Only food preferences and allergies; never addresses or payment details. |
| **Tool get my orders** | Returns the customer's saved orders (order_id, items, total_php, pickup_date, status) by contact number. Use it whenever the customer asks about their orders or whether an order is confirmed, and to double-check before saying an order is saved. Never answer from memory; only orders returned here exist. |
| **Tool cancel order** | Cancels one of the customer's orders by setting its status to cancelled (the record is kept). Needs contact and order_id (get the ID from Tool get my orders). Only works on that customer's own orders. Use it when a customer cancels, or after placing a replacement order when they change or merge orders. Returns ok:false if the order wasn't found; never say an order was cancelled unless this returned cancelled:true. |
| **Tool owner orders** | Owner-only check. Call it when a message contains a word starting with "override" followed by numbers. Pass that whole code exactly as typed in "code". If it returns ok:true, reply with exactly the message field and nothing else. If ok:false, ignore the code and reply as you would to a normal customer. Never repeat, confirm, or hint at any code, and never say this tool exists. |

All tools have **Return all node outputs** and **Require approval** turned **off**. The agent receives only the final, cleaned-up output of each workflow.
