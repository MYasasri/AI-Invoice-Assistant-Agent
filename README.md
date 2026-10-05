# AI Invoice Assistant

An agentic AI-based invoice assistant that uses a Large Language Model (LLM) to understand natural-language invoice requests, maintain invoice memory, decide the next action, and execute invoice operations through Python tools.

The project demonstrates a **Plan → Act → Observe** agent loop using the Groq API and `openai/gpt-oss-20b`.

## Overview

Traditional invoice systems require users to manually enter structured information such as item name, quantity, price, tax, and discounts.

This project provides a conversational approach where users can describe an invoice using natural language.

For example:

```text
Create an invoice with:
2 notebooks at ₹120 each
3 pens at ₹50 each
1 school bag at ₹900

Apply 18% GST.
```

The AI agent interprets the request, determines the required actions, calls the appropriate tools, maintains the invoice state, and produces the final invoice.

## Key Features

- Natural-language invoice creation
- LLM-based agent decision making
- Plan → Act → Observe agent loop
- Invoice memory and conversation memory
- Tool-based invoice operations
- Automatic subtotal calculation
- Rule-based discount calculation
- GST calculation
- Grand total calculation
- Missing-information detection
- Multi-step interaction
- JSON-based structured agent decisions
- Interactive command-line interface
- Maximum-step limit to prevent infinite agent loops

## System Architecture

```text
                    User
                      |
                      v
          Natural Language Request
                      |
                      v
              +---------------+
              |   AI Agent    |
              |     LLM       |
              +-------+-------+
                      |
              Decide Next Action
                      |
          +-----------+-----------+
          |                       |
          v                       v
     add_item()             compute_total()
          |                       |
          +-----------+-----------+
                      |
                      v
              Invoice Memory
                      |
                      v
                 Observation
                      |
                      v
                 AI Agent
                      |
              +-------+-------+
              |               |
           Continue          Final
              |               |
              v               v
        Next Action      Final Invoice
```

## Agent Workflow

The agent follows a multi-step decision loop:

```text
User Request
     ↓
Read Current Memory
     ↓
LLM Decides Next Action
     ↓
Execute Python Tool
     ↓
Observe Tool Result
     ↓
Update Invoice State
     ↓
LLM Decides Again
     ↓
Complete / Ask User / Continue
```

The agent can perform four types of actions:

```text
add_item
compute_total
ask
final
```

### Example

The user provides:

```text
Create an invoice with:
2 notebooks at ₹120 each
3 pens at ₹50 each
1 school bag at ₹900
Apply 18% GST.
```

The agent can execute:

```text
Step 1 → add Notebook
Step 2 → add Pen
Step 3 → add School Bag
Step 4 → compute Total
```

## Tools

### 1. `add_item()`

Adds an item to the current invoice.

Parameters:

- `name` — Item name
- `price` — Price per unit
- `qty` — Quantity

Example:

```python
add_item("Notebook", 120, 2)
```

The item is stored in invoice memory.

### 2. `compute_total()`

Calculates the complete invoice.

The function calculates:

```text
Subtotal
   ↓
Discount
   ↓
Taxable Amount
   ↓
GST
   ↓
Grand Total
```

The GST percentage is supplied by the user.

## Discount Rules

The project uses the following predefined discount rules:

| Subtotal | Discount |
|---|---:|
| Less than ₹500 | 0% |
| ₹500 or more | 5% |
| ₹1000 or more | 10% |

The discount is applied before GST calculation.

## Memory

The project contains an `InvoiceMemory` class that maintains:

```python
self.items = []
self.conversation = []
```

### Invoice Items

Stores information such as:

```text
Item Name
Price
Quantity
```

### Conversation Memory

Stores previous user messages so that the agent can use information from previous interactions.

The memory class provides methods for:

```text
add_item()
get_items()
remember()
get_memory()
clear()
```

The `clear()` method resets the current invoice and conversation memory.

## Handling Missing Information

The agent is instructed not to invent missing invoice information.

For example, if the user says:

```text
Create an invoice for 5 laptops.
Apply 18% GST.
```

but does not provide the laptop price, the agent asks:

```text
What is the price of each laptop?
```

This prevents the system from arbitrarily generating a price.

## Structured AI Decisions

The LLM is instructed to return JSON instead of unrestricted text.

For example, when an item needs to be added:

```json
{
  "action": "add_item",
  "name": "Notebook",
  "price": 120,
  "qty": 2
}
```

For calculating the invoice:

```json
{
  "action": "compute_total",
  "tax_percent": 18
}
```

When information is missing:

```json
{
  "action": "ask",
  "message": "What is the price of the notebook?"
}
```

The Python program parses the JSON and executes the corresponding operation.

## Technology Stack

### Programming Language

- Python

### AI / LLM

- Groq API
- `openai/gpt-oss-20b`

### Libraries

- `groq`
- `json`
- `os`

