# Agent Guidelines for Vertex AI GenMedia Creative Studio

This document provides guidelines for AI agents working on the GenMedia Creative Studio codebase.

## Scope of Work

- Do not modify code within the `experiments/` directory when refactoring or updating the main application. If a global change (like a model deprecation) impacts an experiment, add an inline `# TODO:` or `// TODO:` comment near the affected code in the experiment rather than attempting to refactor its logic.

## Styling

- Prefer using shared styles from `components/styles.py` for common UI elements and layout structures.
- Page-specific or component-specific styles that are not reusable can be defined locally within those files.

## Google Cloud Storage (GCS)

- All interactions with GCS for storing media or other assets should use the `store_to_gcs` utility function located in `common/storage.py`.
- This function is configurable via `config/default.py` for bucket names.

## Configuration

- Application-level configuration values, such as model IDs, API keys (though avoid hardcoding keys directly), GCS bucket names, and feature flags, should be defined in `config/default.py`.
- Access these configurations by importing `cfg = Default()` from `config.default`.

## State Management

- Global application state (e.g., theme, user information) is managed in `state/state.py`.
- Page-specific UI state should be defined in corresponding files within the `state/` directory (e.g., `state/imagen_state.py`, `state/veo_state.py`).

## Error Handling

- For errors that occur during media generation processes and need to be communicated to the user, use the `GenerationError` custom exception defined in `common/error_handling.py`.
- Display these errors to the user via dialogs or appropriate UI elements.
- Log detailed errors to the console/server logs for debugging.

## Adding New Generative Models

- When adding a new generative model capability (e.g., a new type of image model, a different video model):
  - Add model interaction logic (API calls, request/response handling) to a new file in the `models/` directory (e.g., `models/new_model_name.py`).
  - Create UI components for controlling the new model in a subdirectory under `components/` (e.g., `components/new_model_name/generation_controls.py`).
  - Create a new page for the model in `pages/` (e.g., `pages/new_model_name.py`), utilizing the page scaffold and new components.
  - Define any page-specific state in `state/new_model_name_state.py`.
  - Add relevant configurations to `config/default.py`.
  - Update navigation in `config/navigation.json`.

## Metadata

- When storing metadata for generated media, use the `MediaItem` dataclass from `common/metadata.py` and the `add_media_item_to_firestore` function.
- Ensure all relevant fields in `MediaItem` are populated.

## Testing

- Write unit tests for utility functions and model interaction logic.
- Aim to mock external API calls during unit testing.
- Use `pytest` as the testing framework.

## Code Quality

- Use `ruff` for code formatting and linting. Ensure code is formatted (`ruff format .`) and linted (`ruff check --fix .`) before submitting changes.

## Issue Tracking

This project uses **bd (beads)** for issue tracking.
Run `bd prime` for workflow context, or install hooks (`bd hooks install`) for auto-injection.

**Quick reference:**
- `bd ready` - Find unblocked work
- `bd create "Title" --type task --priority 2` - Create issue
- `bd close <id>` - Complete work
- `bd dolt push` - Sync with git (run at session end)

For full workflow details: `bd prime`



## Issue Tracking with Beads (`bd`)

When working with issues using the `bd` (beads) command-line tool, adhere to the following tactical guidelines:

*   **Continuous Synchronization:** Always run `bd dolt push` at the beginning of a session to ensure your local issue tracker is aligned with the remote git repository. Run `bd dolt push` again at the very end of your session before your final `git push`.
*   **State Awareness:** When analyzing a bug or feature request, first run `bd ready` to check if there is already an existing, unblocked issue tracking the work. Do not create duplicate issues.
*   **Atomic Updates:** When you complete a task or fix a bug, immediately close the corresponding issue (`bd close <id>`) in the same commit or PR as the fix. Do not leave issues dangling.
*   **Clear Descriptions:** If you discover a new bug during your workflow that is out of scope for your current task, log it immediately using `bd create "Title" --type bug --priority <level>`. Ensure the description clearly outlines the reproduction steps and the context in which it was found.

## 🤖 GitHub Automation Agents

This repository uses **Google's Gemini CLI** to automate software engineering tasks. Our AI agents assist with code reviews, issue triage, and general maintenance to keep the project moving efficiently.

