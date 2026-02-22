# AI Models Guide

Complete guide to supported AI models, capabilities, and selection guidance in ResonantGenesis.

## Supported Models

### OpenAI Models

| Model | Context | Best For | Cost |
|-------|---------|----------|------|
| `gpt-4` | 8K | Complex reasoning | $$$ |
| `gpt-4-turbo` | 128K | Long context tasks | $$ |
| `gpt-4o` | 128K | Multimodal, fast | $$ |
| `gpt-3.5-turbo` | 16K | Simple tasks | $ |

### Anthropic Models

| Model | Context | Best For | Cost |
|-------|---------|----------|------|
| `claude-3-opus` | 200K | Complex analysis | $$$ |
| `claude-3-sonnet` | 200K | Balanced tasks | $$ |
| `claude-3-haiku` | 200K | Fast, simple tasks | $ |

### Open Source Models

| Model | Context | Best For | Cost |
|-------|---------|----------|------|
| `llama-3-70b` | 8K | Self-hosted | Free* |
| `llama-3-8b` | 8K | Edge deployment | Free* |
| `mixtral-8x7b` | 32K | Code generation | Free* |
| `codellama-34b` | 16K | Code tasks | Free* |

*Compute costs apply for self-hosted models

## Model Capabilities

### Capability Matrix

| Capability | GPT-4 | GPT-4 Turbo | Claude 3 Opus | Claude 3 Sonnet |
|------------|-------|-------------|---------------|-----------------|
| Reasoning | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Coding | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Writing | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Math | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Speed | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | ★★★★☆ |
| Cost | ★★☆☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★☆☆ |

### Feature Support

| Feature | GPT-4 | Claude 3 | Llama 3 |
|---------|-------|----------|---------|
| Function calling | ✅ | ✅ | ✅ |
| JSON mode | ✅ | ✅ | ✅ |
| Streaming | ✅ | ✅ | ✅ |
| Vision | ✅ | ✅ | ❌ |
| System prompts | ✅ | ✅ | ✅ |

## Selecting Models

### By Use Case

```python
# Simple Q&A - use fast, cheap model
agent = client.agents.create(
    name="FAQ Bot",
    model="gpt-3.5-turbo"
)

# Complex analysis - use powerful model
agent = client.agents.create(
    name="Research Analyst",
    model="gpt-4-turbo"
)

# Code generation - use code-optimized model
agent = client.agents.create(
    name="Code Assistant",
    model="gpt-4-turbo"  # or claude-3-opus
)

# Long document processing - use large context
agent = client.agents.create(
    name="Document Processor",
    model="claude-3-sonnet"  # 200K context
)
```

### By Budget

| Budget | Recommended Models |
|--------|-------------------|
| Minimal | gpt-3.5-turbo, claude-3-haiku |
| Moderate | gpt-4-turbo, claude-3-sonnet |
| Premium | gpt-4, claude-3-opus |

### By Speed

| Speed Need | Recommended Models |
|------------|-------------------|
| Real-time | gpt-3.5-turbo, claude-3-haiku |
| Interactive | gpt-4-turbo, claude-3-sonnet |
| Batch | gpt-4, claude-3-opus |

## Model Configuration

### Setting Default Model

```python
# Set at agent creation
agent = client.agents.create(
    name="My Agent",
    model="gpt-4-turbo",
    model_config={
        "temperature": 0.7,
        "max_tokens": 4000
    }
)
```

### Override Per Session

```python
# Override model for specific session
result = agent.execute(
    goal="Complex task",
    model="gpt-4",  # Override agent default
    model_config={
        "temperature": 0.3
    }
)
```

### Model Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `temperature` | float | 0.7 | Randomness (0-2) |
| `max_tokens` | int | 4000 | Max output tokens |
| `top_p` | float | 1.0 | Nucleus sampling |
| `frequency_penalty` | float | 0 | Repetition penalty |
| `presence_penalty` | float | 0 | Topic diversity |

### Example Configuration

```python
agent = client.agents.create(
    name="Creative Writer",
    model="gpt-4-turbo",
    model_config={
        "temperature": 0.9,      # More creative
        "max_tokens": 8000,      # Longer outputs
        "top_p": 0.95,
        "frequency_penalty": 0.5 # Less repetition
    }
)

agent = client.agents.create(
    name="Code Generator",
    model="gpt-4-turbo",
    model_config={
        "temperature": 0.2,      # More deterministic
        "max_tokens": 4000,
        "top_p": 1.0
    }
)
```

## Model Fallbacks

### Automatic Fallback

```python
# Configure fallback chain
agent = client.agents.create(
    name="Resilient Agent",
    model="gpt-4-turbo",
    fallback_models=["gpt-4", "gpt-3.5-turbo"]
)

# If gpt-4-turbo fails, tries gpt-4, then gpt-3.5-turbo
```

