# Contributing to ResonantGenesis

Thank you for your interest in contributing to ResonantGenesis! We welcome contributions from the community.

## 🌟 Ways to Contribute

- **Report Bugs**: Submit detailed bug reports with reproduction steps
- **Suggest Features**: Propose new features or improvements
- **Write Code**: Submit pull requests for bug fixes or new features
- **Improve Docs**: Help us improve documentation
- **Design**: Contribute UI/UX improvements
- **Translate**: Help translate the platform to other languages

## 🚀 Getting Started

### Prerequisites

- **Node.js**: v18.x or higher
- **npm**: v9.x or higher
- **Python**: 3.10+ (for backend development)
- **Git**: Latest version
- **Docker**: Optional, for running full stack locally

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ResonantGenesis.git
   cd ResonantGenesis
   ```
3. Add upstream remote:
   ```bash
   git remote add upstream https://github.com/louienemesh/ResonantGenesis.git
   ```

### Frontend Development Setup

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local

# Configure environment variables
# Edit .env.local with your settings:
# - VITE_API_URL=http://localhost:8000/api/v1
# - VITE_NODE_URL=http://localhost:8081

# Start development server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Backend Development Setup

```bash
# Navigate to backend directory
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Copy environment template
cp .env.example .env

# Configure environment variables
# Edit .env with your settings:
# - DATABASE_URL=postgresql://user:pass@localhost:5432/resonant
# - REDIS_URL=redis://localhost:6379
# - JWT_SECRET=your-secret-key

# Run database migrations
alembic upgrade head

# Start development server
uvicorn main:app --reload --port 8000

# Run tests
pytest
```

### Smart Contract Development

```bash
# Navigate to contracts directory
cd contracts

# Install dependencies
npm install

# Compile contracts
npx hardhat compile

# Run tests
npx hardhat test

# Deploy to testnet (Base Sepolia)
npx hardhat run scripts/deploy.js --network base-sepolia
```

## 🔀 Pull Request Process

### Before Submitting

1. **Sync with upstream**:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/bug-description
   ```

3. **Make your changes** following our code style guidelines

4. **Test thoroughly**:
   - Run all existing tests
   - Add tests for new functionality
   - Test manually in the browser

5. **Commit with conventional commits**:
   ```bash
   git commit -m "feat: add agent collaboration feature"
   ```

### Submitting Your PR

1. Push your branch:
   ```bash
   git push origin feature/your-feature-name
   ```

2. Open a Pull Request on GitHub

3. Fill out the PR template:
   - **Description**: What does this PR do?
   - **Type**: Feature / Bug Fix / Docs / Refactor
   - **Testing**: How was this tested?
   - **Screenshots**: For UI changes
   - **Breaking Changes**: Any breaking changes?

### Review Process

1. **Automated checks** must pass (CI/CD, linting, tests)
2. **Code review** by at least one maintainer
3. **Address feedback** with additional commits
4. **Squash and merge** once approved

### PR Review Checklist

Reviewers will check:
- [ ] Code follows project style guidelines
- [ ] Tests are included and passing
- [ ] Documentation is updated
- [ ] No security vulnerabilities introduced
- [ ] Performance impact is acceptable
- [ ] Backwards compatibility maintained

## 📝 Commit Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

| Type | Description |
|------|-------------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation changes |
| `style:` | Code style changes (formatting, etc.) |
| `refactor:` | Code refactoring |
| `perf:` | Performance improvements |
| `test:` | Adding or updating tests |
| `build:` | Build system changes |
| `ci:` | CI/CD changes |
| `chore:` | Maintenance tasks |

### Examples

```bash
feat: add agent team collaboration API
fix: resolve session timeout issue on mobile
docs: update API reference with pagination examples
refactor: extract agent validation into separate module
test: add integration tests for webhook triggers
```

## 🧪 Testing Guidelines

### Frontend Testing

```bash
# Unit tests
npm test

# E2E tests with Playwright
npm run test:e2e

# Coverage report
npm run test:coverage
```

### Backend Testing

