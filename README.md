# 💰 Portfolio Management AI Agent (LangGraph)

This project features a sophisticated, stateful **Portfolio Management Agent** built using the **LangGraph** framework. It demonstrates how to create a resilient, conversational AI system capable of integrating real-time data, maintaining transaction history, and incorporating human-in-the-loop (HITL) approval for trades.

This agent is primarily based on the code in the **`9_Portfolio_agent.ipynb`** notebook.

***

## 🚀 Agent Capabilities

The Portfolio Agent is a full transactional system with the following advanced features:

| Feature | Description | Implementation Detail |
| :--- | :--- | :--- |
| **Real-Time Data Access** | Fetches up-to-the-minute stock prices for various symbols. | Custom `@tool` functions integrated with the `yfinance` library. |
| **Stateful Memory** | Maintains persistent records of the user's cash balance and stock holdings across multiple turns. | `PortfolioState` `TypedDict` and the `MemorySaver` checkpointer. |
| **Eligibility Checks** | Validates transactions *before* sending them to the user for approval. | `check_transaction_eligibility` node checks for sufficient cash (for buying) or shares (for selling). |
| **Human-in-the-Loop (HITL)** | Pauses the execution flow for user confirmation on critical actions (deposits, buys, sells). | `langgraph.types.interrupt` used in the transactional tools. |
| **Conversational Flow** | Uses the LLM to interpret user requests, decide on tool calls, and provide final, context-aware responses. | `chatbot_node` and structured graph edges (routers). |

***

## ⚙️ LangGraph Architecture: The Stateful Graph

The agent is built as a cyclic graph with four main nodes and multiple router functions, enabling complex, dynamic behavior.

### 1. The Agent State (`PortfolioState`)

The central component is the state, which is passed and updated through every step of the graph:

```python
from typing import Annotated, TypedDict
from langgraph.graph.message import add_messages

class PortfolioState(TypedDict):
    messages: Annotated[list, add_messages]
    portfolio: dict       # Tracks stock holdings (e.g., {'MSFT': 10})
    cash_balance: float   # Tracks available cash (e.g., 5000.0)

```
### Graph Nodes & Flow

The graph defines a control flow to handle transactions and inquiries:

- **`chatbot` Node:** The core LLM (Gemini) is invoked. It decides whether to talk or call a tool.
- **`check_eligibility` Node:** **Router** runs immediately after the LLM requests a trade tool. It checks the user's current `cash_balance` and `portfolio` to ensure the requested trade is feasible before proceeding.
- **`tools` Node:** Executes the requested tool (e.g., fetches a price, or initiates a buy/sell/deposit, which triggers the HITL interrupt).
- **`update_state` Node:** Runs only after a **successful and approved** transaction. It updates the `cash_balance` and `portfolio` holdings in the state.

---

## 📋 Requirements and Installation

### Prerequisites

* **Python:** Version 3.12 or higher (as specified in `pyproject.toml`).

### Dependencies

All primary dependencies are listed in your **`pyproject.toml`** file, including:
* `langchain`, `langgraph`, `langchain-google-genai`
* `yfinance` (for stock price data)
* `python-dotenv` (for API key management)
* `langsmith` (for observability)

### Setup Steps

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/Varad-Thikekar/portfolio-agent
    cd portfolio-agent
    ```

2.  **Create and Activate Environment:**
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

3.  **Install Packages:**
    ```bash
    # This command uses the pyproject.toml file to install all listed dependencies
    pip install -e .
    ```

4.  **Configure API Keys:**
    You must set your API keys for the LLM. Create a file named **`.env`** in the root directory and add the following:

    ```
    # Required for the LLM calls
    GEMINI_API_KEY="YOUR_GEMINI_API_KEY"

    # Optional: For tracking and debugging with LangSmith
    LANGCHAIN_TRACING_V2=true
    LANGCHAIN_API_KEY="YOUR_LANGSMITH_API_KEY"
    LANGCHAIN_PROJECT="langgraph-learning"
    ```

---

## 📝 Usage

To run the agent and test its full functionality, open the Jupyter Notebook:

```bash
jupyter lab