# Claude Coding Agent v3.1 - Complete Modular System

## Core Instructions (Universal)

### Initialize Session
```bash
# 1. Detect environment and load modules
bash .claude/detect-environment.sh 2>/dev/null || echo "No auto-detection available"

# 2. Status check (ALWAYS run if returning to project)
pwd && git status -s && git log --oneline -3

# 3. Load project config
cat .claude-config 2>/dev/null || echo "No project config found"
```

### Workflow Pipeline
**READ** (current state) → **THINK** (approach) → **CODE** (implement) → **TEST** (verify) → **SHIP** (document & commit)

### Iron Rules
1. **NO** secrets, credentials, or API keys in code
2. **READ** files with desktop commander before editing (ALWAYS)
3. **STUDY** 3+ existing files before writing new code
4. **TEST** before committing any changes
5. **ASK** before architectural changes
6. **MATCH** existing patterns and conventions
7. **DOCUMENT** why, not what
8. **CONSIDER** "Should this be built at all?"

### Before Coding
- What problem are we solving?
- What's the simplest solution?
- What could break?
- Who else is affected?

### Get Full Context
- **Technical Stack**: See `.claude/modules/[auto-detected].md`
- **Project Specific**: See `.claude-config`
- **Business Logic**: Check `docs/` or ask for context
- **Architecture**: Look for `docs/adr/` or `ARCHITECTURE.md`

### Context Switching Protocol
When returning to a project:
1. Re-run initialization
2. Check what changed: `git fetch && git log HEAD..origin/main --oneline`
3. Review open PRs: `gh pr list`
4. Ask: "What's the current priority?"

---

## Environment Detection Script v3.1

Save as `.claude/detect-environment.sh`:

```bash
#!/bin/bash
# Claude Environment Detection v3.1

echo "🔍 Detecting project environment..."

# Initialize
MODULES_DIR="$(dirname "$0")/modules"
LOADED_MODULES=""

# Check available MCP tools
echo "🔧 Checking MCP tools..."
[ -n "$CLAUDE_DESKTOP" ] && {
    echo "✓ Desktop Commander available"
    echo "✓ GitHub MCP available"
    echo "✓ Sequential Thinking available"
    echo "✓ Playwright MCP available"
    LOADED_MODULES="$LOADED_MODULES tools-and-practices"
}

# Core detection
[ -f package.json ] && {
    echo "📦 Node.js project detected"
    LOADED_MODULES="$LOADED_MODULES node"
}

[ -f requirements.txt ] || [ -f Pipfile ] || [ -f pyproject.toml ] && {
    echo "🐍 Python project detected"
    LOADED_MODULES="$LOADED_MODULES python"
}

[ -f go.mod ] && {
    echo "🐹 Go project detected"
    LOADED_MODULES="$LOADED_MODULES golang"
}

# Framework detection
[ -f package.json ] && grep -q "react" package.json && {
    echo "⚛️  React detected"
    LOADED_MODULES="$LOADED_MODULES react"
}

[ -d .github/workflows ] || [ -f .gitlab-ci.yml ] || [ -f Jenkinsfile ] && {
    echo "🔄 CI/CD pipeline detected"
    LOADED_MODULES="$LOADED_MODULES ci-cd"
}

# Infrastructure detection
[ -f docker-compose.yml ] || [ -f Dockerfile ] && {
    echo "🐳 Docker detected"
    LOADED_MODULES="$LOADED_MODULES docker"
}

[ -d migrations ] || [ -d alembic ] || [ -f schema.sql ] && {
    echo "🗄️  Database detected"
    LOADED_MODULES="$LOADED_MODULES database"
}

[ -d terraform ] || [ -d cloudformation ] || [ -f serverless.yml ] && {
    echo "☁️  Cloud infrastructure detected"
    LOADED_MODULES="$LOADED_MODULES cloud"
}

# Special detection
find . -name "*.test.*" -o -name "*_test.go" -o -name "test_*.py" | head -1 > /dev/null && {
    echo "🧪 Test suite detected"
    LOADED_MODULES="$LOADED_MODULES testing quality-practices"
}

[ -f .env.example ] || [ -f .env.sample ] && {
    echo "🔐 Environment configuration detected"
    LOADED_MODULES="$LOADED_MODULES env-config"
}

# Dependency files detection
[ -f package-lock.json ] || [ -f yarn.lock ] || [ -f Pipfile.lock ] && {
    echo "📦 Dependency management detected"
    LOADED_MODULES="$LOADED_MODULES dependency-management"
}

# Load detected modules
echo ""
echo "📚 Loading modules: $LOADED_MODULES"
echo "================================"

for module in $LOADED_MODULES; do
    if [ -f "$MODULES_DIR/$module.md" ]; then
        echo ""
        echo "### Module: $module"
        cat "$MODULES_DIR/$module.md"
        echo "--------------------------------"
    fi
done

# Project-specific overrides
if [ -f .claude-modules ]; then
    echo ""
    echo "📎 Loading project-specific modules from .claude-modules"
    cat .claude-modules
fi

echo ""
echo "✅ Environment detection complete!"
echo "💡 Tip: Create .claude-config for project-specific settings"
```

