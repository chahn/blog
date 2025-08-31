*August 2025 - Christian Hahn*

# **AI Agents That Can See the Future 🔮**

Okay, the title is clickbait, but the core premise is real. A new capability is being brought to AI agents that is kind of a game-changer: the ability to know the structure of a tool's output *before* executing it. This is a fundamental shift in agent efficiency.

Tools served over the Model Context Protocol (MCP) are now a common way to extend the capabilities of AI agents. The latest MCP specifications (2025-06-18+) introduced a crucial enhancement: support for [Structured Content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) and the [Output Schema](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema). This allows an agent to be aware of a tool's output data structure before it is ever called.

## **The Rise of Code-Thinking Agents**

In 2024, several [research papers](#further-reading), including [notable work](https://machinelearning.apple.com/research/codeact) from Apple, converged on a key insight: agents that "think" in executable code (Code Generation) consistently outperform agents that reason in natural language. This approach, known as **CodeAct**, results in more precise, reliable, and auditable agent behavior.

The open-source library [smolagents](https://huggingface.co/docs/smolagents/index) by **Hugging Face** is a lightweight and powerful framework for building CodeAct agents. Its core advantage is that the agent translates its intent and tool usage directly into an executable program. This code-first methodology is a **natural fit for the Output Schema**, as the agent can immediately generate precise code that interacts with the known data structure, rather than guessing or parsing raw strings. In addition **smolagents** can make use of the objects returned by the tools by using the structured output.

## **The Problems of the "Print-and-Inspect" Pattern**

The legacy pattern of injecting raw tool output (like large JSON blobs) into the context window is not just inefficient; it comes with different disadvantages:

- **Wastes tokens** on redundant data display
- **Reduces available context** for actual reasoning
- **Increases costs** for token-based models
- **Slows down processing** due to larger context
- **Causes instruction dilution** by burying key directives in noise.
- **Dilutes attention** by forcing the model to spread focus across low-signal tokens.
  
## **A Practical Example: From a Multi-Step to a Single-Step Workflow**

Let's illustrate the problem and the solution with a simple task: 

> "What is the temperature in Tokyo in Fahrenheit?"

### **The Legacy "Print-and-Inspect" Pattern**

Without Output Schema, the agent must first call the tool and print the result to "see" its structure.

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ Step 1 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  tokyo_weather = get_weather_info(city="Tokyo")                                        
  print(tokyo_weather)                                                                  
 ──────────────────────────────────────────────────────────────────────────────────────
Execution logs:
{
  "location": "Tokyo",
  "temperature": 22.5,
  "conditions": "partly cloudy",
  "humidity": 65
}
[Step 1: Duration 1.32 seconds| Input tokens: 2,263 | Output tokens: 66]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ Step 2 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  celsius = 22.5                                                                        
  fahrenheit = (celsius * 9/5) + 32                                                     
  final_answer(fahrenheit)                                                              
 ───────────────────────────────────────────────────────────────────────────────────────
Final answer: 72.5
```
This multi-step process is a direct result of the problems listed above. **Step 1** is a costly, context-polluting operation purely for inspection.

### **The Modern Approach: Direct Usage with Output Schema**

**smolagents** now supports Output Schema for MCP tools. This gives the agent foreknowledge of the tool's return structure.

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ Step 1 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ─ Executing parsed code: ──────────────────────────────────────────────────────────────
  weather_info = get_weather_info(city="Tokyo")                                         
  print(weather_info)                                                                   
  celsius = weather_info["temperature"]                                                 
  fahrenheit = (celsius * 9/5) + 32                                                     
  final_answer(fahrenheit)                                                              
 ──────────────────────────────────────────────────────────────────────────────────────
Execution logs:
{'location': 'Tokyo', 'temperature': 22.5, 'conditions': 'partly cloudy', 'humidity': 65}
Final answer: 72.5
```

The agent generates code for a **single, efficient step**, directly accessing `weather_info["temperature"]` because it knows the schema. The context remains clean.

In addition smolagents can directly use the object returned by the tool, because structured output is now supported.

## **How to Use Structured Output and Output Schema Support in smolagents**

To enable structured output and output schema support, simply pass `structured_output=True` when initializing the `MCPClient`:
```python
from smolagents import MCPClient, CodeAgent

# Enable structured output support
with MCPClient(server_parameters, structured_output=True) as tools:
    agent = CodeAgent(tools=tools, model=model, add_base_tools=True)
    agent.run("What is the temperature in Tokyo in Fahrenheit?")
```

When `structured_output=True`, the following features are enabled:
* Output Schema Support: Tools can define JSON schemas for their outputs
* Structured Content Handling: Support for structuredContent in MCP responses
* JSON Parsing: Automatic parsing of structured data from tool responses

Learn more: [Hugging Face smolagents: Structured Output and Output Schema Support](https://huggingface.co/docs/smolagents/main/en/tutorials/tools#structured-output-and-output-schema-support)

## Agent Model Benchmark Results

### Performance Comparison: Structured vs Unstructured Output

| Model | Structured Output |  |  |  | Unstructured Output |  |  |  |
|-------|------------------|--|--|--|-------------------|--|--|--|
| | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** |
| **Gemini 2.5 Flash** | 100.0% | 1.00 | 1 | 1 | 100.0% | 2.80 | 2 | 3 |
| **Mistral Small** | 100.0% | 1.00 | 1 | 1 | 80.0% | 4.00 | 3 | 5 |
| **GPT-4.1** | 100.0% | 1.20 | 1 | 2 | 100.0% | 2.60 | 2 | 3 |
| **Qwen3-4B-Instruct-2507** | 100.0% | 1.60 | 1 | 2 | 100.0% | 2.60 | 2 | 4 |
| **Gemma 3 27B** | 100.0% | 1.60 | 1 | 2 | 100.0% | 3.60 | 2 | 10 |

### Test Parameters

**Step Limit**: Maximum of 20 steps to complete each task. Tasks exceeding this limit are marked as failed.

**Max Step Failures**: 
- Mistral Small (Unstructured): 1/5 runs exceeded 20 steps and failed
- All other model configurations: 0 failures

**Note**: Min/Max steps in the table represent only successful completions. Failed runs that exceeded 20 steps are excluded from step count statistics.

### Conclusion
In these benchmarks, all agent models performed significantly better using structured output compared to unstructured output. The tasks were consistently completed faster and with fewer steps - most models achieved optimal 1-step completion rates with structured output, while unstructured approaches required 2-4+ steps on average.

Structured output not only improved efficiency but also enhanced reliability, with all models achieving 100% success rates versus the 80% success rate observed with Mistral Small in unstructured mode. The performance gains were substantial across all tested models, with step count reductions ranging from 38% to 75%.

For production agent deployments, implementing structured output schemas appears to be an optimization that delivers both improved performance and predictable execution patterns.

## **Conclusion: Faster, Leaner, and More Reliable Agents**

Adopting Output Schema is an incremental improvement that yields significant, measurable advantages:

*   **Reduced Latency and Higher Throughput:** By keeping the context lean, leading to faster and more scalable agent performance.
*   **Optimized Token and Memory Footprint:** Eliminating raw JSON from the context directly translates to lower token counts and reduced financial costs.
*   **Improved Model Focus and Reliability:** By preventing context pollution and instruction dilution, the agent can maintain focus on its core task, increasing the probability of successful task completion.

This development makes AI agents not just more powerful, but brings us a critical step closer to truly autonomous and efficient systems.

---
## Further Reading
* [Executable Code Actions Elicit Better LLM Agents](https://arxiv.org/abs/2402.01030)
* [DynaSaur: Large Language Agents Beyond Predefined Actions](https://arxiv.org/abs/2411.01747)
* [If LLM Is the Wizard, Then Code Is the Wand: A Survey on How Code Empowers Large Language Models to Serve as Intelligent Agents](https://arxiv.org/abs/2401.00812)

* [MCP Specification: Output Schema for Tools](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema)
