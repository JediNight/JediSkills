# Detection Patterns — All 17 Anti-Patterns + Audit Phases

## Phase 1: Codebase Fingerprinting

```bash
# Check git log for AI co-author signatures
git log --all --format='%an|%ae' | sort -u | grep -iE 'claude|copilot|cursor|ai|bot|agent'

# Check for AI co-authored commits
git log --all --grep='Co-Authored-By' --oneline | head -20

# Estimate AI contribution ratio
git log --all --format='%s' | grep -ciE 'generate|create|implement|add|build' || true

# Check for common AI scaffolding markers
find . -name '*.md' -path '*CLAUDE*' -o -name '.cursorrules' -o -name '.github/copilot*' 2>/dev/null
```

---

## Phase 2: The 17 Vibe Code Anti-Patterns

### 1. Phantom Validation (CRITICAL — Security)

TypeScript types used as runtime validation. Types evaporate at compile time.

```bash
# JS/TS: Find request body typed but not validated
grep -rn 'req\.body as\|req\.body:' --include='*.ts' --include='*.tsx'
# JS/TS: Find missing Zod/Joi/Yup/Valibot in API routes
grep -rL 'z\.\|Joi\.\|yup\.\|v\.\|validate\|parse' $(find . -path '*/api/*' -name '*.ts' 2>/dev/null)
# JS/TS: Check if validation library is even installed
grep -E 'zod|joi|yup|valibot|class-validator' package.json 2>/dev/null

# Python: Find Flask/FastAPI routes without validation
grep -rn 'request\.json\|request\.get_json\|request\.form' --include='*.py' | grep -v 'pydantic\|marshmallow\|cerberus\|voluptuous\|validate'
# Python: Find FastAPI endpoints using dict instead of Pydantic models
grep -rn 'def.*request:\s*dict\|body:\s*dict\|data:\s*dict' --include='*.py' | grep -v test
# Python: Check if validation library is installed
grep -E 'pydantic|marshmallow|cerberus|wtforms|voluptuous' requirements*.txt setup.py pyproject.toml 2>/dev/null
```

**What to report:** Every API endpoint that casts request input to a type annotation or dict without runtime validation.

### 2. Optimistic Auth (CRITICAL — Security)

Auth middleware present on initial routes but missing from endpoints added in later AI sessions.

```bash
# JS/TS: List all route files and their middleware
grep -rn 'app\.\(get\|post\|put\|delete\|patch\)\|router\.\(get\|post\|put\|delete\|patch\)' --include='*.ts' --include='*.js' | head -50
# JS/TS: Check which routes lack auth middleware
grep -rn 'router\.\|app\.' --include='*.ts' --include='*.js' | grep -v 'auth\|protect\|guard\|middleware\|verify\|authenticate'

# Python/FastAPI: Find routes without auth dependencies
grep -rn '@app\.\(get\|post\|put\|delete\|patch\)\|@router\.\(get\|post\|put\|delete\|patch\)' --include='*.py' -A2 | grep -v 'Depends.*auth\|Depends.*current_user\|login_required\|permission_required\|IsAuthenticated'
# Python/Django: Find views without auth decorators
grep -rn 'def.*request' --include='*.py' -B3 | grep -v 'login_required\|permission_required\|IsAuthenticated\|authentication_classes\|test'

# Check which routes were added most recently (context rot risk)
git log --diff-filter=A --since='30 days ago' -p -- '*.ts' '*.js' '*.py' 2>/dev/null | grep -E '^\+.*(route|router|app)\.(get|post|put|delete)' | head -20
```

**What to report:** Routes without auth middleware, especially those created in later commits (check `git log` dates).

### 3. Assertion Mirage (HIGH — Test Quality)

Tests assert existence (`toBeDefined`, `toBeTruthy`) instead of verifying actual business logic values.

```bash
# Count weak vs strong assertions
echo "=== Weak assertions (existence-only) ==="
grep -rc 'toBeDefined\|toBeTruthy\|toBeFalsy\|not\.toBeNull\|not\.toBeUndefined\|toHaveBeenCalled$' --include='*.test.*' --include='*.spec.*' 2>/dev/null | awk -F: '{sum+=$2} END {print sum " total"}'

echo "=== Strong assertions (value-checking) ==="
grep -rc 'toBe(\|toEqual(\|toContain(\|toHaveLength(\|toMatchObject(\|toThrow(' --include='*.test.*' --include='*.spec.*' 2>/dev/null | awk -F: '{sum+=$2} END {print sum " total"}'
```