---

## Module: Tools and Practices (`tools-and-practices.md`)

### MCP Tool Usage Guide

#### Desktop Commander (CRITICAL)
**ALWAYS use before editing any file:**
```bash
# Read before modifying
cat filename.js  # or use appropriate viewer

# Explore project structure
find . -type f -name "*.js" | grep -v node_modules

# Check file permissions
ls -la filename
```

#### Sequential Thinking MCP
**Use for:**
- Breaking down features requiring >2 hours
- Analyzing complex bugs systematically
- Planning architectural changes
- Evaluating multiple solution paths
- Understanding intricate business logic

**Example usage:**
"Use sequential thinking to analyze the authentication flow and identify potential race conditions"

#### Playwright MCP
**Use for:**
- E2E testing of web interfaces
- Visual regression testing
- Automating repetitive browser tasks
- Debugging UI-specific issues
- Testing complex user interactions

**Example:**
```javascript
// Use playwright to test the checkout flow
await page.goto('/checkout');
await page.fill('#email', 'test@example.com');
await page.click('button[type="submit"]');
```

#### GitHub MCP
**Use for:**
- Checking and updating issues
- Creating PRs with templates
- Reviewing PR status
- Linking commits to issues
- Updating project boards

**Workflow:**
```bash
# Check assigned issues
gh issue list --assignee @me

# Create PR with template
gh pr create --template .github/pull_request_template.md

# Link commit to issue
git commit -m "fix: resolve race condition, fixes #123"
```

### Git Commit Message Standards

```
type(scope): subject (50 chars max)

Longer description explaining WHY this change was necessary,
not WHAT changed (the code shows that).

BREAKING CHANGE: Description of breaking changes
Fixes: #123
See also: #456, #789
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, missing semicolons, etc
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvement
- `test`: Adding missing tests
- `chore`: Changes to build process or auxiliary tools

---

## Module: Quality Practices (`quality-practices.md`)

### Testing Standards

#### Coverage Requirements
```bash
# Check coverage
npm run coverage || jest --coverage
pytest --cov --cov-report=term-missing

# Minimum thresholds
# - Statements: 80%
# - Branches: 75%
# - Functions: 80%
# - Lines: 80%
```

#### Test Writing Principles
1. **Arrange, Act, Assert** pattern
2. **One assertion per test** (when practical)
3. **Descriptive test names** that explain the scenario
4. **Test behavior, not implementation**
5. **Include edge cases and error paths**

### Performance Tracking

#### Key Metrics to Monitor
```bash
# Build performance
time npm run build
echo "Build size:" && du -sh dist/

# Bundle analysis
npm run analyze 2>/dev/null || npx webpack-bundle-analyzer stats.json

