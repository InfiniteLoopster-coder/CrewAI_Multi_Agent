# CrewAI_Multi_Agent
Multi-Agent Content Generation with CrewAI
This repository contains a simple but complete example of a multi-agent workflow built using CrewAI
. The notebook shows how to coordinate several specialized AI agents — a Content Planner, a Writer, and an Editor — to produce a polished blog/article from just a single input: a topic.

The idea is to show you the pattern you can later swap with your own agents (researcher, code-writer, evaluator, RAG agent, etc.).

What this project does

Installs and imports CrewAI.

Defines 3 agents:

planner → plans the article

writer → writes the article using the plan

editor → proofreads / finalizes

Defines 3 tasks, each bound to one agent.

Creates a Crew with those agents and tasks.

Runs the whole workflow with crew.kickoff(...).

Displays the final content (Markdown) in the notebook.

All of this happens in:
/notebooks/Multi_Agent.ipynb (your uploaded notebook is named Multi_Agent.ipynb).

Suggested Repo Structure
.
├── notebooks/
│   └── Multi_Agent.ipynb
├── requirements.txt
└── README.md   ← (this file)


requirements.txt (example)

crewai
ipykernel
python-dotenv


(If you use OpenAI models, you’ll also need openai or the provider lib you’re calling through CrewAI.)

Quickstart
1. Clone
git clone [https://github.com/InfiniteLoopster-coder/CrewAI_Multi_Agent.git]
cd CrewAI_Multi_Agent

2. Create & activate venv (optional but recommended)
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

3. Install deps

Inside the notebook you had:

pip install crewai


For the repo do:

pip install -r requirements.txt

4. Set API key

In your notebook you did:

import os
os.environ["OPENAI_API_KEY"] = ""


In a real project, don’t hardcode it. Use .env:

echo "OPENAI_API_KEY=sk-..." > .env


Then in Python:

from dotenv import load_dotenv
load_dotenv()

5. Run the notebook

Open Jupyter / VS Code and run notebooks/Multi_Agent.ipynb.
The last cell does something like:

topic = "Artificial Intelligence"
result = crew.kickoff(inputs={"topic": topic})


and then displays the markdown.

How the Code is Organized (explained)

Your notebook basically does this:

Import core classes

from crewai import Agent, Task, Crew


Create Agents

planner = Agent(
    role="Content Planner",
    goal="Plan engaging and structured content for a given topic",
    allow_delegation=False,
    ...
)


role → who the agent is

goal → what it must achieve

allow_delegation → can it hand work to others?

Similarly for writer and editor.

Create Tasks

plan = Task(
    description="Create an outline for the article on {topic} ...",
    agent=planner
)


Every task is bound to one agent

Task text gives instructions and output expectations

Create the Crew (the orchestrator)

crew = Crew(
    tasks=[plan, write, edit],
    agents=[planner, writer, editor],
    verbose=False
)


This is the “director” that knows:

which agents exist

which tasks to run

in what order

Run the workflow

result = crew.kickoff(inputs={"topic": topic})


kickoff(...) = run the whole pipeline once.

What is CrewAI (explained for you)

CrewAI is a lightweight, Pythonic framework to build multi-agent AI workflows. Think of it as:

“I have several LLM-powered workers (agents). Each worker knows its job (task). I want someone to tell them in what order to work and to pass results between them (the crew).”

Core concepts

Agent

An LLM with a persona (role), goal, and sometimes tools.

Example: “Senior Tech Writer”, “Market Researcher”, “Python Refactorer”.

In your code: planner, writer, editor.

Task

A unit of work assigned to exactly one agent.

Has description, maybe expected_output, sometimes context.

In your code: plan, write, edit.

Crew

The orchestrator that runs tasks in sequence (or later, in parallel).

Knows the agents and tasks and how to pass outputs.

In your code: crew = Crew(...) and crew.kickoff(...).

Run / Kickoff

Executes the workflow once, with given inputs.

Returns the final output (in your case, the final blog post / markdown).

Why CrewAI is useful

You don’t write complex LangGraph graphs just to chain 3 LLM calls.

You make roles explicit → easier to maintain.

You can swap models or add tools to specific agents.

You can extend later (research agent → writer → SEO agent → publisher).

CrewAI vs LangChain vs LangGraph (quick view)

LangChain → big toolbox for LLM apps (prompts, chains, tools, RAG, memory).

LangGraph → graph-based orchestrator (nodes = steps, edges = transitions, state = shared memory).

CrewAI → “I have several people (agents), give each one a task, and run them in order.”

For small, human-like team workflows, CrewAI is super fast to get started.
For complex agentic systems with branching, retries, tool routing, long-lived state, LangGraph is the better fit.

How to Add Web Search / Papers / News (to match your earlier question)

Right now your agents mostly rely on the base LLM. To make them smarter:

Add a tool to the agent (CrewAI supports tools)

Tool can be:

a web search function (SerpAPI, Tavily, Brave, etc.)

an arXiv fetcher

a local RAG retriever

Then in the agent definition:

from crewai import Agent
from my_tools import web_search_tool

researcher = Agent(
    role="Researcher",
    goal="Find latest academic and news sources on the topic",
    tools=[web_search_tool],
    ...
)


And create a research task that runs before plan:

research = Task(
    description="Search the web, news, and academic sources for {topic} and return a structured research brief.",
    agent=researcher
)


Then update the crew:

crew = Crew(
    tasks=[research, plan, write, edit],
    agents=[researcher, planner, writer, editor],
)


Now your writer can use the output of research as context.

Observability / Validation (Langfuse note)

You earlier asked “can you also use Langfuse for validation of prompts?”
Yes — you can wrap each task call or the entire crew.kickoff(...) with Langfuse logging. Typical pattern:

from langfuse import Langfuse

lf = Langfuse()

with lf.trace(name="crewai-run", input={"topic": topic}) as trace:
    result = crew.kickoff(inputs={"topic": topic})
    trace.update(output={"result": str(result)})


This gives you:

traces per run

prompt & output logging

quality / score fields (you can attach RAG / eval scores)

Example Usage
topic = "Use of Multi-Agent Systems in Enterprise RAG"
result = crew.kickoff(inputs={"topic": topic})
print(result)


You’ll get a long, structured article that went through plan → write → edit.

Add to GitHub

Create repo

Add notebooks/ and requirements.txt

Put this README.md

Commit & push

git add .
git commit -m "Add crewai multi-agent demo"
git push origin main