**Red flag:** If weak assertions > 40% of total assertions, tests are likely theater.

### 4. God Prompt Pattern (MEDIUM — Maintainability)

Single massive component handling data fetching, business logic, state, error handling, and UI.

```bash
# Find oversized components (>200 lines)
find . -name '*.tsx' -o -name '*.jsx' | xargs wc -l 2>/dev/null | sort -rn | head -20
# Find components with too many responsibilities
grep -l 'useState\|useEffect' --include='*.tsx' --include='*.jsx' -r | while read f; do
  hooks=$(grep -c 'use[A-Z]' "$f" 2>/dev/null)
  lines=$(wc -l < "$f")
  if [ "$hooks" -gt 5 ] || [ "$lines" -gt 200 ]; then
    echo "GOD COMPONENT: $f (${lines} lines, ${hooks} hooks)"
  fi
done
```

**What to report:** Components over 200 lines with 5+ hooks. Each should be split.

### 5. Secret Sprinkle (CRITICAL — Security)

API keys, database URLs, and secrets hardcoded in source files.

```bash
# Scan for common secret patterns
grep -rn 'sk_live\|sk_test\|pk_live\|AKIA[0-9A-Z]\|postgres://\|mysql://\|mongodb+srv\|SG\.\|xoxb-\|ghp_\|glpat-\|eyJ[A-Za-z0-9]' --include='*.ts' --include='*.js' --include='*.tsx' --include='*.jsx' --include='*.py' --include='*.env*' 2>/dev/null

# Check if .env is in .gitignore
grep '\.env' .gitignore 2>/dev/null || echo "WARNING: .env not in .gitignore"

# Check for secrets in client-side code
grep -rn 'SUPABASE_SERVICE\|SERVICE_ROLE\|ADMIN_KEY\|SECRET_KEY\|PRIVATE_KEY' --include='*.tsx' --include='*.jsx' --include='*.ts' src/ app/ pages/ 2>/dev/null
```

**What to report:** Every hardcoded secret with exact file:line. Flag client-side secrets as CRITICAL.

### 6. Duplicate Divergence (MEDIUM — Maintainability)

Same utility implemented multiple times with subtly different behavior.

```bash
# Find duplicate utility patterns
for pattern in 'formatDate\|format_date' 'formatCurrency\|format_currency\|formatMoney' 'validateEmail\|isValidEmail\|checkEmail' 'fetchData\|getData\|loadData' 'parseJSON\|safeJSON\|tryParse'; do
  count=$(grep -rl "$pattern" --include='*.ts' --include='*.js' --include='*.tsx' --include='*.py' 2>/dev/null | wc -l)
  if [ "$count" -gt 1 ]; then
    echo "DUPLICATE: pattern '$pattern' found in $count files:"
    grep -rl "$pattern" --include='*.ts' --include='*.js' --include='*.tsx' --include='*.py' 2>/dev/null
  fi
done
```

**What to report:** Each set of duplicate implementations. Note behavioral differences between them.

### 7. Missing Middle (HIGH — Error Handling)

All errors return 500 with generic messages. No 400/401/403/404 distinction.

```bash
# Check status code diversity in API handlers
echo "=== Status codes used ==="
grep -rn 'status(\|statusCode\|res\.status\|HttpStatus\.\|status_code' --include='*.ts' --include='*.js' --include='*.py' | grep -oE '(status\(|HttpStatus\.)[0-9A-Z_]+' | sort | uniq -c | sort -rn

# Find catch blocks with generic error responses
grep -A3 'catch\s*(' --include='*.ts' --include='*.js' -r | grep -B1 '500\|"error"\|"Something went wrong"\|"Internal server error"'
```

**Red flag:** If only 200 and 500 are used, error handling is missing.

### 8. Orphan Migration (HIGH — Data Integrity)

Schema changes in ORM models without corresponding migration files.