# Runtime performance
# Add performance marks in code:
performance.mark('feature-start');
// ... code ...
performance.mark('feature-end');
performance.measure('feature', 'feature-start', 'feature-end');
```

#### Performance Checklist
- [ ] Lighthouse score >90 for web apps
- [ ] Build time <60 seconds
- [ ] Bundle size increase <5% per feature
- [ ] API response time <200ms (p95)
- [ ] Memory usage stable over time

---

## Module: Dependency Management (`dependency-management.md`)

### Before Adding Dependencies

#### The Checklist
1. **Need Assessment**
   ```bash
   # Can native solutions work?
   # Check: https://youmightnotneed.com/
   ```

2. **Size Impact**
   ```bash
   # Check package size
   npm view [package-name] size
   npx bundle-phobia [package-name]
   ```

3. **Security Review**
   ```bash
   # Security audit
   npm audit
   npx snyk test
   
   # Check package reputation
   npx npm-check-updates
   ```

4. **License Compatibility**
   ```bash
   npx license-checker --summary
   ```

5. **Maintenance Status**
   - Last publish date <1 year
   - Open issues reasonable
   - Active maintainers
   - Good documentation

### Dependency Hygiene

#### Regular Maintenance
```bash
# Weekly checks
npm outdated                    # See what needs updating
npm audit                       # Security vulnerabilities
npx depcheck                    # Find unused dependencies

# Monthly cleanup
npm prune                       # Remove extraneous packages
npm dedupe                      # Deduplicate dependencies
```

#### Update Strategy
```bash
# Safe updates (patch versions)
npm update

# Minor updates (review changelog)
npx npm-check-updates -u --target minor
npm install

# Major updates (careful testing)
npx npm-check-updates -u
# Review breaking changes first!
```

### Red Flags
- No updates in 2+ years
- Single maintainer for critical package
- No tests in repository
- Dramatically increasing bundle size
- Many open security issues

---

## Module: Database Safety (`database.md`)

### Database Operations Checklist

#### Before ANY Schema Change
```bash
# 1. Backup current state
pg_dump dbname > backup_$(date +%Y%m%d_%H%M%S).sql
mysqldump dbname > backup_$(date +%Y%m%d_%H%M%S).sql

# 2. Test migration
psql test_db < migration.sql
```

#### Migration Safety Rules
1. **Always include rollback**
   ```sql
   -- up.sql
   ALTER TABLE users ADD COLUMN age INTEGER;
   
   -- down.sql
   ALTER TABLE users DROP COLUMN age;
   ```

2. **Avoid locking operations**
   ```sql
   -- Bad: Locks table
   ALTER TABLE users ADD COLUMN status VARCHAR(50) NOT NULL DEFAULT 'active';
   
   -- Good: Non-blocking
   ALTER TABLE users ADD COLUMN status VARCHAR(50);
   UPDATE users SET status = 'active' WHERE status IS NULL;
   ALTER TABLE users ALTER COLUMN status SET NOT NULL;
   ```

3. **Index creation in production**
   ```sql
   CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
   ```

#### Query Safety
```javascript
// NEVER do this
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ALWAYS do this
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);
```

#### Data Privacy Checklist
- [ ] PII is encrypted at rest
- [ ] Sensitive data excluded from logs
- [ ] Row-level security implemented
- [ ] Audit trail for data access
- [ ] Anonymization for non-production

### Transaction Principles
```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('INSERT INTO orders ...');
  await client.query('UPDATE inventory ...');
  await client.query('COMMIT');
} catch (e) {
  await client.query('ROLLBACK');
  throw e;
} finally {
  client.release();
}
```

---

## Module: Production Awareness (`production.md`)

### Think "3 AM Failures"
Before deploying, ask:
- What happens if this fails at 3 AM?
- Can it be rolled back without data loss?
- Will alerts fire for the right people?
- Is there a manual override?

### Production Checklist

#### Feature Flags
```javascript
if (featureFlags.isEnabled('new-checkout-flow', userId)) {
  return newCheckoutFlow();
} else {
  return legacyCheckoutFlow();
}
```

#### Health Checks
```javascript
app.get('/health', (req, res) => {
  const health = {
    uptime: process.uptime(),
    timestamp: Date.now(),
    status: 'OK',
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      external_api: await checkExternalAPI()
    }
  };
  res.status(health.status === 'OK' ? 200 : 503).json(health);
});
```

#### Structured Logging
```javascript
logger.info('Order processed', {
  orderId: order.id,
  userId: user.id,
  amount: order.total,
  duration: Date.now() - startTime,
  correlationId: req.headers['x-correlation-id'],
  // Enables tracing issues across services
});
```

#### Circuit Breakers
```javascript
const circuitBreaker = new CircuitBreaker(externalAPICall, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});

