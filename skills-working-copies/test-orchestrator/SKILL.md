---
name: test-orchestrator
description: "Set up testing infrastructure and strategy for a project. This skill should be used when a project is ready for testing setup, including test framework configuration, initial test scaffolding, and documentation of testing approach. Primarily a setup skill with guidance for ongoing testing."
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Bash
---

# test-orchestrator

<purpose>
Analyze an existing project and set up appropriate testing infrastructure. Creates test framework configuration, initial test scaffolding, and documents testing strategy. Provides educational guidance on testing best practices for the project's tech stack.
</purpose>

<role>
BUILDER role with CONSULTANT guidance. Sets up infrastructure and provides education.
- WILL configure testing frameworks
- WILL create initial test files and scaffolding
- WILL write test strategy documentation
- WILL provide guidance on writing tests
- Will NOT write comprehensive test suites (that's ongoing development)
</role>

<output>
- Test framework configuration files
- Initial test scaffolding (example tests)
- .docs/test-strategy.md (testing approach documentation)
- Test scripts in package.json or equivalent
- Educational guidance on testing patterns
</output>

---

<workflow>

<phase id="0" name="analyze-project">
<action>Scan project to understand tech stack and existing test setup.</action>

<project-analysis>
Look for and analyze:

1. Tech stack indicators:
   - package.json (Node.js/frontend framework)
   - requirements.txt / pyproject.toml (Python)
   - composer.json (PHP)
   - claude.md (project context)

2. Existing test configuration:
   - jest.config.js, vitest.config.ts
   - pytest.ini, pyproject.toml [tool.pytest]
   - phpunit.xml
   - tests/ or __tests__/ directories

3. Framework-specific patterns:
   - Next.js: app/, pages/, components/
   - FastAPI: app/, routers/, models/
   - PHP: src/, Controllers/, Models/

4. Existing tests:
   - Count and categorize existing test files
   - Identify testing patterns in use
</project-analysis>

<check-handoff-docs>
Read .docs/tech-stack-decision.md if it exists to understand:
- Chosen tech stack
- Testing frameworks mentioned
</check-handoff-docs>
</phase>

<phase id="1" name="determine-testing-approach">
<action>Recommend testing strategy based on project type and tech stack.</action>

<testing-layers>
| Layer | Purpose | When to Use |
|-------|---------|-------------|
| Unit Tests | Test individual functions/components in isolation | Always - foundation of testing |
| Integration Tests | Test multiple components working together | API routes, database operations |
| E2E Tests | Test complete user flows | Critical paths, happy paths |
</testing-layers>

<framework-recommendations>

<nextjs-testing>
**Recommended Stack:**
- Vitest or Jest for unit/integration tests
- React Testing Library for component tests
- Playwright or Cypress for E2E tests

**Configuration:**
- vitest.config.ts with jsdom environment
- @testing-library/react for component assertions
- MSW (Mock Service Worker) for API mocking
</nextjs-testing>

<fastapi-testing>
**Recommended Stack:**
- pytest for all test types
- pytest-asyncio for async tests
- httpx for API testing
- Factory Boy for test data

**Configuration:**
- pytest.ini or pyproject.toml [tool.pytest]
- conftest.py for fixtures
- TestClient for FastAPI endpoint testing
</fastapi-testing>

<php-testing>
**Recommended Stack:**
- PHPUnit for unit/integration tests
- Pest PHP (optional, more readable syntax)
- Laravel testing tools (if Laravel)

**Configuration:**
- phpunit.xml
- tests/Unit/, tests/Feature/ directories
</php-testing>

<node-express-testing>
**Recommended Stack:**
- Jest or Vitest for unit tests
- Supertest for API endpoint testing

**Configuration:**
- jest.config.js
- Test database configuration
</node-express-testing>

</framework-recommendations>

<prompt-to-user>
Based on your {tech-stack} project, I recommend:

**Testing Framework:** {framework}
**Test Types:**
- Unit tests for {specific areas}
- Integration tests for {specific areas}
- E2E tests for {specific areas} (optional)

**Coverage Goal:** Start with critical paths, aim for 70-80% over time

Does this approach work for you? Any specific areas you want to prioritize?
</prompt-to-user>
</phase>

<phase id="2" name="configure-framework">
<action>Set up testing framework configuration files.</action>

<configuration-files>

<vitest-config>
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    include: ['**/*.{test,spec}.{js,ts,jsx,tsx}'],
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```
</vitest-config>

<jest-config>
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
  ],
  testMatch: ['**/__tests__/**/*.[jt]s?(x)', '**/?(*.)+(spec|test).[jt]s?(x)'],
}
```
</jest-config>

<pytest-config>
```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_functions = ["test_*"]
asyncio_mode = "auto"
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["app"]
omit = ["tests/*", "alembic/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
]
```
</pytest-config>

<phpunit-config>
```xml
<!-- phpunit.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <coverage>
        <include>
            <directory suffix=".php">src</directory>
        </include>
    </coverage>
</phpunit>
```
</phpunit-config>

</configuration-files>

<package-scripts>
Add test scripts to package.json (Node.js projects):

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest --watch",
    "test:ui": "vitest --ui"
  }
}
```

Or for Python (in pyproject.toml):
```toml
[tool.poetry.scripts]
test = "pytest:main"
```
</package-scripts>
</phase>

<phase id="3" name="create-scaffolding">
<action>Create initial test files demonstrating testing patterns.</action>

<test-directory-structure>
```
tests/
├── setup.ts (or setup.js, conftest.py)
├── unit/
│   └── example.test.ts
├── integration/
│   └── api.test.ts
└── e2e/
    └── (optional) flows.test.ts