```bash
# Compare model changes to migration history
echo "=== Recent model changes ==="
git log --oneline --diff-filter=M -- '**/models/**' '**/entities/**' '**/schema*' 2>/dev/null | head -10

echo "=== Recent migrations ==="
git log --oneline -- '**/migrations/**' '**/alembic/**' '**/prisma/migrations/**' 2>/dev/null | head -10

# Check for Prisma schema drift
npx prisma migrate status 2>/dev/null || true
# Check for Alembic head match
python -c "from alembic.config import Config; from alembic.script import ScriptDirectory; print(ScriptDirectory.from_config(Config('alembic.ini')).get_current_head())" 2>/dev/null || true
```

**What to report:** Model changes without corresponding migrations.

### 9. Console.log Observatory (MEDIUM — Observability)

No structured logging, monitoring, or alerting. Console.log as the only observability.

```bash
# Count console.log vs structured logging
echo "=== Unstructured logging ==="
grep -rc 'console\.\(log\|error\|warn\)' --include='*.ts' --include='*.js' --include='*.tsx' 2>/dev/null | awk -F: '{sum+=$2} END {print sum " console.log calls"}'

echo "=== Structured logging ==="
grep -rc 'logger\.\|winston\.\|pino\.\|bunyan\.\|log\.\(info\|error\|warn\|debug\)' --include='*.ts' --include='*.js' 2>/dev/null | awk -F: '{sum+=$2} END {print sum " structured log calls"}'

# Check for monitoring integration
grep -rl 'sentry\|datadog\|newrelic\|honeycomb\|opentelemetry\|@opentelemetry' package.json requirements.txt 2>/dev/null || echo "WARNING: No monitoring library found"
```

**What to report:** Ratio of console.log to structured logging. Missing monitoring integrations.

### 10. Flat Auth (CRITICAL — Security)

Authentication exists but authorization is missing. Any logged-in user can access any resource by changing the ID (IDOR).

```bash
# Find data queries without tenant/user scoping
grep -rn 'findById\|findOne\|findUnique\|get.*ById\|SELECT.*WHERE.*id' --include='*.ts' --include='*.js' --include='*.py' | grep -v 'user_id\|userId\|tenant\|owner\|created_by\|organization'

# Check for missing RLS in Supabase
find . -name '*.sql' -exec grep -l 'CREATE TABLE' {} \; | while read f; do
  tables=$(grep -c 'CREATE TABLE' "$f")
  rls=$(grep -c 'ENABLE ROW LEVEL SECURITY\|CREATE POLICY' "$f")
  echo "$f: $tables tables, $rls RLS policies"
done

# Check for authorization checks in API routes
grep -rn 'router\.\|app\.' --include='*.ts' --include='*.js' | grep -v 'authorize\|rbac\|permission\|role\|can(\|ability\|casl'
```

**What to report:** Every data access endpoint without authorization scoping.

### 11. Silent Failures (HIGH — Reliability)

Code that runs without errors but produces incorrect results. AI agents often swallow exceptions or return plausible-looking defaults instead of failing visibly.

```bash
# Find empty catch blocks
grep -rn 'catch\s*(' --include='*.ts' --include='*.js' --include='*.py' -A3 | grep -B1 '{\s*}'

# Find catch blocks that only log (no re-throw, no return error)
grep -rn 'catch' --include='*.ts' --include='*.js' -A5 | grep -B3 'console\.\(log\|warn\)' | grep -v 'throw\|reject\|return.*error\|res\.status'

# Find functions returning hardcoded fallback values on error
grep -rn 'catch' --include='*.ts' --include='*.js' --include='*.py' -A3 | grep 'return \[\]\|return {}\|return null\|return ""\|return 0\|return false'

# Python: bare except clauses
grep -rn 'except:' --include='*.py' | grep -v '#.*except'
grep -rn 'except Exception' --include='*.py' -A2 | grep 'pass\|continue'
```

**What to report:** Every catch/except block that silently swallows errors or returns plausible defaults without logging the actual error.

### 12. Tests That Lie (HIGH — Test Quality)

Tests with 100% line coverage but near-zero mutation scores. AI generates tests that execute code paths without verifying behavior.

