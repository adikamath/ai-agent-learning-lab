# Lesson 1: What Makes an AI Application an Agent?

For this lesson, I wanted to get a clearer picture of what actually makes an AI application an agent. The word *agent* is used for everything from a single model call to systems that choose tools, manage state, and decide what to do next. The easiest way for me to make that distinction concrete was to build three versions of the same task and compare where control lives in each one.

All three versions use the same question, fictional company dataset, and language model. The main thing that changes is how much of the execution path the model controls.

## Lesson objective

The goal is to understand how an application evolves from one-shot generation into a system that can make decisions and take actions through tools.

By the end of the lesson, I want to be able to explain:

- what separates an LLM call, a model-routed workflow, and a harness-based agent
- where control lives in each pattern
- how deterministic code and model judgment can work together
- why tool availability and tool descriptions affect agent behavior
- what these choices mean for predictability, flexibility, and product design

## The shared task

Each test case answers the same question:

> Using ARR per employee as the efficiency metric, which company generates recurring revenue most efficiently?

The source data lives in [`data/companies.json`](data/companies.json). It contains a small set of fictional companies with their ARR, employee count, funding, and a short description. ARR, or annual recurring revenue, is the revenue a company expects to repeat over a year from subscriptions or other recurring contracts.

Keeping the question and data constant lets me focus on the architecture. If the outputs differ, I can look at the execution path instead of wondering whether the systems solved different problems.

## The three test cases

### 1. LLM-only application

The first version sends the complete dataset and question to the model in one prompt. There are no tools and no execution loop. The model is responsible for interpreting the question, performing the calculations, ranking the companies, and presenting the answer in a single response.

This is the simplest version, but it also asks the model to do everything correctly at once.

```text
Question + dataset → LLM → Answer
```

### 2. LLM-routed workflow

The second version gives the model one bounded decision: interpret the question and select the appropriate analysis from a set of supported options. The application validates that selection, runs the corresponding calculation and ranking steps, and then asks the model to explain the verified results.

The model influences the route, but the application still controls the larger workflow. It cannot invent new steps, repeatedly call tools, or decide when execution should stop.

```text
Question → LLM selects route → Application runs fixed steps → LLM explains → Answer
```

### 3. Harness-based agent

The third version gives the model a goal and a small set of tools. A simple harness executes the model's requested tool calls, returns the results, and lets the model decide what to do next. The loop continues until the model produces a final answer.

This is where control shifts more clearly toward the model: it can choose the tool, supply arguments, inspect the result, and decide whether another action is needed.

```text
Goal + tools → LLM chooses action → Harness runs tool → Result returns to LLM → Repeat or answer
```

## How I’ll compare them

I’m keeping the evaluation deliberately lightweight. For each version, I’ll check:

- Did it use ARR per employee as the metric?
- Did it calculate the values correctly?
- Did it rank every company correctly?
- Did it identify the correct leader?
- What path did it take to produce the answer?

The last question matters most for this lesson. Two systems can return the same answer while relying on very different control and execution patterns.

## Further agent experiments

Once the harness-based agent works with its baseline toolset, I’ll change one part of the tool interface at a time:

1. **Remove a tool:** See whether the agent recognizes that a capability is missing, finds another route, or produces an unsupported answer.
2. **Weaken a tool description:** Observe how unclear instructions affect tool selection and arguments.
3. **Introduce overlapping tools:** Give the agent multiple plausible choices and see how consistently it selects the appropriate one.

Before each change, I’ll write down what I expect the agent to do. Then I’ll compare that prediction with the actual tool calls and final answer. The point is not to create an exhaustive evaluation suite; it is to build intuition for how tool design shapes model behavior.

## Running the lesson

The implementation lives in [`lesson-01-agent-basics.ipynb`](lesson-01-agent-basics.ipynb). The notebook builds each test case incrementally so the differences in control flow remain visible.

Install the notebook dependencies with:

```bash
./lesson-01-agent-basics/.venv/bin/python -m pip install -r lesson-01-agent-basics/requirements.txt
```

Create a local `.env` file from `.env.example`, replace the example value with your own OpenAI API key, and keep the real `.env` file out of source control.

## Troubleshooting

### VS Code cannot import `dotenv`

If the notebook reports `ModuleNotFoundError: No module named 'dotenv'`, the notebook may be using a different Python interpreter from the lesson's virtual environment.

First, install the dependencies using the virtual environment's Python:

```bash
./lesson-01-agent-basics/.venv/bin/python -m pip install -r lesson-01-agent-basics/requirements.txt ipykernel
```

Then, in VS Code:

1. Open the Command Palette.
2. Select **Python: Select Interpreter**.
3. Choose **Enter interpreter path...**
4. Select `lesson-01-agent-basics/.venv/bin/python`.
5. Return to the notebook and select the same environment as its kernel.

To confirm which Python environment the notebook is using, run:

```python
import sys

print(sys.executable)
```

The displayed path should end with:

```text
lesson-01-agent-basics/.venv/bin/python
```

The `.env` file and installed packages belong to separate parts of the setup: `.env` stores local configuration, while the selected Python environment determines which packages the notebook can import.
