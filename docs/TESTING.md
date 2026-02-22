# Testing Guide

Complete guide to testing strategies, frameworks, and best practices for ResonantGenesis.

## Overview

ResonantGenesis uses a comprehensive testing strategy:
- **Unit Tests** - Test individual functions and classes
- **Integration Tests** - Test component interactions
- **End-to-End Tests** - Test complete user flows
- **API Tests** - Test REST endpoints
- **Contract Tests** - Test smart contracts

## Testing Stack

| Component | Framework |
|-----------|-----------|
| Backend | pytest |
| Frontend | Jest + React Testing Library |
| E2E | Playwright |
| API | pytest + httpx |
| Contracts | Foundry |

## Unit Testing

### Backend (Python)

#### Setup

```bash
pip install pytest pytest-asyncio pytest-cov
```

#### Writing Tests

```python
# tests/unit/test_agent_service.py
import pytest
from unittest.mock import Mock, patch
from app.services.agent_service import AgentService

class TestAgentService:
    @pytest.fixture
    def service(self):
        return AgentService(db=Mock(), cache=Mock())
    
    def test_create_agent(self, service):
        agent = service.create(
            name="Test Agent",
            system_prompt="You are helpful."
        )
        
        assert agent.name == "Test Agent"
        assert agent.status == "active"
    
    @pytest.mark.asyncio
    async def test_execute_agent(self, service):
        result = await service.execute(
            agent_id="agent_123",
            goal="Test goal"
        )
        
        assert result.status == "completed"
        assert result.output is not None
    
    def test_create_agent_invalid_name(self, service):
        with pytest.raises(ValueError) as exc:
            service.create(name="", system_prompt="Test")
        
        assert "Name is required" in str(exc.value)
```

#### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/unit/test_agent_service.py

# Run specific test
pytest tests/unit/test_agent_service.py::TestAgentService::test_create_agent

# Run with verbose output
pytest -v

# Run parallel
pytest -n auto
```

### Frontend (TypeScript)

#### Setup

```bash
npm install --save-dev jest @testing-library/react @testing-library/jest-dom
```

#### Writing Tests

```typescript
// src/components/AgentCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { AgentCard } from './AgentCard';

