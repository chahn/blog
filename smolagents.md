*August 2025 - Christian Hahn*

# **AI Agents That Can See the Future 🔮**

Okay, the title is clickbait, but the core premise is real. A new capability is being brought to AI agents that is kind of a game-changer: the ability to know the structure of a tool's output *before* executing it. This is a fundamental shift in agent efficiency.

Tools served over the Model Context Protocol (MCP) are now a common way to extend the capabilities of AI agents. The latest MCP specifications (2025-06-18+) introduced a crucial enhancement: support for [Structured Content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) and the [Output Schema](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema). This allows an agent to be aware of a tool's output data structure before it is ever called.

## **The Rise of Code-Thinking Agents**

In 2024, several [research papers](#further-reading), including [work](https://machinelearning.apple.com/research/codeact) from Apple, converged on a key insight: agents that "think" in executable code (Code Generation) consistently outperform agents that reason in natural language. This approach, known as **CodeAct**, results in more precise, reliable, and auditable agent behavior.

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

| Model | Structured Output |  |  |  |  | Unstructured Output |  |  |  |  |
|-------|------------------|--|--|--|--|-------------------|--|--|--|--|
| | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** | **Avg Time** | **Success Rate** | **Avg Steps** | **Min Steps** | **Max Steps** | **Avg Time** |
| **Mistral Small** | 100.0% | 1.10 | 1 | 2 | 1.43s | 70.0% | 4.71 | 4 | 6 | 12.16s |
| **GPT-4.1** | 100.0% | 1.20 | 1 | 2 | 2.48s | 100.0% | 2.30 | 2 | 3 | 5.37s |
| **Gemini 2.5 Flash** | 100.0% | 1.10 | 1 | 2 | 3.16s | 100.0% | 2.60 | 2 | 3 | 6.01s |
| **Gemma 3 27B** | 100.0% | 1.80 | 1 | 2 | 3.86s | 100.0% | 2.40 | 2 | 3 | 5.49s |
| **Qwen3-4B-Instruct-2507** | 100.0% | 1.70 | 1 | 2 | 4.15s | 100.0% | 3.60 | 2 | 6 | 8.14s |

### Test Parameters

**Step Limit**: Maximum of 20 steps to complete each task. Tasks exceeding this limit are marked as failed.

**Max Step Failures**: 
- Mistral Small (Unstructured): 3/10 runs exceeded 20 steps and failed
- All other model configurations: 0 failures

**Note**: Min/Max steps in the table represent only successful completions. Failed runs that exceeded 20 steps are excluded from step count statistics.

### Benchmark Discussion

In these benchmarks, all agent models performed significantly better using structured output compared to unstructured output. The tasks were consistently completed faster and with fewer steps - most models achieved near-optimal 1-2 step completion rates with structured output, while unstructured approaches required 2-5+ steps on average. 

The time improvements were equally impressive, with execution speed improvements ranging from 30% to 88%. Mistral Small demonstrated the most dramatic performance difference: lightning-fast 1.43s execution with structured output versus sluggish 12.16s with unstructured output, though this came with reliability concerns.

Structured output not only improved efficiency but also enhanced reliability, with all models achieving 100% success rates versus the concerning 70% success rate observed with Mistral Small in unstructured mode. The performance gains were substantial across all tested models, with step count reductions ranging from 25% to 77%.

For production agent deployments, implementing structured output schemas appears to be a critical optimization that delivers improved performance, faster execution times, and predictable execution patterns. The reliability benefits are particularly important, as demonstrated by Mistral Small's 30% failure rate in unstructured mode versus perfect reliability with structured output.

## **Conclusion: The End of Trial-and-Error Agents**

What we're witnessing isn't just an incremental improvement—it's the emergence of **predictive agents** that can plan their entire execution path before taking the first step. By knowing exactly what data structure a tool will return, agents can write precise, single-step programs instead of fumbling through multi-step trial-and-error processes.

The benchmark results tell a compelling story: **77% fewer steps, 88% faster execution, and 100% reliability** across multiple models. But the real breakthrough is qualitative. We've moved from agents that stumble through tasks like a person searching for light switches in a dark room, to agents that can see the entire room layout before they enter.

This shift has immediate practical implications for anyone building with AI agents:

*   **Cost Efficiency:** Dramatic token savings translate directly to lower operational costs, especially at scale
*   **User Experience:** Sub-second responses replace multi-step delays, making agents feel truly responsive
*   **Production Readiness:** Predictable, single-step execution makes agents suitable for real-time applications

Perhaps most importantly, Output Schema represents a fundamental architectural evolution. We're transitioning from **reactive agents** (that respond to whatever data they receive) to **proactive agents** (that can anticipate and prepare for specific data structures). This predictive capability is what transforms agents from clever chatbots into reliable automation partners.

The "future-seeing" metaphor from our title isn't hyperbole—it's the core advantage that makes everything else possible. When agents can anticipate rather than just react, we unlock a new class of autonomous systems that are finally ready for the reliability demands of production environments.

The age of guessing is over. The age of knowing has begun.

---
## Further Reading
* [Executable Code Actions Elicit Better LLM Agents](https://arxiv.org/abs/2402.01030)
* [DynaSaur: Large Language Agents Beyond Predefined Actions](https://arxiv.org/abs/2411.01747)
* [If LLM Is the Wizard, Then Code Is the Wand: A Survey on How Code Empowers Large Language Models to Serve as Intelligent Agents](https://arxiv.org/abs/2401.00812)