```bash
# Find test files with no assertions at all
for f in $(find . -name '*.test.*' -o -name '*.spec.*' -o -name 'test_*' 2>/dev/null | grep -v node_modules); do
  assertions=$(grep -c 'expect\|assert\|should\|toBe\|toEqual\|assertEqual' "$f" 2>/dev/null)
  tests=$(grep -c 'it(\|test(\|def test_' "$f" 2>/dev/null)
  if [ "$tests" -gt 0 ] && [ "$assertions" -lt "$tests" ]; then
    echo "SUSPICIOUS: $f ($tests tests, $assertions assertions — some tests have NO assertions)"
  fi
done

# Find tests that only check for no-throw (happy path only)
grep -rn 'expect.*not.*toThrow\|expect.*not.*to\.throw' --include='*.test.*' --include='*.spec.*' | wc -l

# Find tests that mock everything (testing mocks, not code)
grep -c 'jest\.mock\|mock\.\|patch(' --include='*.test.*' --include='*.spec.*' -r 2>/dev/null | awk -F: '$2 > 10 {print "OVER-MOCKED: " $1 " (" $2 " mocks)"}'
```

**Red flag:** Tests where assertion count < test count, or where mock count > assertion count.

### 13. Lazy Output / Code Truncation (CRITICAL — Data Loss)

AI replaces working code with placeholder comments like `// ... existing code ...` or `// rest of implementation`. This silently deletes functioning code.

```bash
# Find truncation markers in source code (NOT in test files or docs)
grep -rn '\.\.\..*existing\|\.\.\..*rest of\|\.\.\..*remaining\|\.\.\..*previous\|\/\/ \.\.\.\|# \.\.\.' --include='*.ts' --include='*.js' --include='*.py' --include='*.tsx' --include='*.jsx' | grep -v 'node_modules\|\.test\.\|\.spec\.\|\.md'

# Check git history for code deletions masked as refactors
git log --all --diff-filter=M --stat -- '*.ts' '*.js' '*.py' 2>/dev/null | grep -B5 'deletion' | grep -E '\d{3,} deletion' | head -10
```

**What to report:** Every placeholder comment in production code. These are code deletions disguised as comments.

### 14. Scope Contamination (MEDIUM — Maintainability)

AI refactors code outside the requested scope — renaming variables, restructuring files, or "improving" unrelated code.

```bash
# Check recent commits for suspiciously large diffs relative to described change
git log --oneline --stat -20 2>/dev/null | while IFS= read -r line; do
  if echo "$line" | grep -qE '^\s+\d+ files? changed'; then
    files=$(echo "$line" | grep -oE '[0-9]+ files? changed' | grep -oE '[0-9]+')
    insertions=$(echo "$line" | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+')
    [ "${files:-0}" -gt 10 ] && echo "SCOPE CREEP: $prev_line — $line"
  else
    prev_line="$line"
  fi
done

# Find renamed variables that weren't in the commit message scope
git diff HEAD~5 --name-only 2>/dev/null | wc -l | xargs -I{} echo "Files changed in last 5 commits: {}"
```

**What to report:** Commits touching >10 files for a single-feature change, or changes to files unrelated to the commit message.

### 15. Over-Engineering (MEDIUM — Maintainability)

AI introduces unnecessary abstractions, design patterns, and configuration layers for simple problems.

```bash
# Find single-implementation abstractions
echo "=== Interfaces/Abstracts with single implementation ==="
for iface in $(grep -rn 'interface \|abstract class \|Protocol:' --include='*.ts' --include='*.py' -l 2>/dev/null); do
  grep -oE '(interface|abstract class) \w+' "$iface" 2>/dev/null | while read -r match; do
    name=$(echo "$match" | awk '{print $NF}')
    impls=$(grep -rl "implements $name\|extends $name\|($name)" --include='*.ts' --include='*.py' 2>/dev/null | grep -v "$iface" | wc -l)
    [ "$impls" -le 1 ] && echo "OVER-ENGINEERED: $name in $iface — only $impls implementation(s)"
  done
done

# Find factory/builder patterns for simple object creation
grep -rn 'Factory\|Builder\|Strategy\|Singleton\|Observer' --include='*.ts' --include='*.js' --include='*.py' | grep -v 'node_modules\|\.test\.\|\.spec\.' | head -15

# Count abstraction layers
find . -name '*.ts' -o -name '*.py' | grep -v node_modules | xargs grep -l 'interface \|abstract \|Protocol' 2>/dev/null | wc -l | xargs -I{} echo "Total abstraction files: {}"
```