```
</test-directory-structure>

<example-unit-test-react>
```typescript
// tests/unit/example.test.tsx
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import { Button } from '@/components/ui/button'

describe('Button Component', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button')).toHaveTextContent('Click me')
  })

  it('handles click events', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click me</Button>)

    await userEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('can be disabled', () => {
    render(<Button disabled>Click me</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```
</example-unit-test-react>

<example-integration-test-api>
```typescript
// tests/integration/api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'

describe('API Endpoints', () => {
  beforeAll(async () => {
    // Setup: start test server, seed database
  })

  afterAll(async () => {
    // Cleanup: stop server, clear database
  })

  describe('GET /api/items', () => {
    it('returns list of items', async () => {
      const response = await fetch('http://localhost:3000/api/items')
      const data = await response.json()

      expect(response.status).toBe(200)
      expect(Array.isArray(data)).toBe(true)
    })

    it('returns 401 without authentication', async () => {
      const response = await fetch('http://localhost:3000/api/protected')
      expect(response.status).toBe(401)
    })
  })
})
```
</example-integration-test-api>

<example-pytest>
```python
# tests/test_api.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

class TestItems:
    async def test_get_items(self, client):
        response = await client.get("/api/items")
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    async def test_create_item(self, client):
        response = await client.post(
            "/api/items",
            json={"name": "Test Item", "description": "A test"}
        )
        assert response.status_code == 201
        assert response.json()["name"] == "Test Item"

    async def test_unauthorized_access(self, client):
        response = await client.get("/api/protected")
        assert response.status_code == 401
```
</example-pytest>

<setup-file>
```typescript
// tests/setup.ts
import '@testing-library/jest-dom'
import { cleanup } from '@testing-library/react'
import { afterEach } from 'vitest'

// Cleanup after each test
afterEach(() => {
  cleanup()
})

// Global mocks
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
  }),
  useSearchParams: () => new URLSearchParams(),
  usePathname: () => '/',
}))
```
</setup-file>
</phase>

<phase id="4" name="create-strategy-doc">
<action>Create .docs/test-strategy.md documenting the testing approach.</action>

<strategy-template>
```markdown
# Test Strategy

**Project:** {project_name}
**Created:** {date}
**Framework:** {test_framework}

## Testing Philosophy

This project follows a testing pyramid approach:
- Many unit tests (fast, isolated)
- Fewer integration tests (verify component interaction)
- Few E2E tests (critical user paths only)

## Test Types

### Unit Tests
**Location:** `tests/unit/`
**Purpose:** Test individual functions and components in isolation
**Run:** `npm test` or `pytest tests/unit`

**What to test:**
- Pure functions (utils, helpers)
- Component rendering
- State management logic
- Data transformations

### Integration Tests
**Location:** `tests/integration/`
**Purpose:** Test API endpoints and component interactions
**Run:** `npm test tests/integration` or `pytest tests/integration`

**What to test:**
- API endpoint responses
- Database operations
- Authentication flows
- Multi-component interactions

### E2E Tests (Optional)
**Location:** `tests/e2e/`
**Purpose:** Test complete user flows
**Run:** `npm run test:e2e`

**What to test:**
- Critical user journeys
- Happy path flows
- Authentication end-to-end

## Test Commands

| Command | Description |
|---------|-------------|
| `npm test` | Run all tests once |
| `npm run test:watch` | Run tests in watch mode |
| `npm run test:coverage` | Run tests with coverage report |
| `npm run test:ui` | Open visual test UI |

## Coverage Goals

**Current:** {current}%
**Target:** 70-80% for critical paths

Focus coverage on:
- Business logic
- API endpoints
- Authentication
- Data validation

## Writing New Tests

### Naming Convention
- Test files: `*.test.ts` or `*.spec.ts`
- Test descriptions: "should [expected behavior] when [condition]"

