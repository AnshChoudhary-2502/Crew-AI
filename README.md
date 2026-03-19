# CrewAI Documentation

## The Philosophy of CrewAI
CrewAI is built on one core idea: complex problems are solved better by a team of specialists than by one generalist.

Think about how humans work. When a company wants to launch a product, they don't hire one person to do everything — they have a product manager, a designer, a developer, and a marketer. Each person is focused, skilled, and responsible for their piece. CrewAI replicates this same structure for AI.

### The Three Pillars

1.  **Role-based thinking** — every agent has a defined identity. It knows who it is, what it's good at, and what its goal is. This focus makes it perform much better than asking a generic AI to "do everything."

2.  **Collaborative autonomy** — agents are not micromanaged. You define the task, and they figure out how to accomplish it, which tools to use, and how to communicate results to the next agent. They are autonomous workers, not scripted bots.

3.  **Composability** — you build a crew the way you'd build a LEGO set. Snap together agents, tasks, and tools in different combinations for different workflows.

Now let's go deep into every component.

---

## 1. Agents — The Workers
An Agent is the fundamental unit of CrewAI. It is an AI persona with a specific identity. Here's what makes up an agent:

*   **Role** — This is the agent's job title. It shapes how the LLM thinks and responds. "Senior Financial Analyst" will behave differently from "Creative Copywriter" even if they use the same underlying model. The role is essentially a persona.
*   **Goal** — This is what the agent is trying to achieve. It's a clear, specific objective like "Find the most recent and credible sources on climate change." The agent always keeps this goal in mind while working.
*   **Backstory** — Think of this as the agent's résumé. It provides context: "You are a veteran data scientist with 15 years of experience in machine learning." This shapes the style and depth of the agent's responses.
*   **LLM (Brain)** — Every agent is powered by a large language model (like Claude, GPT-4, or Gemini). This is the actual intelligence behind the agent. The role/goal/backstory are just instructions given to this model.
*   **Tools** — These are the agent's abilities. Without tools, an agent can only "think." With tools, it can browse the web, read files, call APIs, write code, query databases, etc.
*   **Memory** — Agents can optionally remember what they've done in previous steps, so they don't repeat themselves or lose context.

---

## 2. Tasks — The Work
A Task is a specific, defined piece of work. It is not vague like "do research" — it's precise: "Search the internet for the top 10 AI breakthroughs of 2024 and write a bullet-point summary."

Every task has these parts:

*   **Description** — The actual instruction. The more detailed and clear this is, the better the agent performs. Think of it like writing a good prompt.
*   **Expected output** — You tell the task what the result should look like: "A numbered list of findings," "A 500-word essay," "A JSON object with keys: name, date, source." This guides the agent toward the right format.
*   **Assigned agent** — Every task is given to a specific agent. The research task goes to the Researcher agent. The writing task goes to the Writer agent.
*   **Context** — Tasks can receive the output of previous tasks as input. So the writer gets the researcher's findings automatically — no manual copying required.
*   **Output file** — Optionally, you can tell a task to save its output to a file (like report.txt or data.json).

---

## 3. The Crew — The Team Container
The Crew is the top-level object that ties everything together. It's like the company itself — it holds all the agents, all the tasks, and decides how they work together.

When you call `crew.kickoff()`, the Crew springs into action and starts executing tasks according to the chosen process.

### The Two Processes
*   **Sequential Process** — Tasks run one after another, like an assembly line. Task 1 finishes -> its output goes to Task 2 -> Task 2 finishes -> output goes to Task 3, and so on. This is the default, simplest mode. Perfect for things like: research -> write -> review.
*   **Hierarchical Process** — A manager agent is created automatically (or you can define one). This manager reads the overall goal, then decides which agent to assign each task to, in what order. It's more flexible and powerful, but uses more LLM calls since the manager is always "thinking." Useful for complex, open-ended problems where you don't know the exact steps in advance.

---

## 4. Tools — The Abilities
Tools are what give agents real-world capabilities. Without tools, an agent is just a very good text generator. With tools, it can interact with the world.

CrewAI comes with many built-in tools, and you can build custom ones:
*   **Built-in tools** include web search (SerperDevTool), website scraping, reading PDF files, reading CSV files, directory searching, GitHub browsing, YouTube channel search, and more.
*   **LangChain tools** — CrewAI integrates with LangChain's huge ecosystem of tools, so anything LangChain supports (hundreds of integrations) can be used in CrewAI.
*   **Custom tools** — You can write your own tool in Python. For example, a tool that queries your internal company database, or one that sends a Slack message, or one that checks live stock prices. Any Python function can become a tool.