**Red flag:** More interface/abstract files than concrete implementation files. Factory patterns with single products.

### 16. Regression Cascades (HIGH — Stability)

Fixing one AI-generated bug introduces 2-3 new bugs because the AI doesn't understand cross-cutting concerns.

```bash
# Find files with high churn (many commits in short period = fix-fix-fix cycle)
echo "=== High-churn files (potential regression cascades) ==="
git log --since='2 weeks ago' --format='%H' -- '*.ts' '*.js' '*.py' 2>/dev/null | while read commit; do
  git diff-tree --no-commit-id --name-only -r "$commit" 2>/dev/null
done | sort | uniq -c | sort -rn | head -10

# Find revert commits
git log --all --oneline --grep='revert\|Revert\|fix.*fix\|undo' 2>/dev/null | head -10

# Find fix-on-fix patterns in commit messages
git log --all --oneline -50 2>/dev/null | grep -iE 'fix|bug|broken|revert|undo|rollback|hotfix' | head -15
```

**Red flag:** Same file appearing in >5 commits within 2 weeks with fix-related messages.

### 17. Security Removal (CRITICAL — Security)

AI agents remove security controls, validation checks, or auth middleware to make error messages disappear. Instead of fixing the root cause, the AI deletes the guard that caused the error. Documented in Columbia University research and DryRun Security study (87% of AI PRs contained vulnerabilities).

```bash
# Check recent git history for removed security patterns (last 100 commits)
git log -100 -p -- '*.ts' '*.js' '*.py' 2>/dev/null | grep -B5 '^-.*\(auth\|validate\|sanitize\|guard\|protect\|verify\|permission\|authorize\|csrf\|cors\|helmet\|rate.limit\)' | grep -A1 '^commit' | head -30

# Find commented-out security code
grep -rn '//.*auth\|//.*valid\|//.*guard\|//.*protect\|#.*auth\|#.*valid' --include='*.ts' --include='*.js' --include='*.py' | grep -v 'node_modules\|\.test\.\|\.spec\.' | head -20

# Check for RLS/policy disabled patterns
grep -rn 'anon\|public\|disable.*security\|skip.*auth\|bypass\|no.verify\|allow_all\|permit_all' --include='*.ts' --include='*.js' --include='*.py' --include='*.sql' | grep -v 'node_modules\|\.test\.' | head -20

# Find recent commits that removed more code than they added (potential security stripping)
git log --oneline --shortstat -20 2>/dev/null | grep -B1 'deletion' | grep -E 'more deletion' | head -10
```

**What to report:** Every instance of removed or commented-out security controls. Cross-reference with git blame to identify if removal happened in an AI-authored commit.

---

## Phase 3: Slopsquatting Check

AI models hallucinate package names. Verify all dependencies actually exist.

```bash
# For Node.js projects
if [ -f package.json ]; then
  echo "=== Checking npm packages ==="
  cat package.json | python3 -c "
import json, sys, urllib.request
data = json.load(sys.stdin)
for section in ['dependencies', 'devDependencies']:
    for pkg in data.get(section, {}):
        try:
            urllib.request.urlopen(f'https://registry.npmjs.org/{pkg}', timeout=5)
        except:
            print(f'SLOPSQUATTING RISK: {pkg} — package may not exist on npm')
"
fi

# For Python projects
if [ -f requirements.txt ]; then
  echo "=== Checking PyPI packages ==="
  while IFS= read -r line; do
    pkg=$(echo "$line" | sed 's/[>=<].*//' | tr -d ' ')
    [ -z "$pkg" ] && continue
    [[ "$pkg" == \#* ]] && continue
    curl -sf "https://pypi.org/pypi/${pkg}/json" > /dev/null || echo "SLOPSQUATTING RISK: $pkg"
  done < requirements.txt
fi
```

---

## Phase 4: Context Rot Indicators

Detect artifacts left when AI agents lose project context. Factory.ai benchmarks (36,611 messages) show artifact trail fidelity of only 2.19-2.45/5.0 across all compression methods — agents forget what they've already built.

