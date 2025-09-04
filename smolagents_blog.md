# AI Agents That Can See the Future 🔮

![image/jpeg](https://cdn-uploads.huggingface.co/production/uploads/6684fb935b96dbe402f795d9/Dxmx9_Hpr-XcgrTiaLiJK.jpeg)

While the title may sound fantastical, the underlying technology is remarkably concrete. AI agents have gained a new capability that fundamentally changes how they interact with tools: predictive data structure awareness. Instead of executing a tool and then discovering what data it returns, agents can now know the exact output schema beforehand.

This shift from reactive inspection to predictive planning represents a significant architectural advancement in agent design.

## The Current Problem: Agents That Stumble in the Dark

Today's AI agents work like someone fumbling for a light switch in an unfamiliar room. They call a tool, inspect whatever data comes back, then figure out their next move. This trial-and-error approach creates a cascade of inefficiencies that plague modern agent systems.

The legacy pattern of injecting raw tool output (like large JSON blobs) into the context window isn’t just inefficient, it simply comes with many drawbacks:

- **Wastes tokens** on redundant data display
- **Reduces available context** for actual reasoning
- **Increases costs** for token-based models
- **Slows down processing** due to larger context
- **Causes instruction dilution** by burying key directives in noise
- **Dilutes attention** by forcing the model to spread focus across low-signal tokens

This "print-and-inspect" pattern forces agents into multi-step workflows where the first step is always exploratory, burning cycles and context just to understand what they're working with.

## The Breakthrough: Output Schema Changes Everything

Tools served over the Model Context Protocol (MCP) are now a common way to extend the capabilities of AI agents. The latest MCP specifications (2025-06-18+) introduced crucial enhancements: support for [Structured Content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) and the [Output Schema](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema).

This allows an agent to be aware of a tool's output data structure before it is ever called. Think of it as giving agents architectural blueprints instead of making them explore by touch.

## Why This Matters: The Code-First Revolution

Recent research, including [work](https://machinelearning.apple.com/research/codeact) from Apple and several other [studies](#further-reading), has converged on a key insight: agents that "think" in executable code consistently outperform agents that reason in natural language. This approach, known as **CodeAct**, results in more precise, reliable, and auditable agent behavior.

But here's the crucial connection: CodeAct agents need to know what they're working with to write effective code.

**Without Output Schema**: Agents write exploratory code → inspect results → write more code  
**With Output Schema**: Agents write precise, complete programs from the start

The open-source library [smolagents](https://huggingface.co/docs/smolagents/index) by **Hugging Face** exemplifies this synergy. As a lightweight framework for building CodeAct agents, it translates intent and tool usage directly into executable programs. When combined with Output Schema, the agent can immediately generate precise code that interacts with known data structures, rather than guessing or parsing raw strings.

This code-first methodology is a **natural fit for Output Schema**, as the agent can work with structured objects returned by tools instead of string manipulation.

## The Information Gap: What Agents Actually See

The difference becomes clear when we examine the **tool signatures** that agents receive in their **system prompts**. These function signatures are how agents discover all available tools and understand how they work - they're essentially the "API documentation" that gets injected directly into the agent's context.

When an agent starts up, it receives detailed descriptions of every tool at its disposal. The critical difference lies in how much structural information about the output is included in these descriptions.

Let's examine the tool signatures for `get_weather_info` - a tool that provides current weather data for any location. Agents can use this tool to answer user questions about weather conditions or temperatures.

**Without Output Schema:**
```python
def get_weather_info(city: string) -> object:
    """Get weather information for a location.

        Returns a JSON object as string with:
        - location (str): The location name
        - temperature (float): Temperature in Celsius
        - conditions (str): Weather conditions description
        - humidity (int): Humidity percentage (0-100)

    Args:
        city: The name of the city or location to get weather information for (e.g., 'New York')
    """
```

**With Output Schema:**
```python
def get_weather_info(city: string) -> dict:
    """Get weather information for a location.

        Returns a JSON object as string with:
        - location (str): The location name
        - temperature (float): Temperature in Celsius
        - conditions (str): Weather conditions description
        - humidity (int): Humidity percentage (0-100)

    Important: This tool returns structured output! Use the JSON schema below to directly access fields like result['field_name']. NO print() statements needed to inspect the output!

    Args:
        city: The name of the city or location to get weather information for (e.g., 'New York')

    Returns:
        dict (structured output): This tool ALWAYS returns a dictionary that strictly adheres to the following JSON schema:
            {
                "properties": {
                    "location": {
                        "description": "The location name",
                        "title": "Location",
                        "type": "string"
                    },
                    "temperature": {
                        "description": "Temperature in Celsius",
                        "title": "Temperature",
                        "type": "number"
                    },
                    "conditions": {
                        "description": "Weather conditions",
                        "title": "Conditions",
                        "type": "string"
                    },
                    "humidity": {
                        "description": "Humidity percentage",
                        "maximum": 100,
                        "minimum": 0,
                        "title": "Humidity",
                        "type": "integer"
                    }
                },
                "required": [
                    "location",
                    "temperature",
                    "conditions",
                    "humidity"
                ],
                "title": "WeatherInfo",
                "type": "object"
            }
    """
```

The contrast is striking. With Output Schema, the agent receives a complete blueprint of the data structure, explicit instructions about direct access patterns, and even guidance to avoid unnecessary print() statements. Without it, the agent only knows that "something JSON-like" will be returned as a string.

This information asymmetry explains why structured output enables single-step execution while legacy approaches require exploratory steps.

## From Theory to Practice: A Real Example

Let's illustrate the transformation with a simple task: `"What is the temperature in Tokyo in Fahrenheit?"`

### The Legacy "Print-and-Inspect" Pattern

Without Output Schema, the agent must first call the tool and print the result to "see" its structure.
Crucially, the tool's output is returned as a raw string that must be **interpreted by the agent LLM** or **parsed by a JSON parser** to be processed in a subsequent step where the agent generates the code for handling the tool output.

This becomes particularly problematic when tools return **extremely large JSON payloads**, imagine a database returning hundreds of records or an API call fetching detailed product catalogs. These massive string outputs flood the context window with raw data that the agent must handle.

This creates a multi-layered inefficiency: first the string parsing overhead, then the cognitive load of interpreting the structure, and finally the generation of appropriate handling code. Each layer adds latency, token consumption, and potential points of failure.

```text
━━━━━━━━━━━━━━━━━━━━━━━━━ Step 1 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  tokyo_weather = get_weather_info(city="Tokyo")                                        
  print(tokyo_weather)  # Raw string output that needs interpretation
 ──────────────────────────────────────────────────────────────────────────────────────
Execution logs:
{
  "location": "Tokyo",
  "temperature": 22.5,
  "conditions": "partly cloudy",
  "humidity": 65
}
[Step 1: Duration 1.32 seconds| Input tokens: 2,263 | Output tokens: 66]
```

```text
━━━━━━━━━━━━━━━━━━━━━━━━━ Step 2 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  celsius = 22.5  # Manually extracted from printed output
  fahrenheit = (celsius * 9/5) + 32                                                     
  final_answer(fahrenheit)                                                              
 ───────────────────────────────────────────────────────────────────────────────────────
Final answer: 72.5
```

This multi-step process is a direct result of the agent's blindness. **Step 1** is a costly, context-polluting operation purely for inspection.

### The New Approach: Direct Usage with Output Schema

**smolagents** now supports Output Schema for MCP tools. This gives the agent foreknowledge of the tool's return structure.

```text
━━━━━━━━━━━━━━━━━━━━━━━━━ Step 1 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  weather_info = get_weather_info(city="Tokyo")                                         
  celsius = weather_info["temperature"]                                                 
  fahrenheit = (celsius * 9/5) + 32                                                     
  final_answer(fahrenheit)                                                              
 ──────────────────────────────────────────────────────────────────────────────────────
Execution logs:
{'location': 'Tokyo', 'temperature': 22.5, 'conditions': 'partly cloudy', 'humidity': 65}
Final answer: 72.5
```

The agent generates code for a **single, efficient step**, directly accessing `weather_info["temperature"]` because it knows the schema. 

Crucially, the newly introduced **structured output provides structured objects the agent can directly handle** - no need to use a JSON parser to create objects from a returned string containing raw JSON. The context remains clean, and the agent can work immediately with native structured data types.

## The Results Speak for Themselves

We benchmarked multiple AI models comparing structured vs. unstructured output performance:

### Performance Comparison: Structured vs Unstructured Output

| Model | Structured Output |  |  |  |  | Unstructured Output |  |  |  |  |
|-------|------------------|--|--|--|--|-------------------|--|--|--|--|
| | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** | **Avg Time** | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** | **Avg Time** |
| **Mistral Small** | 100.0% | 1.10 | 1 | 2 | 1.43s | 70.0% | 4.71 | 4 | 6 | 12.16s |
| **GPT-4.1** | 100.0% | 1.20 | 1 | 2 | 2.48s | 100.0% | 2.30 | 2 | 3 | 5.37s |
| **Gemini 2.5 Flash** | 100.0% | 1.10 | 1 | 2 | 3.16s | 100.0% | 2.60 | 2 | 3 | 6.01s |
| **Gemma 3 27B** | 100.0% | 1.80 | 1 | 2 | 3.86s | 100.0% | 2.40 | 2 | 3 | 5.49s |
| **Qwen3-4B-Instruct-2507** | 100.0% | 1.70 | 1 | 2 | 4.15s | 100.0% | 3.60 | 2 | 6 | 8.14s |

The numbers are striking: **step count reductions of 25-77%** and **execution speed improvements of 30-88%**. Most importantly, structured output achieved 100% success rates across all models, while unstructured approaches showed reliability issues.

**Test Parameters**: Maximum of 20 steps per task. Mistral Small failed 3/10 runs in unstructured mode by exceeding this limit, while achieving perfect reliability with structured output.

## Getting Started with smolagents

To enable structured output and output schema support, simply pass `structured_output=True` when initializing the `MCPClient`:

```python
from smolagents import MCPClient, CodeAgent

# Enable structured output support
with MCPClient(server_parameters, structured_output=True) as tools:
    agent = CodeAgent(tools=tools, model=model, add_base_tools=True)
    agent.run("What is the temperature in Tokyo in Fahrenheit?")
```

When `structured_output=True`, the following features are enabled:
* **Output Schema Support**: Tools can define JSON schemas for their outputs
* **Structured Content Handling**: Support for structuredContent in MCP responses  
* **JSON Parsing**: Automatic parsing of structured data from tool responses

Learn more: [Hugging Face smolagents: Structured Output and Output Schema Support](https://huggingface.co/docs/smolagents/main/en/tutorials/tools#structured-output-and-output-schema-support)

## Conclusion: The End of Trial-and-Error Agents

What we're witnessing isn't just an incremental improvement, it's the emergence of **predictive agents** that can plan their entire execution path before taking the first step. By knowing exactly what data structure a tool will return, agents can write precise, single-step programs instead of fumbling through multi-step trial-and-error processes.

The benchmark results tell a compelling story: **77% fewer steps, 88% faster execution, and 100% reliability** across multiple models. But the real breakthrough is qualitative. We've moved from agents that stumble through tasks like a person searching for light switches in a dark room, to agents that can see the entire room layout before they enter.

This shift has immediate practical implications for anyone building with AI agents:

*   **Cost Efficiency:** Dramatic token savings translate directly to lower operational costs, especially at scale
*   **User Experience:** Sub-second responses replace multi-step delays, making agents feel truly responsive
*   **Production Readiness:** Predictable, single-step execution makes agents suitable for real-time applications

Perhaps most importantly, Output Schema represents a fundamental architectural evolution. We're transitioning from **reactive agents** (that respond to whatever data they receive) to **proactive agents** (that can anticipate and prepare for specific data structures). This predictive capability is what transforms agents from clever chatbots into reliable automation partners.

The "future-seeing" metaphor from our title isn't hyperbole, it's the core advantage that makes everything else possible. When agents can anticipate rather than just react, we unlock a new class of autonomous systems that are finally ready for the reliability demands of production environments.

The age of guessing is over. The age of knowing has begun.

## Acknowledgments
Special thanks to **Albert Villanova del Moral** and **Guillaume Raille**, for their comprehensive support throughout this work. Their feedback, guidance, and thorough reviews were essential in bringing these structured output and output schema capabilities to fruition.

---
## Further Reading
* [Executable Code Actions Elicit Better LLM Agents](https://arxiv.org/abs/2402.01030)
* [DynaSaur: Large Language Agents Beyond Predefined Actions](https://arxiv.org/abs/2411.01747)
* [If LLM Is the Wizard, Then Code Is the Wand: A Survey on How Code Empowers Large Language Models to Serve as Intelligent Agents](https://arxiv.org/abs/2401.00812)