## Automatic Behaviors

These agents run automatically based on events in the repository.

### 🔎 Code Reviewer

- **Trigger:** When a **Pull Request** is opened.

- **Action:** The agent reviews the code changes (diff), looking for bugs, security issues, and style improvements.
- **Output:** It posts review comments directly on the PR.
- **Note:** The agent focuses on the *diff* only and provides constructive feedback. It does not replace human review.

### 📋 Issue Triage

- **Trigger:** When a new **Issue** is opened.

- **Action:** The agent analyzes the title and body of the issue.
- **Output:** It automatically applies relevant **Labels** (e.g., `bug`, `enhancement`, `question`) to help organize the backlog.

---

## Maintainer Commands

Project maintainers (Owners, Members, Collaborators) can manually invoke the agents using comment commands.

| Command | Description |
| :--- | :--- |
| `@gemini-cli /review` | Manually triggers a full code review on the current Pull Request. |
| `@gemini-cli /triage` | Manually triggers label analysis on the current Issue or PR. |
| `@gemini-cli [question]` | Ask the agent a question about the codebase or request a specific task.  

*Example:* `@gemini-cli Explain how the authentication flow works.` |

> **Note:** These commands are restricted to project maintainers to prevent abuse and manage costs.

---

## Workflow Architecture

The automation is built on GitHub Actions using a "Router-Worker" pattern:

1. **Dispatch Router (`gemini-dispatch.yml`):** The entry point. It listens for events, validates permissions, and routes the request to the correct worker.
2. **Worker Workflows:**
    - `gemini-review.yml`: Handles code analysis.
    - `gemini-triage.yml`: Handles labeling.
    - `gemini-invoke.yml`: Handles general Q&A.

