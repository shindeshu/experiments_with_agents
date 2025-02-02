https://huggingface.co/docs/smolagents/en/conceptual_guides/intro_agents

## Agentic Workflows

AI Agents are programs where LLM outputs control the workflow.

Types of agentic workflows

| Agency Level | Description                                      | How that’s called  | Example Pattern                                     |
|-------------|--------------------------------------------------|--------------------|-----------------------------------------------------|
| ☆☆☆        | LLM output has no impact on program flow        | Simple Processor  | `process_llm_output(llm_response)`                 |
| ★☆☆        | LLM output determines an if/else switch         | Router            | `if llm_decision(): path_a() else: path_b()`      |
| ★★☆        | LLM output determines function execution       | Tool Caller       | `run_function(llm_chosen_tool, llm_chosen_args)`   |
| ★★★        | LLM output controls iteration and program continuation | Multi-step Agent | `while llm_should_continue(): execute_next_step()` |
| ★★★        | One agentic workflow can start another agentic workflow | Multi-Agent     | `if llm_trigger(): execute_agent()`               |
The multi-step agent has this code structure:

```
memory = [user_defined_task]
while llm_should_continue(memory): # this loop is the multi-step part
    action = llm_get_next_action(memory) # this is the tool-calling part
    observations = execute_action(action)
    memory += [action, observations]
```

![](assets/agent.gif)

When to use/ not use Agents?

- When the workflow is deterministic, code it up without agents
- When the workflow is not deterministic, use agent. start simple.

Starting simple is key.

## What does smolagents offer?

- the system prompts, the addition of tools to it, parsing of tools *and* executing tool calls. all this is supported by the framework
- memory management.
- error handling and retrying. if there's an error (e.g. syntax error in the code produced by LLM), you need to retry again. this is supported.
- multi-step agent behavior (i.e. going in a while loop till we get the final answer)
- planning interval (at x intervals, the model to think back and reflect)
- uses code agents (i.e. LLMs giving code output instead of jsons of function arguments)

## How do Multi-Step Agents Work?

The ReAct framework is the main approach to building MSA.

All agents in smolagents are based on singular MultiStepAgent class, which is an abstraction of ReAct framework.

On a basic level, this class performs actions on a cycle of following steps, where existing variables and knowledge is incorporated into the agent logs like below:

Initialization: the system prompt is stored in a SystemPromptStep, and the user query is logged into a TaskStep .

```
While loop (ReAct loop) till final_answer tool is called:
    - Use agent.write_memory_to_messages() to write the agent logs into a list of LLM-readable chat messages.
    - Send these messages to a Model object to get its completion. Parse the completion to get the action
        (a JSON blob for ToolCallingAgent, a code snippet for CodeAgent).
    - Execute the action and logs result into memory (an ActionStep).
    - At the end of each step, we run all callback functions defined in agent.step_callbacks .
Optionally, when planning is activated, a plan can be periodically revised and stored in a PlanningStep . This includes feeding facts about the task at hand to the memory.
```

![](assets/codeagent.png)