The way tools work: when an agent is working on a task and realizes it needs external data, it automatically calls the right tool, gets the result, and incorporates it into its thinking. This is called the ReAct loop (Reason -> Act -> Observe -> Repeat).

---

## 5. Memory — Making Agents Smarter Over Time
Memory is what prevents agents from being goldfish. Without memory, every task starts fresh and the agent has no idea what it did before. CrewAI provides three kinds:

*   **Short-term memory** lives only within the current run. It's basically the conversation context — what the agent has done, seen, and said during this task execution. Once the run ends, it's gone.
*   **Long-term memory** persists across multiple runs. It's stored in a database (SQLite by default). So if you run your crew on Monday and Tuesday, the Tuesday run can remember what happened on Monday. This is powerful for things like ongoing research or iterative document improvement.
*   **Entity memory** tracks specific things the agent encounters — people, organizations, topics, places. So if an agent reads 20 articles and five of them mention "OpenAI," it builds up a profile of OpenAI across those articles rather than treating each mention in isolation.

---

## 6. The ReAct Loop — How an Agent Thinks
When an agent starts working on a task, it doesn't just generate an answer immediately. It goes through a loop called ReAct (Reason + Act):

1.  **Reason**: It first reasons — "What do I need to answer this? Do I need to search the web? Do I need to read a file?"
2.  **Act**: Then it acts — calls the appropriate tool.
3.  **Observe**: Then it observes — reads what the tool returned.
4.  **Repeat/Decide**: Then it decides: is this enough, or do I need to do more? If more is needed, it loops again. When satisfied, it writes the final answer.

This is what makes agents feel "intelligent" — they don't just respond, they think through what they need and go get it.

---

## 7. Putting It All Together — A Real Example
Here's how you'd build an AI news report crew in code (simplified):

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool

# 1. Tools
search_tool = SerperDevTool()

# 2. Agents
researcher = Agent(
    role="AI News Researcher",
    goal="Find the top 5 AI developments this week",
    backstory="You are an expert technology journalist with 10 years of experience.",
    tools=[search_tool]
)

analyst = Agent(
    role="Trend Analyst",
    goal="Identify key patterns and implications in the news",
    backstory="You are a strategic analyst who spots trends before others do."
)

writer = Agent(
    role="Content Writer",
    goal="Write a clear, engaging news report",
    backstory="You write for a general audience. You make complex topics simple."
)

# 3. Tasks
research_task = Task(
    description="Search for the most important AI news from the past 7 days",
    expected_output="A list of 10 news items with source, date, and summary",
    agent=researcher
)

analysis_task = Task(
    description="Analyze the research and identify the 5 most significant trends",
    expected_output="5 trend summaries, each 2-3 sentences",
    agent=analyst,
    context=[research_task]  # gets researcher's output
)

writing_task = Task(
    description="Write an 800-word article covering the 5 trends",
    expected_output="A complete article with intro, body, and conclusion",
    agent=writer,
    context=[analysis_task],
    output_file="ai_news_report.txt"
)

# 4. Crew
crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[research_task, analysis_task, writing_task],
    process=Process.sequential
)

# 5. Run it
result = crew.kickoff()
```
That's it. With about 50 lines of code, you have a fully automated multi-agent pipeline.

---

## Key Characteristics of CrewAI
*   **Autonomy** — You define the goal, not the steps. Agents figure out the steps themselves using the ReAct loop.
*   **Role specialization** — Each agent is laser-focused on its domain. This specialization produces better results than a single general-purpose prompt.
*   **Composability** — Mix and match agents, tools, and tasks. Reuse an agent across multiple crews or projects.
*   **LLM agnostic** — Works with Claude (Anthropic), GPT-4 (OpenAI), Gemini (Google), Llama (local), and more. You pick the brain.
*   **Human-in-the-loop** — You can configure any agent to pause and ask a human for input before proceeding. Useful for high-stakes decisions.
*   **Verbose mode** — You can turn on detailed logging to see exactly what each agent is thinking, what tools it called, and what it got back. Great for debugging.
*   **Guardrails** — You can set max iterations per task, max requests per minute, and error handling, so agents don't spiral into infinite loops or blow your API budget.

---

## Where CrewAI is Used in the Real World
CrewAI is commonly used for:
*   Automated research and report generation
*   Customer support ticket triage and response drafting
*   Code review and bug-finding pipelines
*   Competitive intelligence gathering
*   Content marketing pipelines (research -> write -> SEO -> publish)
*   Data analysis and interpretation workflows
*   Any task that benefits from a structured handoff between specialized roles.

In short: if you can describe a job that a team of humans would do together — with each person playing a specific role — CrewAI can automate that workflow.