This system is powered by the [Gemini CLI](https://github.com/google-github-actions/run-gemini-cli) action.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
- **SSH PASSPHRASE**: If a `git push` command hangs or fails asking for an SSH passphrase, ABORT the automated push attempt and explicitly instruct the user to execute the push manually.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:ca08a54f -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
- **SSH PASSPHRASE**: If a `git push` command hangs or fails asking for an SSH passphrase, ABORT the automated push attempt and explicitly instruct the user to execute the push manually.
<!-- END BEADS INTEGRATION -->

---

# 🕵️ Custom OSINT Agents

## OSINTNeoAiXXX
**Type:** Advanced Research & Threat Intelligence Agent  
**Version:** 1.0.0  
**Status:** ✅ Production Ready  
**Specialization:** Open Source Intelligence (OSINT) Investigation

### Overview
OSINTNeoAiXXX is a sophisticated autonomous agent that conducts systematic OSINT investigations with institutional-grade rigor. It specializes in threat intelligence gathering, target profiling, infrastructure reconnaissance, and generating comprehensive investigative reports.

### Core Capabilities

#### 🔍 Investigation Types
1. **Target Profiling** - Identity, background, aliases, and public presence mapping
2. **Infrastructure Reconnaissance** - Domain, IP, server, hosting, and DNS analysis
3. **Personnel Research** - Employee directories, organizational hierarchy, staff profiling
4. **Threat Intelligence** - Risk assessment, breach correlation, threat scoring
5. **Digital Footprint Tracking** - Exposed data, leaked information, breach database correlation
6. **Social Media Analysis** - Account discovery, activity patterns, network mapping
7. **Financial Intelligence** - Revenue, funding, SEC filings, business metrics
8. **Geolocation Analysis** - Physical locations, office addresses, infrastructure locations
9. **Technical Stack Analysis** - Technologies, frameworks, dependencies, software detection
10. **Legal & Regulatory** - Lawsuits, compliance records, regulatory filings, violations

#### 📊 Investigation Depth Levels
- **shallow** - Quick scan (5-10 min) - Surface-level overview
- **standard** - Balanced investigation (15-30 min) - Core intelligence
- **deep** - Comprehensive analysis (30-60 min) - In-depth findings
- **comprehensive** - Full spectrum (1-2 hours) - Complete intelligence profile

#### 📋 Output Formats Available
- `summary` - Executive summary & key findings
- `detailed` - Full investigation report with evidence
- `technical` - Infrastructure & technology focus
- `timeline` - Chronological event mapping
- `risk` - Threat assessment & scoring
- `network` - Relationship & connection maps
- `json` - Machine-readable structured data

### Invocation Syntax

```bash
@OSINTNeoAiXXX investigate <target> [--focus <category>] [--depth <level>] [--format <output>]
@OSINTNeoAiXXX profile <target> [--relationships] [--depth <level>]
@OSINTNeoAiXXX threat-intel <target> [--risk-score] [--correlate-breaches]
@OSINTNeoAiXXX timeline <target> [--date-range YYYY-YYYY] [--depth <level>]
```

### Usage Examples

#### Example 1: Corporate Threat Assessment
```
@OSINTNeoAiXXX investigate acme-corp.com --focus infrastructure,employees,financial --depth deep --format risk
```
Produces: Risk profile with threat scores, infrastructure details, employee count, financial standing

#### Example 2: Individual Profile Investigation
```
@OSINTNeoAiXXX profile john.doe@company.com --focus identity,social,digital-footprint --depth standard --format detailed
```
Produces: Identity verification, social media accounts, breach database correlation, activity patterns

#### Example 3: Infrastructure Deep Dive
```
@OSINTNeoAiXXX investigate target-domain.io --focus infrastructure,technical --depth comprehensive --format network
```
Produces: Complete infrastructure map, hosting details, DNS records, connected services, tech stack

#### Example 4: Timeline-Based Historical Analysis
```
@OSINTNeoAiXXX timeline target.com --date-range 2020-2025 --depth deep --format timeline
```
Produces: Chronological event map, domain registration changes, organizational shifts, significant milestones

### Advanced Options

#### Breach & Exposure Correlation
```bash
--correlate-breaches      # Cross-reference with major breach databases
--include-exposed-data    # Show publicly exposed information
--risk-score              # Generate threat/exposure scores
```

#### Relationship Mapping
```bash
--relationships           # Map connections & associations
--extended-network        # Include secondary connections
--hierarchy               # Show organizational structure
```

#### Custom Analysis
```bash
--timeline                # Create chronological event map
--date-range YYYY-YYYY    # Analyze specific time period
--geo-analysis            # Geographic distribution analysis
```

#### Report Customization
```bash
--include-recommendations # Add actionable recommendations
--executive-summary       # High-level summary only
--confidence-levels       # Show data reliability scores
--format json             # Machine-readable output
```

### Investigation Workflow

```
┌─────────────────────────────────────────────────────┐
│          RECONNAISSANCE PHASE                        │
├─────────────────────────────────────────────────────┤
│ • Target validation & identification                │
│ • Scope definition & boundaries                     │
│ • Objective setting & questions                     │
│ • Resource & method planning                        │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│       INFORMATION GATHERING PHASE                   │
├─────────────────────────────────────────────────────┤
│ • Passive intelligence collection                   │
│ • Multi-source data aggregation                     │
│ • Cross-reference verification                      │
│ • Evidence documentation & archival                 │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│          ANALYSIS PHASE                             │
├─────────────────────────────────────────────────────┤
│ • Pattern identification & recognition              │
│ • Connection & relationship mapping                 │
│ • Anomaly detection & flagging                      │
│ • Risk assessment & threat scoring                  │
│ • Confidence level assignment                       │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│        REPORTING & SYNTHESIS PHASE                  │
├─────────────────────────────────────────────────────┤
│ • Executive summary generation                      │
│ • Detailed findings documentation                   │
│ • Evidence trail presentation                       │
│ • Recommendation formulation                        │
│ • Follow-up action items                            │
└─────────────────────────────────────────────────────┘
```

### Ethical & Legal Framework

#### ✅ Approved Use Cases
- Cybersecurity research & threat assessment
- Fraud investigation & due diligence
- Competitive intelligence (legal & ethical)
- Incident response & forensics
- Academic & security research
- Compliance & regulatory investigation
- Employee background checks (with authorization)
- Vendor security assessment
- Risk management & assessment
- Crisis response & investigation

#### ❌ Prohibited Use Cases
- Unauthorized system access or intrusion
- Illegal data acquisition or misuse
- Privacy violations or stalking
- Harassment, intimidation, or threats
- Identity theft preparation
- Fraud perpetration or facilitation
- Violating terms of service deliberately
- Unauthorized surveillance
- Impersonation or deception
- Criminal activity facilitation

### Key Operating Principles

1. **Legality First** - Only uses legally accessible, publicly available information
2. **Documentation** - Maintains complete evidence chains and source attribution
3. **Multi-Source** - Requires multiple independent verification sources
4. **Transparency** - Clearly separates facts from inference and analysis
5. **Confidence Scoring** - Assigns reliability & staleness metrics to all data
6. **Privacy Respect** - Balances investigation necessity with ethical boundaries
7. **Compliance** - Adheres to local laws, regulations, and data protection
8. **Source Verification** - Validates reliability of all information sources

### Data Sources & Intelligence

#### Primary Information Sources
- **Public Records** - WHOIS, ICANN, USPTO, registries
- **Social Media** - Twitter, LinkedIn, GitHub, Reddit, Discord, Facebook
- **Breach Databases** - Have I Been Pwned, Breach Compilation, leak repositories
- **Business Intelligence** - SEC filings, company websites, news archives
- **Technical Data** - GitHub, Stack Overflow, Docker Hub, npm registry
- **IP Intelligence** - MaxMind, ASN lookups, geolocation services
- **Archive Services** - Wayback Machine, DNS history, URL archives
- **Certificates** - Certificate transparency logs, SSL/TLS records
- **Domain Tools** - WHOIS history, DNS records, routing information
- **News & Media** - News archives, press releases, public statements

#### Integration Capabilities
- Real-time data from public APIs
- Historical data analysis & correlation
- Trend identification & prediction
- Temporal analysis & event mapping
- Geographic analysis & visualization
- Network mapping & relationship discovery

#### Geolocation & Environmental Data Sources
- **California Water Boards** - https://geotracker.waterboards.ca.gov/ (Site profiles, environmental data, compliance records)
- **EPA Databases** - Environmental Protection Agency records and geolocation
- **USGS** - United States Geological Survey data
- **OpenStreetMap** - Geographic data & location verification
- **Google Maps** - Location verification and satellite imagery
- **MaxMind GeoIP** - IP geolocation services
- **IP2Location** - Geographic IP intelligence
- **Satellite Imagery** - Property and infrastructure visualization

### Technical Specifications

| Aspect | Details |
|--------|---------|
| **Response Latency** | 30 seconds - 2 hours (depends on depth) |
| **Accuracy Target** | 95%+ with multi-source verification |
| **Concurrent Investigations** | Multiple (parallel processing) |
| **Language Support** | English (extensible) |
| **Data Retention** | Investigation logs per policy |
| **Update Frequency** | Real-time source monitoring capable |
| **Scalability** | Enterprise-grade |
| **API Rate Limits** | Subject to source rate limits |

### Performance Metrics

- **Shallow Investigation**: ~5-10 minutes, 20-50 data points
- **Standard Investigation**: ~15-30 minutes, 50-150 data points
- **Deep Investigation**: ~30-60 minutes, 150-400 data points
- **Comprehensive Investigation**: ~1-2 hours, 400+ data points

### Troubleshooting Guide

| Issue | Solution |
|-------|----------|
| **Incomplete Results** | Increase `--depth` or specify narrower `--focus` categories |
| **High False Positives** | Enable `--correlate` flag, require multiple independent sources |
| **Performance Issues** | Reduce `--depth`, use `--focus` to narrow investigation scope |
| **Source Not Found** | Try alternative categories, expand date ranges, check target validity |
| **Outdated Information** | Data is current as of collection; add `--include-recent-only` flag |
| **Privacy Concerns** | Agent respects all privacy settings and excludes PII when requested |

### Continuous Learning

The agent improves through:
- Source reliability scoring & validation
- Investigation pattern optimization
- Report accuracy feedback loops
- User interaction analysis
- Threat intelligence updates
- Emerging OSINT technique integration

---

**Status:** ✅ Production Ready  
**Last Updated:** 2026-06-02  
**Maintenance Level:** Active & Monitored  
**Support:** Enterprise-grade