### Manual Fallback

```python
async def execute_with_fallback(agent, goal):
    models = ["gpt-4-turbo", "gpt-4", "gpt-3.5-turbo"]
    
    for model in models:
        try:
            return await agent.execute(goal=goal, model=model)
        except ModelUnavailableError:
            continue
    
    raise Exception("All models unavailable")
```

## Cost Management

### Token Pricing

| Model | Input (per 1M) | Output (per 1M) |
|-------|----------------|-----------------|
| gpt-4 | $30.00 | $60.00 |
| gpt-4-turbo | $10.00 | $30.00 |
| gpt-4o | $5.00 | $15.00 |
| gpt-3.5-turbo | $0.50 | $1.50 |
| claude-3-opus | $15.00 | $75.00 |
| claude-3-sonnet | $3.00 | $15.00 |
| claude-3-haiku | $0.25 | $1.25 |

### Cost Estimation

```python
def estimate_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    pricing = {
        "gpt-4": {"input": 30.0, "output": 60.0},
        "gpt-4-turbo": {"input": 10.0, "output": 30.0},
        "gpt-3.5-turbo": {"input": 0.5, "output": 1.5},
        "claude-3-opus": {"input": 15.0, "output": 75.0},
        "claude-3-sonnet": {"input": 3.0, "output": 15.0}
    }
    
    rates = pricing.get(model, pricing["gpt-4-turbo"])
    cost = (input_tokens / 1_000_000 * rates["input"] +
            output_tokens / 1_000_000 * rates["output"])
    return cost

# Example
cost = estimate_cost("gpt-4-turbo", 5000, 2000)
print(f"Estimated cost: ${cost:.4f}")
```

### Cost Optimization

1. **Use appropriate models** - Don't use GPT-4 for simple tasks
2. **Optimize prompts** - Shorter prompts = fewer tokens
3. **Set token limits** - Prevent runaway costs
4. **Cache responses** - Avoid duplicate calls
5. **Monitor usage** - Track costs regularly

```python
# Set strict limits
result = agent.execute(
    goal="Task",
    max_tokens=2000,  # Limit output
    model="gpt-3.5-turbo"  # Use cheaper model
)
```

## Custom Models

### Bring Your Own Model

```python
# Configure custom model endpoint
client.models.register(
    name="my-custom-model",
    endpoint="https://my-model-server.com/v1",
    api_key="my_api_key",
    config={
        "type": "openai-compatible",
        "max_tokens": 8000
    }
)

# Use custom model
agent = client.agents.create(
    name="Custom Agent",
    model="my-custom-model"
)
```

### Self-Hosted Models

```python
# Register self-hosted Llama
client.models.register(
    name="local-llama",
    endpoint="http://localhost:8080/v1",
    config={
        "type": "openai-compatible",
        "context_length": 8192
    }
)
```

### Fine-Tuned Models

```python
# Use fine-tuned OpenAI model
agent = client.agents.create(
    name="Specialized Agent",
    model="ft:gpt-3.5-turbo:my-org:custom:id"
)
```

## Model Comparison

### Benchmark Results

| Task | GPT-4 Turbo | Claude 3 Opus | GPT-3.5 Turbo |
|------|-------------|---------------|---------------|
| Code generation | 92% | 91% | 78% |
| Reasoning | 94% | 95% | 72% |
| Summarization | 89% | 91% | 82% |
| Translation | 88% | 87% | 85% |
| Math | 91% | 93% | 68% |

### Latency Comparison

| Model | Avg Latency | P99 Latency |
|-------|-------------|-------------|
| gpt-3.5-turbo | 0.8s | 2.1s |
| gpt-4-turbo | 2.5s | 6.0s |
| gpt-4 | 4.2s | 12.0s |
| claude-3-haiku | 0.6s | 1.5s |
| claude-3-sonnet | 1.8s | 4.5s |
| claude-3-opus | 3.5s | 9.0s |

## Best Practices

### Model Selection

1. **Start simple** - Begin with cheaper models
2. **Test thoroughly** - Compare model outputs
3. **Consider context** - Match context length to task
4. **Monitor quality** - Track output quality over time

### Configuration

1. **Temperature** - Lower for factual, higher for creative
2. **Max tokens** - Set based on expected output
3. **Fallbacks** - Always configure backup models

### Cost Control

1. **Set budgets** - Use spending limits
2. **Monitor usage** - Review daily/weekly
3. **Optimize prompts** - Reduce token usage
4. **Cache results** - Avoid redundant calls

## API Reference

### List Models

```bash
GET /api/v1/models
```

### Get Model Info

```bash
GET /api/v1/models/{model_id}
```

### Register Custom Model

```bash
POST /api/v1/models
```

### Model Usage

```bash
GET /api/v1/models/{model_id}/usage
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