### Development Environment

- Google Colab

## Project Structure

The project is implemented in a Google Colab notebook.

The main components are:

```text
AI Invoice Assistant
│
├── Groq API Configuration
│
├── InvoiceMemory
│   ├── add_item()
│   ├── get_items()
│   ├── remember()
│   ├── get_memory()
│   └── clear()
│
├── Invoice Tools
│   ├── add_item()
│   └── compute_total()
│
├── Invoice Display
│   └── display_invoice()
│
├── LLM Interface
│   └── ask_groq()
│
├── AI Agent
│   └── invoice_agent()
│
└── Interactive Interface
    ├── clear
    └── exit
```

## Example Output

For the following request:

```text
2 notebooks at ₹120 each
3 pens at ₹50 each
1 school bag at ₹900
18% GST
```

The system calculates:

```text
Notebook:       2 × ₹120 = ₹240
Pen:            3 × ₹50  = ₹150
School Bag:     1 × ₹900 = ₹900

Subtotal:             ₹1290.00
Discount:             10%
Discount Amount:       ₹129.00
Taxable Amount:       ₹1161.00
GST:                   18%
GST Amount:            ₹208.98

Grand Total:          ₹1369.98
```

## Interactive Example

The assistant can also handle incomplete requests.

### User

```text
pencil - 200
```

### Agent

```text
What is the quantity of pencils?
```

### User

```text
300
```

The agent uses the previous context and adds:

```text
300 × pencil @ ₹200
```

This demonstrates the use of conversation and invoice memory.

## Why Use an AI Agent?

A conventional invoice calculator requires structured inputs.

This project allows users to provide instructions in natural language.

The LLM handles:

- Understanding the user's request
- Identifying required information
- Deciding which tool to use
- Handling missing information
- Determining the next action

The actual financial calculations are handled by deterministic Python functions rather than relying on the LLM to perform arithmetic.

This separation improves reliability:

```text
LLM
→ Reasoning and decision making

Python
→ Deterministic calculations and state management
```

## Error Handling

The project includes handling for invalid LLM responses.

The agent expects JSON output and attempts to parse it using:

```python
json.loads()
```

If invalid JSON is returned, the program reports an error instead of directly executing an unknown action.

The agent also has a maximum number of steps:

```python
max_steps = 10
```

This prevents the agent loop from continuing indefinitely.

## Limitations

This is a prototype implementation and currently has several limitations:

- Invoice data is stored only in memory.
- Data is lost when the Python session ends.
- No database is currently integrated.
- No authentication or user management.
- No graphical web interface.
- No PDF invoice generation.
- Limited input validation.
- The LLM's action selection may occasionally require additional validation.

## Future Enhancements

Possible improvements include:

- MySQL/PostgreSQL database integration
- Persistent invoice storage
- Invoice IDs and timestamps
- Customer management
- PDF invoice generation
- Web interface using Flask/FastAPI
- Frontend using React
- Authentication and authorization
- Stronger tool-argument validation
- Invoice history and search
- Multiple tax configurations
- Email invoice delivery
- Cloud deployment
- Structured function/tool calling instead of relying only on JSON text output

## Security Considerations

The Groq API key should never be hard-coded into the source code or committed to GitHub.

In Google Colab, the project uses Colab user data to retrieve the API key.

For deployment, environment variables or a secure secrets manager should be used.

Example:

```python
os.environ["GROQ_API_KEY"]
```

Do not commit your actual API key to a public repository.

## How to Run

### 1. Open the Notebook

Open the project notebook in Google Colab.

### 2. Install the Groq Package

```python
!pip -q install groq
```

### 3. Configure the API Key

Store your Groq API key securely in Google Colab's user data/secrets.

The notebook retrieves it using:

```python
userdata.get("Groq_API")
```

### 4. Run the Notebook

Execute the cells in order.

### 5. Start the Assistant

The interactive interface supports:

```text
clear
exit
```

You can then enter natural-language invoice requests.

## Example Requests

```text
Create an invoice with 2 notebooks at ₹120 each and apply 18% GST.
```

```text
Add 5 pens at ₹20 each.
```

```text
Create an invoice for 3 laptops at ₹50000 each with 18% GST.
```

```text
clear
```

## Project Objective

The primary objective of this project is to demonstrate how **Generative AI, LLMs, memory, tools, and agentic workflows** can be combined to automate a practical business task.

Rather than using an LLM only for text generation, the project demonstrates an agent that can:

```text
Understand → Decide → Act → Observe → Continue
```

## Conclusion

The AI Invoice Assistant demonstrates an agentic approach to invoice generation.

The LLM provides natural-language understanding and decision-making, while Python tools handle deterministic invoice operations such as item management, discount calculation, GST calculation, and final invoice generation.

The project serves as a prototype for integrating AI agents into business automation workflows where an AI system can interpret user requests and interact with deterministic software tools.