### 4a. Contradictory Implementations (HIGH)

Multiple approaches to the same concern within the same codebase. The AI forgot what pattern was established and introduced a different one.

```bash
echo "=== 4a: Contradictory Implementations ==="

# State management (JS/TS)
echo "--- State management ---"
for pattern in 'useState' 'useReducer' 'redux\|createSlice' 'zustand' 'jotai' 'recoil' 'mobx'; do
  count=$(grep -rl "$pattern" --include='*.tsx' --include='*.jsx' --include='*.ts' 2>/dev/null | grep -v node_modules | wc -l | tr -d ' ')
  [ "$count" -gt 0 ] && echo "  $pattern: $count files"
done

# Data fetching (JS/TS)
echo "--- Data fetching ---"
for pattern in 'fetch(' 'axios' 'useQuery\|@tanstack' 'useSWR' 'got(' 'ky('; do
  count=$(grep -rl "$pattern" --include='*.ts' --include='*.tsx' --include='*.js' 2>/dev/null | grep -v node_modules | wc -l | tr -d ' ')
  [ "$count" -gt 0 ] && echo "  $pattern: $count files"
done

# Error handling strategies
echo "--- Error handling ---"
for pattern in 'try.*catch' 'Result<\|Result\[' '\.catch(' 'ErrorBoundary' 'error_handler\|errorhandler'; do
  count=$(grep -rl "$pattern" --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test | wc -l | tr -d ' ')
  [ "$count" -gt 0 ] && echo "  $pattern: $count files"
done

# Config access patterns (Python)
echo "--- Config access ---"
for pattern in 'os\.environ\|os\.getenv' 'pydantic.*Settings\|BaseSettings' 'dotenv\|load_dotenv' 'configparser\|ConfigParser' 'dynaconf'; do
  count=$(grep -rl "$pattern" --include='*.py' 2>/dev/null | grep -v test | wc -l | tr -d ' ')
  [ "$count" -gt 0 ] && echo "  $pattern: $count files"
done

# Logging patterns
echo "--- Logging ---"
for pattern in 'console\.\(log\|error\|warn\)' 'logger\.\|logging\.' 'winston\|pino\|bunyan' 'structlog\|loguru'; do
  count=$(grep -rl "$pattern" --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test | wc -l | tr -d ' ')
  [ "$count" -gt 0 ] && echo "  $pattern: $count files"
done
```

**Red flag:** More than 1 approach per concern = context rot. The AI introduced a second approach because it forgot the first.

### 4b. Lost Context Artifacts (MEDIUM)

Code the AI wrote and then forgot about — dead imports, orphaned functions, unused parameters.

```bash
echo "=== 4b: Lost Context Artifacts ==="

# Python: unused imports (quick heuristic)
echo "--- Unused imports (Python) ---"
for f in $(find . -name '*.py' -not -path '*/node_modules/*' -not -path '*/__init__.py' -not -name 'test_*' 2>/dev/null); do
  while IFS= read -r imp; do
    name=$(echo "$imp" | grep -oE 'import (\w+)' | awk '{print $2}')
    [ -z "$name" ] && continue
    uses=$(grep -c "\b${name}\b" "$f" 2>/dev/null)
    [ "$uses" -le 1 ] && echo "  DEAD IMPORT: $name in $f"
  done < <(grep '^from.*import\|^import ' "$f" 2>/dev/null | head -30)
done 2>/dev/null | head -30

# JS/TS: unused imports (quick heuristic — eslint or tsc is more reliable)
echo "--- Unused imports (JS/TS) ---"
npx tsc --noEmit 2>&1 | grep "is declared but" | head -20 || \
  echo "  (run 'npx eslint --rule no-unused-vars:warn' for detailed detection)"

# Orphaned exported functions (exported but never imported elsewhere)
echo "--- Orphaned exports ---"
for f in $(grep -rl 'export function\|export const\|export class\|export def' --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test | head -20); do
  grep -oE 'export (function|const|class) (\w+)' "$f" 2>/dev/null | awk '{print $3}' | while read name; do
    refs=$(grep -rl "\b${name}\b" --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v "$f" | grep -v node_modules | wc -l | tr -d ' ')
    [ "$refs" -eq 0 ] && echo "  ORPHANED: $name exported from $f, imported nowhere"
  done
done 2>/dev/null | head -20

# Unused function parameters
echo "--- Unused parameters ---"
grep -rn 'def .*(.*)' --include='*.py' 2>/dev/null | grep -v test | grep -v '__' | while IFS= read -r line; do
  file=$(echo "$line" | cut -d: -f1)
  params=$(echo "$line" | grep -oE 'def \w+\(([^)]+)\)' | sed 's/def \w+(\(.*\))/\1/' | tr ',' '\n' | grep -v 'self\|cls\|args\|kwargs' | sed 's/:.*//' | tr -d ' ')
  for p in $params; do
    [ -z "$p" ] && continue
    uses=$(grep -c "\b${p}\b" "$file" 2>/dev/null)
    [ "$uses" -le 1 ] && echo "  UNUSED PARAM: $p in $file"
  done
done 2>/dev/null | head -20
```