circuitBreaker.fallback(() => {
  return { status: 'degraded', cached: true, data: getCachedData() };
});
```

### Gradual Rollout Strategy
1. **Internal testing** (employees only)
2. **Beta users** (1% of traffic)
3. **Gradual increase** (5% → 10% → 25% → 50% → 100%)
4. **Monitor each stage** for 24 hours
5. **Automatic rollback** on error spike

---

## Module: Legacy Code (`legacy-code.md`)

### The Prime Directive
"Assume the previous developer did their best with the knowledge and constraints they had at the time."

### When to Refactor vs Preserve

#### Refactor When:
- Adding new features to the area
- Fixing bugs repeatedly (3+ times)
- Performance is measurably impacted
- Security vulnerabilities exist
- Test coverage can be added first

#### Preserve When:
- It works and isn't changing
- Refactoring risk > benefit
- No tests exist (add tests first!)
- Near deadline or freeze period
- Business logic is unclear

### Safe Legacy Code Changes

#### 1. Characterization Tests First
```javascript
// Before changing legacy code, document current behavior
describe('Legacy discount calculator', () => {
  it('should maintain existing behavior for standard discounts', () => {
    // Test current behavior, even if it seems wrong
    expect(legacyCalculateDiscount(100, 'SAVE10')).toBe(89.99);
    // Note: Returns 89.99, not 90.00 due to legacy rounding
  });
});
```

#### 2. Strangler Fig Pattern
```javascript
// New code alongside old
function processOrder(order) {
  if (featureFlags.useNewOrderSystem) {
    return newOrderProcessor.process(order);
  } else {
    return legacyOrderSystem.processOrder(order);
  }
}
```

#### 3. Minimal Changes
```javascript
// Instead of rewriting the entire function
function legacyCalculateShipping(order) {
  // 500 lines of complex logic...
  
  // Add ONLY what's needed
  if (order.expressShipping) {
    return existingTotal + EXPRESS_SHIPPING_COST;
  }
  
  // Rest remains untouched
}
```

### Documentation for Legacy Code
```javascript
/**
 * LEGACY: Pre-2019 pricing engine
 * 
 * DO NOT MODIFY without approval from:
 * - Finance team (pricing rules)
 * - Legal team (compliance)
 * 
 * Known issues:
 * - Rounding errors for currencies with >2 decimals (#1234)
 * - Doesn't handle negative quantities (#5678)
 * - Performance degrades with >100 items (#9012)
 * 
 * Replacement planned: Q3 2024 (PROJECT-789)
 * See: docs/legacy/pricing-v1-behavior.md
 */
```

---

## Module: Debugging Production (`debugging.md`)

### Safe Production Debugging

#### Information Gathering (Always Safe)
```bash
# Application logs
kubectl logs -f deployment/app --tail=100
journalctl -u app-service -f

# Metrics without modifying
curl -s http://localhost:9090/metrics | grep -i error

# Recent changes
kubectl rollout history deployment/app
git log --oneline --since="2 days ago" --grep="deploy"
```

#### Active Debugging (Be Careful!)

##### 1. Temporary Debug Logging
```javascript
// ALWAYS add expiration
if (process.env.DEBUG_ISSUE_1234 === 'true' && Date.now() < 1706745600000) {
  logger.debug('DEBUG-1234: Payment flow state', {
    step: 'validation',
    payload: sanitizeForLogging(request),
    timestamp: new Date().toISOString(),
    expires: '2024-02-01T00:00:00Z'
  });
}
```

##### 2. Read-Only Database Queries
```sql
-- Use read replica
\c readonly_replica

-- Add query timeout
SET statement_timeout = '5s';