```bash
# Unit tests
pytest tests/unit

# Integration tests
pytest tests/integration

# Full test suite with coverage
pytest --cov=app --cov-report=html
```

### Test Requirements

- **New features**: Must include unit tests
- **Bug fixes**: Must include regression test
- **API changes**: Must include integration tests
- **Coverage**: Aim for >80% coverage on new code

## 📖 Code Style

### TypeScript/React (Frontend)

```typescript
// Use functional components with hooks
const AgentCard: React.FC<AgentCardProps> = ({ agent, onSelect }) => {
  const [isLoading, setIsLoading] = useState(false);
  
  // Use descriptive variable names
  const handleAgentSelection = useCallback(() => {
    setIsLoading(true);
    onSelect(agent.id);
  }, [agent.id, onSelect]);

  return (
    <Card onClick={handleAgentSelection}>
      <CardTitle>{agent.name}</CardTitle>
    </Card>
  );
};
```

### Python (Backend)

```python
from typing import Optional, List
from pydantic import BaseModel

class AgentCreate(BaseModel):
    """Schema for creating a new agent."""
    name: str
    description: Optional[str] = None
    tools: List[str] = []

async def create_agent(
    agent_data: AgentCreate,
    user_id: str,
    db: AsyncSession
) -> Agent:
    """
    Create a new agent for the specified user.
    
    Args:
        agent_data: Agent creation parameters
        user_id: ID of the owning user
        db: Database session
        
    Returns:
        The created Agent instance
    """
    agent = Agent(**agent_data.dict(), owner_id=user_id)
    db.add(agent)
    await db.commit()
    return agent
```

### Solidity (Smart Contracts)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/// @title AgentRegistry
/// @notice Manages on-chain agent registration
contract AgentRegistry {
    /// @notice Emitted when a new agent is registered
    event AgentRegistered(
        bytes32 indexed agentId,
        address indexed owner,
        string metadataUri
    );
    
    /// @notice Register a new agent
    /// @param agentId Unique identifier for the agent
    /// @param metadataUri IPFS URI for agent metadata
    function registerAgent(
        bytes32 agentId,
        string calldata metadataUri
    ) external {
        // Implementation
    }
}
```

## 🏗️ Architecture Guidelines

### Frontend Architecture

```
src/
├── components/     # Reusable UI components
│   ├── common/     # Generic components (Button, Card, etc.)
│   └── features/   # Feature-specific components
├── pages/          # Route pages
├── hooks/          # Custom React hooks
├── services/       # API client services
├── stores/         # State management (Zustand)
├── types/          # TypeScript type definitions
└── utils/          # Utility functions
```

### Backend Architecture

```
app/
├── api/            # API route handlers
│   └── v1/         # Version 1 endpoints
├── core/           # Core configuration
├── models/         # Database models
├── schemas/        # Pydantic schemas
├── services/       # Business logic
└── utils/          # Utility functions
```

### Key Principles

1. **Separation of Concerns**: Keep UI, business logic, and data access separate
2. **Single Responsibility**: Each module/function should do one thing well
3. **DRY**: Don't repeat yourself - extract common logic
4. **SOLID**: Follow SOLID principles for maintainable code
5. **Security First**: Always validate input, sanitize output, use parameterized queries

## 🔒 Security Guidelines

- **Never commit secrets** (API keys, passwords, private keys)
- **Use environment variables** for configuration
- **Validate all user input** on both client and server
- **Use parameterized queries** to prevent SQL injection
- **Implement rate limiting** on API endpoints
- **Follow OWASP guidelines** for web security

## 🤝 Code of Conduct

Be respectful, inclusive, and professional. We're building the future together!

- Be welcoming to newcomers
- Respect differing viewpoints
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

## 📧 Getting Help

- **Discord**: [Join our community](https://discord.gg/resonantgenesis)
- **GitHub Discussions**: For questions and ideas
- **GitHub Issues**: For bug reports and feature requests
- **Email**: contributors@resonantgenesis.xyz

## 🎉 Recognition

Contributors are recognized in:
- Our README contributors section
- Release notes for significant contributions
- Special Discord roles for active contributors

Thank you for contributing to ResonantGenesis! 🚀