describe('AgentCard', () => {
  const mockAgent = {
    id: 'agent_123',
    name: 'Test Agent',
    status: 'active',
    description: 'A test agent'
  };

  it('renders agent name', () => {
    render(<AgentCard agent={mockAgent} />);
    expect(screen.getByText('Test Agent')).toBeInTheDocument();
  });

  it('shows status badge', () => {
    render(<AgentCard agent={mockAgent} />);
    expect(screen.getByText('active')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const onClick = jest.fn();
    render(<AgentCard agent={mockAgent} onClick={onClick} />);
    
    fireEvent.click(screen.getByRole('article'));
    expect(onClick).toHaveBeenCalledWith(mockAgent);
  });
});
```

#### Running Tests

```bash
# Run all tests
npm test

# Run with coverage
npm test -- --coverage

# Run in watch mode
npm test -- --watch

# Run specific file
npm test -- AgentCard.test.tsx
```

## Integration Testing

### Backend Integration Tests

```python
# tests/integration/test_agent_api.py
import pytest
from httpx import AsyncClient
from app.main import app
from app.database import get_test_db

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.fixture
async def auth_headers(client):
    response = await client.post("/api/v1/auth/login", json={
        "email": "test@example.com",
        "password": "testpassword"
    })
    token = response.json()["access_token"]
    return {"Authorization": f"Bearer {token}"}

class TestAgentAPI:
    @pytest.mark.asyncio
    async def test_create_agent(self, client, auth_headers):
        response = await client.post(
            "/api/v1/agents",
            json={
                "name": "Test Agent",
                "system_prompt": "You are helpful."
            },
            headers=auth_headers
        )
        
        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Test Agent"
        assert "id" in data
    
    @pytest.mark.asyncio
    async def test_list_agents(self, client, auth_headers):
        response = await client.get(
            "/api/v1/agents",
            headers=auth_headers
        )
        
        assert response.status_code == 200
        data = response.json()
        assert "agents" in data
        assert isinstance(data["agents"], list)
    
    @pytest.mark.asyncio
    async def test_unauthorized_access(self, client):
        response = await client.get("/api/v1/agents")
        assert response.status_code == 401
```

### Database Integration Tests

```python
# tests/integration/test_database.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from app.models import Agent, User

@pytest.fixture
async def db_session():
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async with AsyncSession(engine) as session:
        yield session

class TestAgentModel:
    @pytest.mark.asyncio
    async def test_create_agent(self, db_session):
        agent = Agent(
            name="Test Agent",
            system_prompt="You are helpful.",
            owner_id="user_123"
        )
        db_session.add(agent)
        await db_session.commit()
        
        assert agent.id is not None
        assert agent.created_at is not None
    
    @pytest.mark.asyncio
    async def test_agent_relationships(self, db_session):
        user = User(email="test@example.com")
        agent = Agent(name="Test", system_prompt="Test", owner=user)
        
        db_session.add_all([user, agent])
        await db_session.commit()
        
        assert agent.owner.email == "test@example.com"
```

## End-to-End Testing

### Playwright Setup

```bash
npm install --save-dev @playwright/test
npx playwright install
```

### Writing E2E Tests

```typescript
// tests/e2e/agent-creation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Agent Creation', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'testpassword');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
  });

  test('creates a new agent', async ({ page }) => {
    // Navigate to agent studio
    await page.click('text=Agent Studio');
    await page.click('text=Create Agent');
    
    // Fill form
    await page.fill('[name="name"]', 'E2E Test Agent');
    await page.fill('[name="description"]', 'Created by E2E test');
    await page.fill('[name="systemPrompt"]', 'You are a helpful assistant.');
    
    // Submit
    await page.click('button:has-text("Create")');
    
    // Verify
    await expect(page.locator('text=E2E Test Agent')).toBeVisible();
    await expect(page.locator('text=Agent created successfully')).toBeVisible();
  });

  test('validates required fields', async ({ page }) => {
    await page.click('text=Agent Studio');
    await page.click('text=Create Agent');
    
    // Submit without filling
    await page.click('button:has-text("Create")');
    
    // Check validation errors
    await expect(page.locator('text=Name is required')).toBeVisible();
    await expect(page.locator('text=System prompt is required')).toBeVisible();
  });
});
```

### Running E2E Tests

```bash
# Run all E2E tests
npx playwright test

# Run with UI
npx playwright test --ui

# Run specific test
npx playwright test agent-creation.spec.ts

# Run in headed mode
npx playwright test --headed

# Generate report
npx playwright show-report
```

## API Testing

### Using pytest

```python
# tests/api/test_agents_api.py
import pytest
from httpx import AsyncClient

BASE_URL = "https://api.resonantgenesis.xyz"

@pytest.fixture
def api_key():
    return "rg_test_sk_123456"

class TestAgentsAPI:
    @pytest.mark.asyncio
    async def test_list_agents(self, api_key):
        async with AsyncClient() as client:
            response = await client.get(
                f"{BASE_URL}/api/v1/agents",
                headers={"Authorization": f"Bearer {api_key}"}
            )
        
        assert response.status_code == 200
        assert "agents" in response.json()
    
    @pytest.mark.asyncio
    async def test_create_and_delete_agent(self, api_key):
        async with AsyncClient() as client:
            # Create
            create_response = await client.post(
                f"{BASE_URL}/api/v1/agents",
                headers={"Authorization": f"Bearer {api_key}"},
                json={
                    "name": "API Test Agent",
                    "system_prompt": "Test"
                }
            )
            assert create_response.status_code == 201
            agent_id = create_response.json()["id"]
            
            # Delete
            delete_response = await client.delete(
                f"{BASE_URL}/api/v1/agents/{agent_id}",
                headers={"Authorization": f"Bearer {api_key}"}
            )
            assert delete_response.status_code == 204