-- Explain plan first
EXPLAIN (ANALYZE, BUFFERS) 
SELECT ... FROM ... WHERE ...;
```

##### 3. Shadow Traffic Testing
```javascript
// Test fixes without affecting real users
if (req.headers['x-shadow-traffic'] === 'true') {
  return experimentalHandler(req, res);
}
```

### Debug Information Checklist
- [ ] Full stack traces with line numbers
- [ ] Request/correlation IDs
- [ ] User context (anonymized)
- [ ] System state (memory, CPU, connections)
- [ ] Recent deployments and config changes
- [ ] Database query performance
- [ ] External service latencies
- [ ] Error rates before/after issue started

### Production Debug Utilities
```javascript
// Temporary debug endpoint (REMOVE AFTER DEBUGGING)
if (process.env.ENABLE_DEBUG_ENDPOINT === 'true') {
  app.get('/_debug/health-detailed', authenticate, rateLimit, (req, res) => {
    res.json({
      warning: 'Debug endpoint - remove by 2024-02-01',
      memory: process.memoryUsage(),
      uptime: process.uptime(),
      connections: getActiveConnections(),
      cache_stats: getCacheStats(),
      // Never expose sensitive data
    });
  });
}
```

---

## Module: Claude Meta (`claude-meta.md`)

### Working with Claude's Constraints

#### Context Management
```bash
# Save detailed context between sessions
cat > .claude-session << EOF
Task: Implementing payment processing
Branch: feature/payment-integration
Progress: 
  ✓ Database schema for payments
  ✓ Stripe webhook endpoints
  ✓ Basic payment flow
  ⏳ Testing refund process
  ⏳ Adding payment analytics
Next Steps:
  1. Complete refund flow tests
  2. Add idempotency keys
  3. Implement payment retry logic
Blockers:
  - Need Stripe webhook secret for staging
  - Clarify refund policy with product team
Key Decisions:
  - Using Stripe Payment Intents (not Charges)
  - Storing amounts in cents (integer)
  - UTC timestamps only
EOF
```

#### When to Ask for Human Decision
1. **Multiple valid approaches** with different tradeoffs
2. **Business logic** ambiguity
3. **Security implications** beyond standard practices
4. **Significant cost** implications
5. **Breaking changes** to public APIs
6. **Data privacy** concerns
7. **Legal/compliance** questions

#### Effective Claude Interactions

##### DO:
- Provide complete error messages
- Share relevant file contents
- Specify constraints clearly
- Give examples of desired outcome
- Mention tech stack versions

##### DON'T:
- Assume Claude remembers previous conversations
- Use ambiguous pronouns without context
- Forget to mention external dependencies
- Skip error messages as "it doesn't work"

#### Breaking Down Large Tasks
```markdown
## Large Task: Add Multi-tenant Support

### Split into PRs:
1. **PR 1**: Database schema changes
   - Add tenant_id to all tables
   - Update indexes
   - Migration scripts

2. **PR 2**: Data isolation layer
   - Repository pattern updates
   - Tenant context middleware
   - Query builders

3. **PR 3**: API updates
   - Tenant resolution
   - Permission checks
   - API documentation

4. **PR 4**: UI changes
   - Tenant switcher
   - Scoped navigation
   - Tenant admin panel

Each PR: <500 lines, single responsibility, tested
```

---

## Project Configuration Template (`.claude-config`)

```yaml
# Claude Coding Agent Configuration v3.1
project_name: "My Awesome App"
version: "2.1.0"
primary_language: "javascript"
framework: "express"
database: "postgresql"

# Priorities (affects decision making)
priority_order:
  1: "security"
  2: "reliability"  
  3: "performance"
  4: "features"

# Testing requirements
testing:
  min_coverage: 80
  required_for_pr: true
  frameworks: ["jest", "playwright"]
  
# Performance budgets
performance:
  build_time_max: "60s"
  bundle_size_max: "500KB"
  lighthouse_min: 90

# Project-specific rules
rules:
  - "Use async/await, not callbacks"
  - "All monetary values in cents (integer)"
  - "UTC timestamps only, display converts to user timezone"
  - "Feature flags for all new features"
  - "No direct database access from controllers"

# External integrations
integrations:
  - name: "Stripe"
    docs: "docs/stripe-integration.md"
    test_mode: "Use test keys in .env.test"
  - name: "SendGrid"  
    docs: "docs/email-setup.md"
    templates: "src/email-templates/"

# Team contacts
team:
  lead: "@tech-lead"
  security: "@security-team"
  devops: "@platform-team"
  product: "@product-manager"

# Deployment
deployment:
  environments:
    staging: 
      url: "https://staging.example.com"
      branch: "develop"
      auto_deploy: true
    production:
      url: "https://example.com"
      branch: "main"
      approval_required: true
  rollback_procedure: "See docs/emergency-procedures.md"

# Current sprint
current_sprint:
  number: 42
  goal: "Launch payment processing"
  ends: "2024-02-01"
  tasks:
    - "Complete Stripe integration"
    - "Add payment analytics"
    - "Load test payment flow"
  blockers:
    - "Waiting for Stripe webhook configuration"
    - "Need clarification on refund policy"

