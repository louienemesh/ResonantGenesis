# Use Cases Guide

Practical implementation patterns for common ResonantGenesis use cases.

## Table of Contents

- [Customer Support](#customer-support)
- [Content Creation](#content-creation)
- [Research & Analysis](#research--analysis)
- [Code Assistance](#code-assistance)
- [Data Processing](#data-processing)
- [Workflow Automation](#workflow-automation)
- [Sales & Marketing](#sales--marketing)
- [Education & Training](#education--training)

---

## Customer Support

### Automated Ticket Triage

```python
# Create support triage agent
triage_agent = client.agents.create(
    name="Support Triage",
    system_prompt="""You are a support ticket triage specialist.

Analyze incoming tickets and:
1. Categorize by type (billing, technical, account, general)
2. Assess priority (critical, high, medium, low)
3. Extract key information
4. Suggest initial response

Output as JSON:
{
    "category": "...",
    "priority": "...",
    "summary": "...",
    "key_info": {...},
    "suggested_response": "...",
    "escalate": true/false
}
""",
    model="gpt-4-turbo"
)

# Process incoming ticket
result = triage_agent.execute(
    goal=f"Triage this support ticket: {ticket_content}",
    context={
        "customer_tier": customer.tier,
        "previous_tickets": customer.ticket_count
    }
)
```

### Live Chat Assistant

```python
# Create chat support agent
chat_agent = client.agents.create(
    name="Chat Support",
    system_prompt="""You are a friendly customer support agent.

Guidelines:
- Be helpful and empathetic
- Provide accurate information
- Escalate complex issues to humans
- Never make promises you can't keep

You have access to:
- Product documentation
- FAQ database
- Order lookup
""",
    tools=["web_search"],
    model="gpt-4-turbo"
)

# Handle chat message
async def handle_chat(session_id: str, message: str):
    result = await chat_agent.execute(
        goal=f"Respond to customer: {message}",
        context={
            "session_history": get_chat_history(session_id),
            "customer_info": get_customer_info(session_id)
        }
    )
    return result.output
```

### Email Response Generator

```python
# Create email response agent
email_agent = client.agents.create(
    name="Email Responder",
    system_prompt="""You draft professional email responses.

Match the tone of the original email.
Be concise but thorough.
Include relevant links when helpful.
Sign off appropriately.
""",
    model="gpt-4-turbo"
)

# Generate response
result = email_agent.execute(
    goal="Draft a response to this customer email",
    context={
        "email": incoming_email,
        "customer_name": customer.name,
        "account_status": customer.status,
        "relevant_docs": search_knowledge_base(incoming_email)
    }
)
```

---

## Content Creation

### Blog Post Writer

```python
# Create content team
researcher = client.agents.create(
    name="Content Researcher",
    system_prompt="You research topics thoroughly and gather key facts.",
    tools=["web_search"]
)

writer = client.agents.create(
    name="Blog Writer",
    system_prompt="""You write engaging blog posts.

Style:
- Conversational but professional
- Use headers and bullet points
- Include practical examples
- 1000-1500 words
"""
)

editor = client.agents.create(
    name="Content Editor",
    system_prompt="""You edit and improve content.

Focus on:
- Clarity and flow
- Grammar and spelling
- SEO optimization
- Engaging headlines
"""
)

# Create content pipeline
async def create_blog_post(topic: str):
    # Research
    research = await researcher.execute(
        goal=f"Research: {topic}. Gather key facts, statistics, and expert opinions."
    )
    
    # Write
    draft = await writer.execute(
        goal=f"Write a blog post about {topic}",
        context={"research": research.output}
    )
    
    # Edit
    final = await editor.execute(
        goal="Edit and improve this blog post",
        context={"draft": draft.output}
    )
    
    return final.output
```

### Social Media Manager

```python
# Create social media agent
social_agent = client.agents.create(
    name="Social Media Manager",
    system_prompt="""You create engaging social media content.

Platforms:
- Twitter: 280 chars, hashtags, engaging
- LinkedIn: Professional, longer form
- Instagram: Visual descriptions, emojis

Always include:
- Call to action
- Relevant hashtags
- Engagement hooks
"""
)

# Generate posts
result = social_agent.execute(
    goal="Create social media posts for our new product launch",
    context={
        "product": product_info,
        "target_audience": "tech professionals",
        "platforms": ["twitter", "linkedin"],
        "brand_voice": "innovative, friendly"
    }
)
```

### Documentation Generator

```python
# Create docs agent
docs_agent = client.agents.create(
    name="Documentation Writer",
    system_prompt="""You write technical documentation.

Format:
- Clear headings
- Code examples
- Step-by-step instructions
- Troubleshooting sections
""",
    tools=["code_exec"]
)

# Generate API docs
result = docs_agent.execute(
    goal="Document this API endpoint",
    context={
        "endpoint": "/api/v1/agents",
        "method": "POST",
        "request_schema": request_schema,
        "response_schema": response_schema
    }
)
```

---

## Research & Analysis

### Market Research

```python
# Create research agent
market_agent = client.agents.create(
    name="Market Researcher",
    system_prompt="""You conduct thorough market research.

Analyze:
- Market size and growth
- Key players and competitors
- Trends and opportunities
- Risks and challenges

Cite sources for all data.
""",
    tools=["web_search"]
)

# Conduct research
result = market_agent.execute(
    goal="Research the AI agent market for 2026",
    max_steps=15
)
```

### Competitive Analysis

```python
# Create competitive analysis agent
competitive_agent = client.agents.create(
    name="Competitive Analyst",
    system_prompt="""You analyze competitors.

For each competitor, identify:
- Products and pricing
- Strengths and weaknesses
- Market positioning
- Recent news and developments

Present as a comparison matrix.
""",
    tools=["web_search"]
)

# Analyze competitors
result = competitive_agent.execute(
    goal="Analyze our top 5 competitors in the AI agent space",
    context={
        "our_product": product_info,
        "competitors": ["Company A", "Company B", "Company C"]
    }
)
```

### Data Analysis

```python
# Create data analyst agent
analyst_agent = client.agents.create(
    name="Data Analyst",
    system_prompt="""You analyze data and provide insights.

Approach:
1. Understand the data structure
2. Identify patterns and anomalies
3. Calculate key metrics
4. Visualize findings
5. Provide actionable recommendations
""",
    tools=["code_exec"],
    tool_config={
        "code_exec": {
            "languages": ["python"],
            "allowed_packages": ["pandas", "numpy", "matplotlib"]
        }
    }
)

# Analyze sales data
result = analyst_agent.execute(
    goal="Analyze Q4 sales data and identify trends",
    context={
        "data_path": "/data/sales_q4.csv",
        "metrics": ["revenue", "units", "growth"]
    }
)
```

---

## Code Assistance

### Code Review

```python
# Create code reviewer agent
reviewer_agent = client.agents.create(
    name="Code Reviewer",
    system_prompt="""You review code for quality and issues.

Check for:
- Bugs and logic errors
- Security vulnerabilities
- Performance issues
- Code style and best practices
- Test coverage

Provide specific, actionable feedback.
""",
    tools=["code_exec"]
)

# Review pull request
result = reviewer_agent.execute(
    goal="Review this code change",
    context={
        "diff": pr_diff,
        "language": "python",
        "project_style_guide": style_guide
    }
)
```

### Bug Fixer

```python
# Create bug fixer agent
fixer_agent = client.agents.create(
    name="Bug Fixer",
    system_prompt="""You diagnose and fix bugs.

Process:
1. Understand the bug report
2. Reproduce the issue
3. Identify root cause
4. Implement fix
5. Verify solution

Explain your reasoning.
""",
    tools=["code_exec", "file_access"]
)

# Fix bug
result = fixer_agent.execute(
    goal="Fix this bug",
    context={
        "bug_report": bug_description,
        "error_log": error_log,
        "relevant_code": code_snippet
    }
)
```

### Test Generator

```python
# Create test generator agent
test_agent = client.agents.create(
    name="Test Generator",
    system_prompt="""You write comprehensive tests.

Include:
- Unit tests for all functions
- Edge cases
- Error handling tests
- Integration tests where needed

Use pytest for Python, Jest for JavaScript.
""",
    tools=["code_exec"]
)

# Generate tests
result = test_agent.execute(
    goal="Generate tests for this module",
    context={
        "code": module_code,
        "language": "python",
        "framework": "pytest"
    }
)
```

---

## Data Processing

### ETL Pipeline

```python
# Create ETL agent
etl_agent = client.agents.create(
    name="ETL Processor",
    system_prompt="""You process and transform data.

Steps:
1. Extract data from source
2. Clean and validate
3. Transform to target format
4. Load to destination

Handle errors gracefully and log progress.
""",
    tools=["code_exec", "file_access", "database"]
)

# Process data
result = etl_agent.execute(
    goal="Process daily sales data from CSV to database",
    context={
        "source": "/data/daily_sales.csv",
        "destination": "sales_db.transactions",
        "transformations": ["normalize_dates", "calculate_totals"]
    }
)
```

### Report Generator

```python
# Create report agent
report_agent = client.agents.create(
    name="Report Generator",
    system_prompt="""You generate business reports.

Include:
- Executive summary
- Key metrics with visualizations
- Trend analysis
- Recommendations

Format as PDF-ready Markdown.
""",
    tools=["code_exec", "database"]
)

# Generate weekly report
result = report_agent.execute(
    goal="Generate weekly performance report",
    context={
        "period": "2026-02-14 to 2026-02-21",
        "metrics": ["revenue", "users", "engagement"],
        "comparison": "week_over_week"
    }
)
```

---

## Workflow Automation

### Approval Workflow

```python
# Create approval agent
approval_agent = client.agents.create(
    name="Approval Processor",
    system_prompt="""You process approval requests.

For each request:
1. Validate required information
2. Check against policies
3. Route to appropriate approver
4. Track status

Auto-approve if within policy limits.
"""
)

# Process approval
result = approval_agent.execute(
    goal="Process this expense approval request",
    context={
        "request": expense_request,
        "requester": employee_info,
        "policy": expense_policy,
        "budget_remaining": department_budget
    }
)
```

### Scheduling Assistant

```python
# Create scheduling agent
scheduler_agent = client.agents.create(
    name="Scheduling Assistant",
    system_prompt="""You manage scheduling and calendar.

Capabilities:
- Find available time slots
- Schedule meetings
- Send invitations
- Handle conflicts
- Respect preferences
""",
    tools=["calendar"]
)

# Schedule meeting
result = scheduler_agent.execute(
    goal="Schedule a 1-hour meeting with the engineering team next week",
    context={
        "attendees": ["alice@company.com", "bob@company.com"],
        "preferences": {"time": "morning", "days": ["tue", "wed", "thu"]},
        "meeting_type": "standup"
    }
)
```

---

## Sales & Marketing

### Lead Qualification

```python
# Create lead qualifier agent
lead_agent = client.agents.create(
    name="Lead Qualifier",
    system_prompt="""You qualify sales leads.

Score leads based on:
- Company size and industry
- Budget indicators
- Timeline urgency
- Decision-maker status

Output: score (1-100), qualification status, next steps.
""",
    tools=["web_search"]
)

# Qualify lead
result = lead_agent.execute(
    goal="Qualify this lead",
    context={
        "lead": lead_info,
        "ideal_customer_profile": icp,
        "scoring_criteria": scoring_rules
    }
)
```

### Proposal Generator

```python
# Create proposal agent
proposal_agent = client.agents.create(
    name="Proposal Generator",
    system_prompt="""You create sales proposals.

Include:
- Executive summary
- Solution overview
- Pricing and terms
- Timeline
- Case studies

Customize for the prospect's industry and needs.
"""
)

# Generate proposal
result = proposal_agent.execute(
    goal="Create a proposal for this prospect",
    context={
        "prospect": prospect_info,
        "requirements": requirements,
        "products": available_products,
        "pricing_tiers": pricing
    }
)
```

---

## Education & Training

### Course Content Creator

```python
# Create course agent
course_agent = client.agents.create(
    name="Course Creator",
    system_prompt="""You create educational content.

Structure:
- Learning objectives
- Lesson content
- Examples and exercises
- Quizzes
- Summary

Adapt to the target audience level.
"""
)

# Create course module
result = course_agent.execute(
    goal="Create a module on Python basics",
    context={
        "audience": "beginners",
        "duration": "2 hours",
        "format": "video script + exercises"
    }
)
```

### Quiz Generator

```python
# Create quiz agent
quiz_agent = client.agents.create(
    name="Quiz Generator",
    system_prompt="""You create educational quizzes.

Question types:
- Multiple choice
- True/false
- Fill in the blank
- Short answer

Include explanations for answers.
"""
)

# Generate quiz
result = quiz_agent.execute(
    goal="Create a quiz on machine learning basics",
    context={
        "topic": "supervised learning",
        "difficulty": "intermediate",
        "num_questions": 10
    }
)
```

---

## Implementation Tips

### Start Simple

```python
# Start with a focused agent
simple_agent = client.agents.create(
    name="Simple Helper",
    system_prompt="You help with one specific task.",
    tools=[]  # No tools initially
)

# Add complexity gradually
# After testing, add tools as needed
```

### Monitor and Iterate

```python
# Track performance
metrics = client.metrics.get(
    agent_id=agent.id,
    metrics=["success_rate", "avg_duration", "user_satisfaction"]
)

# Iterate based on feedback
if metrics.success_rate < 0.9:
    # Improve system prompt
    # Add more context
    # Adjust model
    pass
```

### Scale Responsibly

```python
# Use rate limiting
from asyncio import Semaphore

semaphore = Semaphore(10)  # Max 10 concurrent

async def rate_limited_execute(agent, goal):
    async with semaphore:
        return await agent.execute(goal=goal)
```

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