```

## Smart Contract Testing

### Foundry Setup

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### Writing Contract Tests

```solidity
// test/AgentRegistry.t.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "../src/AgentRegistry.sol";

contract AgentRegistryTest is Test {
    AgentRegistry public registry;
    address public owner;
    address public user;

    function setUp() public {
        owner = address(this);
        user = address(0x1);
        registry = new AgentRegistry();
    }

    function testRegisterAgent() public {
        bytes32 manifestHash = keccak256("test-manifest");
        string memory metadataUri = "ipfs://test";
        
        vm.prank(user);
        registry.registerAgent(manifestHash, metadataUri);
        
        AgentRegistry.Agent memory agent = registry.getAgent(manifestHash);
        assertEq(agent.owner, user);
        assertEq(agent.metadataUri, metadataUri);
    }

    function testCannotRegisterDuplicate() public {
        bytes32 manifestHash = keccak256("test-manifest");
        
        registry.registerAgent(manifestHash, "ipfs://test");
        
        vm.expectRevert(AgentRegistry.AlreadyRegistered.selector);
        registry.registerAgent(manifestHash, "ipfs://test2");
    }

    function testOnlyOwnerCanUpdate() public {
        bytes32 manifestHash = keccak256("test-manifest");
        registry.registerAgent(manifestHash, "ipfs://test");
        
        vm.prank(user);
        vm.expectRevert(AgentRegistry.NotOwner.selector);
        registry.updateMetadata(manifestHash, "ipfs://new");
    }
}
```

### Running Contract Tests

```bash
# Run all tests
forge test

# Run with verbosity
forge test -vvv

# Run specific test
forge test --match-test testRegisterAgent

# Run with gas report
forge test --gas-report

# Run with coverage
forge coverage
```

## Test Coverage

### Backend Coverage

```bash
# Generate coverage report
pytest --cov=app --cov-report=html --cov-report=term

# Coverage thresholds
pytest --cov=app --cov-fail-under=80
```

### Frontend Coverage

```bash
# Generate coverage
npm test -- --coverage --coverageReporters=html

# Coverage thresholds in jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### Contract Coverage

```bash
forge coverage --report lcov
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest --cov=app --cov-report=xml
      - uses: codecov/codecov-action@v3

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test -- --coverage

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test

  contract-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: foundry-rs/foundry-toolchain@v1
      - run: forge test
```

## Best Practices

### Test Organization

```
tests/
├── unit/
│   ├── services/
│   ├── models/
│   └── utils/
├── integration/
│   ├── api/
│   └── database/
├── e2e/
│   ├── auth/
│   └── agents/
├── fixtures/
│   └── data.json
└── conftest.py
```

### Test Naming

```python
# Good: Descriptive names
def test_create_agent_with_valid_data_returns_agent():
    pass

def test_create_agent_without_name_raises_validation_error():
    pass

# Bad: Vague names
def test_create():
    pass

def test_error():
    pass
```

### Test Isolation

```python
@pytest.fixture(autouse=True)
async def clean_database(db_session):
    yield
    # Cleanup after each test
    await db_session.rollback()
```

### Mocking External Services

```python
from unittest.mock import patch, AsyncMock

@patch('app.services.ai_service.OpenAI')
async def test_agent_execution(mock_openai):
    mock_openai.return_value.chat.completions.create = AsyncMock(
        return_value=Mock(choices=[Mock(message=Mock(content="Test response"))])
    )
    
    result = await agent_service.execute(agent_id, "Test goal")
    assert result.output == "Test response"
```

## Troubleshooting

### Tests Hanging

1. Check for missing `await`
2. Look for infinite loops
3. Add timeouts to async operations

### Flaky Tests

1. Avoid time-dependent assertions
2. Use explicit waits in E2E tests
3. Isolate test data

### Coverage Gaps

1. Review uncovered branches
2. Add edge case tests
3. Test error paths

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