**Red flag:** >5% of files with dead imports/orphaned exports = significant context loss.

**False positive note:** Python heuristic will flag library re-exports (`__init__.py`) and type-annotation-only imports. JS/TS detection via `tsc --noEmit` is more reliable than grep heuristics. Orphaned export detection is O(n*m) — slow on large repos; limit to `src/` directory.

### 4c. Style Drift (MEDIUM)

Coding conventions shift mid-file or across the git timeline — the AI's "personality" changes when context resets.

```bash
echo "=== 4c: Style Drift ==="

# Mixed naming conventions in same file
echo "--- Mixed naming in same file ---"
for f in $(find . -name '*.py' -not -path '*/node_modules/*' -not -name 'test_*' 2>/dev/null | head -30); do
  camel=$(grep -cE '\b[a-z]+[A-Z][a-z]+\b' "$f" 2>/dev/null)
  snake=$(grep -cE '\b[a-z]+_[a-z]+\b' "$f" 2>/dev/null)
  # Python should be snake_case; camelCase presence is suspicious
  [ "$camel" -gt 5 ] && [ "$snake" -gt 5 ] && echo "  MIXED: $f (camel: $camel, snake: $snake)"
done

for f in $(find . -name '*.ts' -name '*.js' -not -path '*/node_modules/*' -not -name '*.test.*' 2>/dev/null | head -30); do
  camel=$(grep -cE '\b[a-z]+[A-Z][a-z]+\b' "$f" 2>/dev/null)
  snake=$(grep -cE '\b[a-z]+_[a-z]+\b' "$f" 2>/dev/null)
  # TS/JS should be camelCase; snake_case presence is suspicious
  [ "$snake" -gt 5 ] && [ "$camel" -gt 5 ] && echo "  MIXED: $f (camel: $camel, snake: $snake)"
done

# Quote style inconsistency (JS/TS)
echo "--- Quote style drift ---"
for f in $(find . -name '*.ts' -name '*.js' -name '*.tsx' -not -path '*/node_modules/*' 2>/dev/null | head -20); do
  singles=$(grep -cE "'" "$f" 2>/dev/null)
  doubles=$(grep -cE '"' "$f" 2>/dev/null)
  [ "$singles" -gt 10 ] && [ "$doubles" -gt 10 ] && echo "  MIXED QUOTES: $f (single: $singles, double: $doubles)"
done

# Indentation inconsistency
echo "--- Indentation drift ---"
for f in $(find . \( -name '*.ts' -o -name '*.js' -o -name '*.py' \) -not -path '*/node_modules/*' 2>/dev/null | head -30); do
  tabs=$(grep -cP '^\t' "$f" 2>/dev/null)
  spaces=$(grep -cP '^ {2,}' "$f" 2>/dev/null)
  [ "$tabs" -gt 5 ] && [ "$spaces" -gt 5 ] && echo "  MIXED INDENT: $f (tabs: $tabs, spaces: $spaces)"
done
```

**Red flag:** Style shifts correlating with git timeline = context window resets.

**False positive note:** Python files legitimately mix camelCase (from library APIs like `unittest.TestCase`, `setUp`) with snake_case user code. JS/TS files mix camelCase with PascalCase for React components. Only flag when the inconsistency is within user-authored code in the same language convention layer.

### 4d. Incomplete Refactors (HIGH)

