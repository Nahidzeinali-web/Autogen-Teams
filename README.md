
# 📘 Tutorial: Using AutoGen Teams to Build Multi-Agent Workflows

This tutorial demonstrates how to use [AutoGen](https://github.com/microsoft/autogen) to create a simple multi-agent team that collaborates to solve a task using GPT-based agents.

---

## 🛠️ Prerequisites

- Python 3.10+
- `autogen-agentchat`
- `openai`
- `.env` file with your OpenAI API key

```bash
pip install autogen-agentchat openai python-dotenv
```

Create a `.env` file with:
```env
OPENAI_API_KEY=your_openai_api_key_here
```

---

## 📦 Step 1: Set Up Environment

```python
import os
from dotenv import load_dotenv
load_dotenv()

# Load your OpenAI API Key
api_key = os.getenv("OPENAI_API_KEY")
if not api_key:
    raise ValueError("Please set the OPENAI_API_KEY environment variable.")
```

---

## 🤖 Step 2: Create the Model Client

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="gpt-4o",
    api_key=api_key
)
```

---

## 🧠 Step 3: Define Agents

```python
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent

user_proxy = UserProxyAgent(
    name="User",
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    human_input_mode="NEVER",
    code_execution_config=False,
)

agent1 = AssistantAgent(
    name="Agent1",
    model_client=model_client,
    system_message="Add 1 to the number. First number is 0. Return only the result."
)

agent2 = AssistantAgent(
    name="Agent2",
    model_client=model_client,
    system_message="Add 1 to the number from the previous result. Return only the result."
)

agent3 = AssistantAgent(
    name="Agent3",
    model_client=model_client,
    system_message="Add 1 to the number from the previous result. Return only the result."
)
```

---

## 👥 Step 4: Form a Team

```python
from autogen_agentchat.teams import Team
from autogen_agentchat.coordinator import SequentialCoordinator

team = Team(
    name="AddOneTeam",
    agents=[user_proxy, agent1, agent2, agent3],
    coordinator=SequentialCoordinator(name="SequentialCoordinator")
)
```

---

## 🚀 Step 5: Run the Task

```python
import asyncio
from autogen_agentchat.ui import Console

await team.reset()

# Stream the result to the console
await Console(team.run_stream(task="Add 1 to the number starting from 0. Do it three times."))
```

---

## ✅ Final Output

Expected output:
```
Agent1: 1
Agent2: 2
Agent3: 3
```

---

## 📚 Additional Notes

- You can expand this setup to include tools, memory, or complex logic chains.
- To interact with real users or tools, set `human_input_mode="ALWAYS"` or use `FunctionTool`.