### Test Structure
```typescript
describe('ComponentName', () => {
  describe('method or behavior', () => {
    it('should do something when condition', () => {
      // Arrange
      // Act
      // Assert
    })
  })
})
```

### Best Practices
1. One assertion per test (when practical)
2. Test behavior, not implementation
3. Use descriptive test names
4. Keep tests independent
5. Mock external dependencies

## CI Integration

Tests run automatically on:
- Every push to `main` and `dev` branches
- Every pull request

See `.github/workflows/ci.yml` for configuration.

## Resources

- [Testing Library Docs](https://testing-library.com/)
- [Vitest Docs](https://vitest.dev/)
- [Pytest Docs](https://docs.pytest.org/)
```
</strategy-template>
</phase>

<phase id="5" name="provide-guidance">
<action>Summarize what was created and provide educational guidance.</action>

<summary-template>
## Test Infrastructure Setup Complete

**Project:** {project_name}
**Framework:** {test_framework}

---

### Files Created

- {config_file} - Test framework configuration
- tests/setup.{ext} - Test setup and global mocks
- tests/unit/example.test.{ext} - Example unit test
- tests/integration/api.test.{ext} - Example integration test
- .docs/test-strategy.md - Testing strategy documentation

---

### Test Commands

```bash
# Run all tests
{run_command}

# Run tests in watch mode
{watch_command}

# Run with coverage
{coverage_command}
```

---

### What's Next

Your testing infrastructure is set up. Here's how to proceed:

1. **Run the example tests** to verify setup:
   ```bash
   {run_command}
   ```

2. **Write tests as you build features:**
   - Write tests before or alongside new code
   - Focus on critical business logic first
   - Use example tests as templates

3. **Maintain test health:**
   - Keep tests passing
   - Review coverage periodically
   - Update tests when behavior changes

---

### Educational Notes

**Testing Pyramid:**
- Unit tests are fast and numerous - test small pieces in isolation
- Integration tests verify components work together
- E2E tests are slow but valuable for critical paths

**When to write tests:**
- Before implementing (TDD) - helps design better code
- After implementing - validates behavior
- When fixing bugs - prevent regression

**Test-Driven Development (TDD) cycle:**
1. Write a failing test
2. Write minimum code to pass
3. Refactor while keeping tests green

---

### Workflow Status

This is an **optional** phase in the Skills workflow.

**Previous:** Phase 3 (project-spinup) - Project foundation
**Current:** Phase 4 (test-orchestrator) - Test infrastructure
**Next:** Phase 5 (deploy-guide) - Deploy your application

You can continue building features and write tests as you go.
When ready to deploy, use the **deploy-guide** skill.
</summary-template>
</phase>

</workflow>

---

<guardrails>

<must-do>
- Analyze project structure before recommending framework
- Match testing tools to tech stack
- Create working example tests
- Document testing strategy in .docs/test-strategy.md
- Provide educational guidance on testing patterns
- Include commands for running tests
- Create setup files for test environment
</must-do>

<must-not-do>
- Write comprehensive test suites (that's ongoing development)
- Force a specific testing approach without consideration
- Skip the strategy documentation
- Create tests that won't run with current project setup
- Recommend E2E testing infrastructure for simple projects
</must-not-do>

</guardrails>

---

<workflow-status>
Phase 4 of 7: Test Strategy (Optional)

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Tech Stack (tech-stack-advisor)
  Phase 2: Deployment Strategy (deployment-advisor)
  Phase 3: Project Foundation (project-spinup) <- TERMINATION POINT (localhost)
  Phase 4: Test Strategy (you are here) - optional
  Phase 5: Deployment (deploy-guide) <- TERMINATION POINT (manual deploy)
  Phase 6: CI/CD (ci-cd-implement) <- TERMINATION POINT (full automation)
</workflow-status>

---

<integration-notes>

<workflow-position>
Phase 4 of 7 in the Skills workflow chain.
This is an OPTIONAL phase - can be skipped or invoked anytime.
Expected input: Project with code structure (after development begins)
Produces: Test infrastructure, .docs/test-strategy.md
</workflow-position>

<when-to-invoke>
- When project has initial code and is ready for testing infrastructure
- When user wants to add tests to an existing project
- Before deploy-guide if automated testing is desired
</when-to-invoke>

<flexible-entry>
This skill can be invoked standalone on any project. It analyzes the project structure to recommend appropriate testing tools.
</flexible-entry>

<status-utility>
Users can invoke the **workflow-status** skill at any time to:
- See current workflow progress
- Check which phases are complete
- Get guidance on next steps
- Review all handoff documents

Mention this option when users seem uncertain about their progress.
</status-utility>

</integration-notes>