Old pattern and new pattern coexist — the AI started a migration but forgot to finish it.

```bash
echo "=== 4d: Incomplete Refactors ==="

# Find migration artifacts: old and new versions of the same module/function
echo "--- Old/new file pairs ---"
find . -name '*.old.*' -o -name '*.bak.*' -o -name '*.deprecated.*' -o -name '*_old.*' -o -name '*_backup.*' -o -name '*_v2.*' 2>/dev/null | grep -v node_modules

# Find TODO/FIXME referencing migration or refactoring
echo "--- Unfinished migration TODOs ---"
grep -rn 'TODO.*migrat\|TODO.*refactor\|TODO.*deprecat\|FIXME.*old\|FIXME.*legacy\|TODO.*remove.*old' --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | head -15

# Find deprecated imports still in use
echo "--- Deprecated imports still used ---"
grep -rn 'from.*deprecated\|from.*legacy\|from.*old' --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test | head -10

# Detect split-brain: same entity defined in two places
echo "--- Duplicate definitions ---"
grep -rn 'class \w\+\|interface \w\+\|type \w\+ =' --include='*.ts' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test | awk -F'[ :{]' '{print $NF}' | sort | uniq -c | sort -rn | awk '$1 > 1 {print "  DUPLICATE DEF: " $2 " (" $1 " definitions)"}'
```

**Red flag:** Coexisting old/new patterns in the same layer = the AI forgot to complete the migration.

### 4e. Session Boundary Artifacts (MEDIUM)

Signs of context window resets visible in git history — the AI re-introduced something it had already built.

```bash
echo "=== 4e: Session Boundary Artifacts ==="

# Functions that were deleted and re-added (context loss indicator)
echo "--- Deleted-then-recreated code ---"
git log --all --diff-filter=D --name-only --pretty=format: -- '*.ts' '*.js' '*.py' 2>/dev/null | sort -u | while read f; do
  [ -z "$f" ] && continue
  [ -f "$f" ] && echo "  RESURRECTED: $f (was deleted, now exists again)"
done | head -10

# Large commit spikes (context window reset often followed by large regeneration)
echo "--- Sudden large diffs (session resets) ---"
git log --oneline --shortstat -30 2>/dev/null | grep -B1 -E '[0-9]{3,} insertion' | grep -v '^--$' | head -15

# Same utility re-implemented in different files (AI forgot it already existed)
echo "--- Potential re-implementations ---"
for util in 'formatDate\|format_date' 'slugify' 'debounce' 'throttle' 'deepClone\|deep_copy' 'isEqual\|deep_equal' 'retry' 'sleep\|delay' 'hash\|hashCode'; do
  files=$(grep -rl "$util" --include='*.ts' --include='*.js' --include='*.py' 2>/dev/null | grep -v node_modules | grep -v test)
  count=$(echo "$files" | grep -c . 2>/dev/null)
  [ "$count" -gt 1 ] && echo "  RE-IMPLEMENTED: '$util' appears in $count files — AI may have forgotten it already existed"
done

# Detect AI co-author signature gaps (sessions where AI tool changed or was restarted)
echo "--- Session breaks in AI co-author trail ---"
git log --all --format='%H|%s|%an' -50 2>/dev/null | grep -iE 'claude|copilot|cursor|ai|bot' | head -20
```

**Red flag:** Utility functions implemented in multiple files, or files deleted and recreated = the AI lost track of what exists.

---

## Phase 5: Cognitive Debt Assessment

Code that runs but nobody understands:

```bash
# Find files with no comments explaining business logic
find . -name '*.ts' -o -name '*.py' -o -name '*.js' | grep -v node_modules | grep -v '.test.' | while read f; do
  lines=$(wc -l < "$f")
  comments=$(grep -c '//\|#\|/\*\|\*/' "$f" 2>/dev/null || echo 0)
  if [ "$lines" -gt 100 ] && [ "$comments" -lt 3 ]; then
    echo "COGNITIVE DEBT: $f ($lines lines, $comments comments)"
  fi
done

# Find complex functions without documentation
grep -B1 'function\s\|const.*=.*=>' --include='*.ts' --include='*.js' -rn | grep -v 'test\|spec\|node_modules' | head -20
```
