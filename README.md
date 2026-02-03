# Code is Law

Automatically generate documentation that shows what your code **actually enforces** versus what your RULES.md says.

> **Code is Law**: Rules in `RULES.md` are for human guidance. If a workflow or code contradicts written rules, **the code takes precedence**. Always inspect the codebase to understand actual system behavior.

## Quick Start

```bash
# 1. Copy this workflow to your repo
cp .github/workflows/generate-code-is-law.yml <your-repo>/.github/workflows/

# 2. Add a secret (choose one option below)
# Option A: Use free model (no API key needed)
# Add secret: OPENCODE_MODEL=opencode/glm-4.7-free

# Option B: Use your own API key
# Add secret: ZHIPU_API_KEY=your_key_here (for Z.AI models)
# Or add: OPENCODE_API_KEY=your_key_here (for other providers)

# 3. Create a RULES.md file with your rules

# 4. Push to main - CODE_IS_LAW.md will be auto-generated!
```

## What It Does

When you push changes to `RULES.md` or `.github/workflows/`:

1. The workflow reads all your rules and GitHub workflows
2. OpenCode AI analyzes what the code **actually enforces**
3. Generates `CODE_IS_LAW.md` with:
   - **ELI5** - Plain-English summary
   - **The Commandments** - Enforced behaviors in "THOU SHALT / THOU SHALT NOT" style with RFC 2119 keywords (MUST, SHALL, SHOULD, MAY)
   - **Rule vs Code Comparison** - Table showing each rule vs implementation
   - **Workflow Analysis** - What each workflow actually does
4. Commits `CODE_IS_LAW.md` back to your repo

## Configuration

### Step 1: Add Secrets

You need at least **one** of these secrets:

| Secret | Purpose | Required |
|--------|---------|----------|
| `OPENCODE_API_KEY` | Generic API key for any provider | One of these |
| `ZHIPU_API_KEY` | Z.AI API key (for Z.AI models) | |
| `OPENAI_API_KEY` | OpenAI API key | |
| `ANTHROPIC_API_KEY` | Anthropic API key | |
| `GOOGLE_API_KEY` | Google AI API key | |
| `GROQ_API_KEY` | Groq API key | |
| `OPENROUTER_API_KEY` | OpenRouter API key | |
| `GIT_PAT` | Personal Access Token for git push | See below |

**Optional secrets:**
- `OPENCODE_MODEL` - Override model (e.g., `openai/gpt-4o`, `zai-coding-plan/glm-4.7`, `opencode/glm-4.7-free`)
- `OPENCODE_CONFIG_CONTENT` - Full JSON config for custom providers

#### Using `GIT_PAT` (Personal Access Token)

If branch protection is enabled on your main branch, add a `GIT_PAT` secret:

| Setting | Purpose |
|---------|---------|
| **Name** | `GIT_PAT` |
| **Value** | A GitHub Personal Access Token with `repo` scope |
| **How to create** | GitHub Settings → Developer settings → Personal access tokens → Fine-grained token |
| **Permissions needed** | `Contents: Read and write` |

The PAT is used to bypass branch protection when committing `CODE_IS_LAW.md`. Without it, the workflow will fail if branch protection is enabled and prevents direct pushes.

### Step 2: Choose a Model

**Free models (no API key required):**
```
OPENCODE_MODEL=opencode/glm-4.7-free
```

Available free models:
- `opencode/glm-4.7-free` - GLM 4.7 (recommended)
- `opencode/gpt-5-nano`
- `opencode/kimi-k2.5-free`
- `opencode/minimax-m2.1-free`
- `opencode/trinity-large-preview-free`
- `opencode/big-pickle`

**Z.AI models (requires `ZHIPU_API_KEY`):**
```
ZHIPU_API_KEY=your_key_here
OPENCODE_MODEL=zai-coding-plan/glm-4.7
```

**Other providers (requires provider-specific API key):**
```
OPENAI_API_KEY=sk-proj-xxxxx
OPENCODE_MODEL=openai/gpt-4o
```

```
ANTHROPIC_API_KEY=sk-ant-xxxxx
OPENCODE_MODEL=anthropic/claude-sonnet-4
```

### Step 3: Create RULES.md

```markdown
# Repository Rules

## Pull Requests
- All changes must go through pull requests
- At least one approval required
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Workflow fails with "API key not found" | Add at least one API key secret (or use `OPENCODE_MODEL=opencode/glm-4.7-free` for free models) |
| Workflow runs but CODE_IS_LAW.md not updated | Check if `CODE_IS_LAW.md` was the only change (it gets skipped to prevent loops) |
| Cache not working | Wait for the workflow to run twice (first run populates cache) |
| "Model not found" error | Verify model name format: `provider/model` (e.g., `openai/gpt-4o`) |

## How It Works

The `.github/workflows/generate-code-is-law.yml` workflow:

| Feature | Description |
|---------|-------------|
| **Triggers** | Push to `main` when `RULES.md` or `.github/workflows/**` changes |
| **Manual run** | Can trigger manually via GitHub Actions UI |
| **CLI caching** | Caches OpenCode CLI installation for faster subsequent runs |
| **Retry logic** | 3 attempts with exponential backoff on failures |
| **Validation** | Checks generated markdown is valid before committing |
| **Fallback** | Creates placeholder if generation fails |

## Safety Features

- **Path filtering** - Only runs on relevant file changes
- **Infinite loop prevention** - Skips when only `CODE_IS_LAW.md` changed
- **Skip CI** - Uses `[skip ci]` to prevent CI re-triggering
- **Validation** - Ensures generated file meets minimum quality standards

## Example Output

`CODE_IS_LAW.md` is generated with:

```markdown
## ELI5

This repo uses GitHub Actions to enforce that all code changes must go through pull requests with tests passing before they can be merged. If you try to push directly to main, the workflow will block it.

## The Commandments

1. THOU SHALT NOT push directly to main; all changes MUST go through pull requests.
2. Tests MUST pass before any PR can merge.
3. At least one maintainer SHALL review and approve before merge.
4. Contributors MAY request exceptions, but maintainers SHALL have final authority.

## Rule vs Code Comparison

| Rule from RULES.md | Implementation in Code | Status |
|-------------------|------------------------|---------|
| Changes must use PRs | Protected branch rule in settings | ✅ Enforced |
| Tests must pass | CI workflow blocks merge on failure | ✅ Enforced |
```

## Files

- **`RULES.md`** - Your human-readable rules
- **`CODE_IS_LAW.md`** - Auto-generated documentation of actual enforcement
- **`.github/workflows/generate-code-is-law.yml`** - The workflow file

## More Information

- **Supported providers**: https://opencode.ai/docs/providers/
- **OpenCode docs**: https://opencode.ai/docs/