# Module configuration
modules:
  skip: ["legacy-code"]  # Not needed for new project
  additional: ["performance-critical", "pci-compliance"]

# Development workflow
workflow:
  branch_pattern: "(feature|fix|chore)/[ticket-number]-description"
  pr_size_limit: 500  # lines
  require_issue_link: true
  protected_branches: ["main", "develop"]
  merge_strategy: "squash"  # or "merge" or "rebase"

# Custom scripts
scripts:
  setup: "npm install && cp .env.example .env && npm run db:migrate"
  test: "npm run lint && npm test && npm run test:e2e"
  clean: "rm -rf node_modules dist coverage .cache"

# Architecture decisions
architecture:
  style: "Domain-driven design"
  patterns: ["Repository", "Service layer", "DTO"]
  state_management: "Redux Toolkit"
  api_style: "RESTful with OpenAPI spec"
```

---

## Quick Command Reference

```bash
# === Session Management ===
.claude/detect-environment.sh          # Start new session
cat .claude-session                    # Check previous context
echo "Status: ..." >> .claude-session  # Update progress

# === Quick Status ===
git status -s && echo "---" && git log --oneline -5
git fetch && git diff --stat HEAD origin/main

# === Code Quality ===
npm test -- --coverage                 # Test with coverage
npm run lint -- --fix                  # Auto-fix linting
npm audit fix                          # Fix vulnerabilities

# === Security Checks ===
# Pre-commit scan
git diff --staged | grep -E "(password|key|secret|token)" || echo "✓ No secrets"

# Full security audit  
npm audit
grep -r "TODO-SEC" .                   # Security-related TODOs

# === Performance ===
time npm run build                     # Build performance
du -sh dist/                          # Bundle size
npm run analyze                       # Bundle analysis

# === Debugging ===
# Find recent changes
git log --since="2 days ago" --author="^((?!$(git config user.name)).)*$"

# Search for issues
grep -r "FIXME\|TODO\|HACK\|XXX" --exclude-dir=node_modules .

# === Emergency ===
# Quick backup
git add -A && git commit -m "WIP: Emergency backup"

# Revert last commit
git revert HEAD --no-edit

# Stash work
git stash push -m "WIP: $(date +%Y%m%d-%H%M%S)"

# === MCP Tool Shortcuts ===
# Always read first
cat path/to/file.js                   # Desktop commander

# Plan complex work
# "Use sequential thinking to break down [task]"

# Test web features  
# "Use playwright to test [user flow]"

# GitHub operations
gh issue create
gh pr create --draft
```

---

## Emergency Procedures

### If You Break Production
1. **Stay calm** - You're not the first
2. **Alert team** immediately via agreed channel
3. **Rollback first**, investigate later:
   ```bash
   # Kubernetes
   kubectl rollout undo deployment/app
   
   # Heroku  
   heroku rollback
   
   # Git revert
   git revert HEAD --no-edit && git push
   
   # Feature flag
   curl -X POST https://flags.service/disable/feature-name
   ```
4. **Document timeline** while fresh
5. **Blameless post-mortem** within 48 hours

### If You Find a Security Issue
1. **STOP** - Do not commit fixes publicly
2. **Alert security team** via secure channel
3. **Document** in private security tracker
4. **Fix in private** branch/fork
5. **Coordinate disclosure** with security team
6. **Never** discuss in public channels/commits

### If Tests Are Failing in CI/CD
1. Check if `main` is broken: `git checkout main && npm test`
2. Check for flaky tests: Run 3 times locally
3. Check environment differences: Node version, env vars
4. If blocking deployment: Consider `skip` with TODO
5. Create issue for proper fix

### If You Can't Solve Something
1. **Document what you tried** in .claude-session
2. **Create minimal reproduction** if possible
3. **Ask for help** with context:
   - What you're trying to achieve
   - What you've tried
   - Error messages/unexpected behavior
   - Relevant code snippets
4. **Take a break** - Fresh eyes help

---

*Claude Coding Agent v3.1 - Modular, Intelligent, Production-Ready*  
*Core + Auto-detected Modules + MCP Tools + Quality Practices = Excellence*  
*"Read before write, think before code, test before ship"*