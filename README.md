<div align="center">

# 🤖 AutoCodeRover JetBrains Plugin

**Autonomous Program Improvement — Directly in Your IDE**

[![JetBrains Plugin](https://img.shields.io/badge/JetBrains-Plugin-blue?logo=jetbrains)](https://plugins.jetbrains.com/)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.8.21-purple?logo=kotlin)](https://kotlinlang.org/)
[![IntelliJ Platform](https://img.shields.io/badge/IntelliJ%20Platform-2023.1+-orange?logo=intellij-idea)](https://plugins.jetbrains.com/docs/intellij/welcome.html)
[![JDK](https://img.shields.io/badge/JDK-17-red?logo=openjdk)](https://openjdk.org/)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Getting Started](#-getting-started)
  - [Build from Source](#1-build-from-source)
  - [Run in Development Mode](#2-run-in-development-mode)
  - [Install the Plugin](#3-install-the-plugin)
  - [Configure the Plugin](#4-configure-the-plugin)
- [Usage Guide](#-usage-guide)
  - [Opening the Plugin](#opening-the-plugin)
  - [Fix a Bug / Add a Feature](#fix-a-bug--add-a-feature)
  - [Build Failure Troubleshooting](#build-failure-troubleshooting)
  - [Test Failure Diagnosis](#test-failure-diagnosis)
  - [Code-Quality Checks (SonarLint)](#code-quality-checks-sonarlint)
- [After an ACR Agent Run](#-after-an-acr-agent-run)
  - [Reviewing Agent Reasoning](#1-reviewing-agent-reasoning--providing-feedback)
  - [Applying the Patch](#2-applying-the-patch)
  - [Rating the Patch](#3-rating-the-patch)
- [Project Structure](#-project-structure)
- [Technical Deep Dive](#-technical-deep-dive)
- [Troubleshooting](#-troubleshooting)
- [Contact](#-contact)

---

## 🔍 Overview

**[AutoCodeRover (ACR)](https://autocoderover.dev/)** is a fully automated approach for resolving GitHub issues — combining **Large Language Models (LLMs)** with program analysis and debugging capabilities to autonomously localize faults, reason about code context, and generate targeted patches.

**AutoCodeRover JetBrains Plugin** brings this autonomous program improvement agent directly into your JetBrains IDE (IntelliJ IDEA, PyCharm, etc.). Instead of context-switching between the terminal, web consoles, and your editor, developers interact with the ACR Agent through a conversational tool window embedded in the IDE — enabling a seamless, human-in-the-loop workflow for:

- 🐛 **Bug fixing** — Describe the issue in natural language; ACR autonomously localizes and patches the code.
- ✨ **Feature addition** — Describe the desired feature; ACR generates the implementation.
- 🔨 **Build failure troubleshooting** — Automatically captures build errors and submits them to ACR for resolution.
- 🧪 **Test failure diagnosis** — Automatically captures test failure reports (including HTML Gradle reports) and prompts ACR for a fix.
- 🔍 **Code-quality remediation** — Runs SonarLint static analysis, displays categorized issues, and sends them to ACR for automated fixes — individually or in batch.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| **Conversational Agent UI** | Chat-based interface with real-time SSE streaming, typewriter animation, and full conversation history. |
| **Autonomous Bug Fixing** | Describe a bug → ACR localizes the fault, retrieves context, and generates a unified diff patch. |
| **Feature Addition** | Describe a feature → ACR analyzes the codebase and produces implementation code. |
| **Build Failure Auto-Capture** | Listens to IntelliJ's `BuildViewManager` and automatically surfaces build errors with one-click ACR submission. |
| **Test Failure Auto-Capture** | Intercepts Gradle HTML test reports, extracts class/method/stack trace details, and prompts ACR. |
| **SonarLint Integration** | Embedded SonarLint analysis engine for Java and Python with expandable issue trees, per-file grouping, individual/batch ACR fixes, and live progress tracking. |
| **Intelligent Patch Application** | Parses unified diffs, applies patches file-by-file, and opens modified files with temporary line highlighting. |
| **Three-Way Merge (Patch Alignment)** | When local code has diverged from ACR's baseline, performs AST-based three-way merge using GumTree to auto-resolve non-conflicting changes. |
| **Agent Reasoning Transparency** | Collapsible panel showing all intermediate agent steps (Context Retrieval, API Selection, Patch Generation) with per-step feedback. |
| **Feedback Loop** | Submit targeted feedback on any agent reasoning step or the final patch — triggers a guided re-run with your instructions. |
| **Patch Rating** | Thumbs-up/down rating sent to the ACR WebConsole for satisfaction tracking and system improvement. |
| **Multi-Provider LLM Support** | Configure OpenAI (GPT-4o, GPT-4 Turbo), Anthropic (Claude 3.5 Sonnet, Claude 3 Opus), or Google Gemini models. |
| **Context Enrichment** | Automatically enriches prompts with code references (PSI-based), cursor history, and currently open files. |
| **IDE Event Listeners** | Real-time monitoring of build/test events via IntelliJ's `BuildProgressListener` API. |
| **Cross-IDE Support** | Compatible with IntelliJ IDEA (Ultimate & Community), PyCharm, and other JetBrains IDEs. |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        JetBrains IDE                                │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                  ACR Tool Window (Swing UI)                   │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐  │  │
│  │  │ Chat History │  │  Input Area  │  │ Send/Stop/Reset/SL  │  │  │
│  │  └──────┬──────┘  └──────┬───────┘  └──────────┬──────────┘  │  │
│  └─────────┼────────────────┼──────────────────────┼─────────────┘  │
│            │                │                      │                │
│  ┌─────────▼────────────────▼──────────────────────▼─────────────┐  │
│  │                   ACRToolWindowFactory                         │  │
│  │  (Orchestration: state management, coroutines, UI updates)    │  │
│  └───┬──────────┬────────────┬──────────────┬───────────────┬────┘  │
│      │          │            │              │               │       │
│  ┌───▼───┐ ┌───▼────┐  ┌───▼──────┐  ┌───▼──────────┐ ┌──▼────┐  │
│  │Actions│ │Listeners│  │  Parser  │  │   Settings   │ │  UI   │  │
│  └───┬───┘ └───┬────┘  └───┬──────┘  └───┬──────────┘ └──┬────┘  │
│      │         │            │             │               │       │
│  ┌───▼─────────▼────────────▼─────────────▼───────────────▼────┐  │
│  │                     IntelliJ Platform SDK                    │  │
│  │  (PSI, VFS, Editor API, Build API, Git4Idea, Document API)  │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────────┘
                              │  HTTPS + SSE
                              ▼
                 ┌────────────────────────┐
                 │  AutoCodeRover Backend │
                 │  (Cloud API Server)    │
                 │                        │
                 │  POST /repo/task       │
                 │  POST /repo/rerun      │
                 │  POST /repo/task/rate  │
                 │  POST /repo/task/stop  │
                 │  GET  /subscribe (SSE) │
                 └────────────────────────┘
```

### Component Breakdown

| Layer | Component | Responsibility |
|---|---|---|
| **UI** | `ACRToolWindowFactory` | Main orchestrator — creates the chat panel, manages state, handles user input, coordinates API calls and UI updates via Kotlin coroutines. |
| **UI** | `addACRMenu` / `addSonarLintMenu` | Renders collapsible menus for ACR agent reasoning steps and SonarLint issue trees with interactive checkboxes and per-issue action buttons. |
| **UI** | `SharedItems` / `MenuData` / `ProgressBorder` | Reusable UI primitives — shared menu components, animated rainbow progress border, `Menu`/`MenuItem` data models. |
| **Actions** | `applyDiff` | Core patch application engine — parses unified diffs, applies changes file-by-file, triggers three-way merge for diverged files, and highlights modified lines in the editor. |
| **Actions** | `SonarLintReportCollector` | Embedded SonarLint analysis engine — loads SonarLint plugins, configures analysis for Java/Python, captures issues with suggested fixes. |
| **Actions** | `LLMClient` | Direct OpenAI API client for auxiliary tasks (AST-to-code conversion, element/class extraction from issue descriptions). |
| **Actions** | `CursorTracker` | Tracks the developer's last 10 cursor positions across files for context enrichment in ACR prompts. |
| **Actions** | `getRepoURL` | Retrieves the Git remote URL and latest commit hash using IntelliJ's `Git4Idea` API. |
| **Actions** | `retrieveFailureDetailsFromHtml` | Parses Gradle HTML test reports to extract class names, method names, and stack traces. |
| **Actions / Git** | `GitDiffUtils` | JGit-based utility — computes changed files against a baseline commit, retrieves file content at specific commits. |
| **Actions / Git** | `GitRepositoryUtils` | JGit-based utility — repository resolution, remote branch detection, latest pushed commit retrieval. |
| **Listeners** | `BuildFailureListener` | Attaches to IntelliJ's `BuildViewManager` to capture build/test failure events in real-time, deduplicates messages, and triggers the chat callback. |
| **Parser** | `applyDiffToBaseline` | Creates a temporary Git repository, commits the baseline file, and applies a unified diff patch using JGit's `git apply`. |
| **Parser** | `converter` | Converts GumTree AST nodes back into Java source code — handles packages, imports, classes, fields, methods, and control flow. |
| **Parser** | `ReferenceFinder` | PSI-based reference finder — traverses Java/Kotlin PSI trees to locate all usages of a given symbol with line numbers and context. |
| **Parser / GumTree** | `Parser` | Wraps GumTree's `JdtTreeGenerator` to parse Java source code (from strings or files) into ASTs. |
| **Parser / GumTree** | `Matcher` | Three-way AST merge engine — computes node triples (baseline/modified/patched), resolves conflicts via isomorphism checks, produces merged code with LLM-assisted AST-to-code conversion. |
| **Parser / GumTree** | `ThreeVersionAST` | Constructs the three AST versions (baseline from Git, current working copy, patched from diff) for three-way merge. |
| **Settings** | `ACRToolWindowSettings` | Persistent project-level settings (API key, URL, project ID, LLM provider/model) stored via IntelliJ's `PersistentStateComponent`. |
| **Settings** | `ACRToolWindowConfigurable` | Settings UI panel — provider/model dropdowns with dynamic model population based on provider selection. |
| **Entry Points** | `OpenIDE` | `JBProtocolCommand` handler for `apply-diff` — enables external tools to trigger diff application via JetBrains protocol URLs. |
| **Entry Points** | `ApplyCodeToEditorAction` | Manual action (Tools menu) — opens a dialog for pasting and applying a unified diff patch directly. |
| **Entry Points** | `ShowRepoInfoAction` | Manual action (Tools menu) — displays the current repository URL and latest commit hash. |

---

## 🛠 Tech Stack

| Category | Technology |
|---|---|
| **Language** | Kotlin 1.8.21 |
| **Build System** | Gradle (Kotlin DSL) with IntelliJ Gradle Plugin 1.13.3 |
| **IDE Platform** | IntelliJ Platform SDK 2023.1.5 (compatible with builds 231–242.*) |
| **HTTP Client** | OkHttp 4.9.2 (with SSE support for real-time streaming) |
| **JSON Processing** | org.json 20220320 |
| **AST Diffing** | GumTree 4.0.0-beta3 (core, client, gen.jdt) + gumtree-spoon-ast-diff 1.100 |
| **Git Operations** | JGit (via GumTree dependency) + IntelliJ Git4Idea plugin API |
| **Static Analysis** | SonarLint Core 10.3.0 + Sonar Plugin API 10.11.0 |
| **Testing** | JUnit 5, Mockito, Kotlin Test |
| **Code Coverage** | JaCoCo |
| **License Reporting** | dependency-license-report 2.0 |

---

## 📦 Prerequisites

| Requirement | Version |
|---|---|
| **JDK** | 17 or later |
| **IntelliJ IDEA** | 2023.1 or later (Ultimate or Community) |
| **Git** | Installed and configured with a remote repository |

---

## 🚀 Getting Started

### 1. Build from Source

```bash
# Clone the repository
git clone https://github.com/AutoCodeRoverSG/jetbrains-plugin.git
cd jetbrains-plugin

# Build the plugin distribution
./gradlew buildPlugin
```

The plugin ZIP file will be generated at `build/distributions/jetbrains-plugin-ACR-1.0.zip`.

### 2. Run in Development Mode

To launch a sandboxed IntelliJ IDEA instance with the plugin pre-loaded for development and testing:

```bash
./gradlew runIde
```

This starts an isolated IDE environment with auto-reload enabled. The plugin is automatically installed in the sandbox — ideal for iterating on changes without affecting your primary IDE installation.

### 3. Install the Plugin

1. Open IntelliJ IDEA.
2. Navigate to **File → Settings → Plugins** (or **IntelliJ IDEA → Preferences → Plugins** on macOS).
3. Click the gear icon (⚙️) → **Install Plugin from Disk...**
4. Select the `jetbrains-plugin-ACR-1.0.zip` file from `build/distributions/`.
5. Click **OK** and restart the IDE to activate the plugin.

### 4. Configure the Plugin

Navigate to **File → Settings → AutoCodeRover Plugin Settings** and fill in:

| Setting | Description | Where to Find |
|---|---|---|
| **AutoCodeRover Authentication Token** | Your ACR API auth token. | [ACR WebConsole](https://uat.autocoderover.dev/) |
| **AutoCodeRover API Domain** | Backend API base URL (default: `https://uat.autocoderover.dev/`). | Provided by your ACR admin. |
| **Project ID** | The ACR project ID for your repository. | [Projects page](https://uat.autocoderover.dev/project/index) on the WebConsole. |
| **LLM Key Name** | The key name for your configured LLM API key. | [User page](https://uat.autocoderover.dev/user/account) on the WebConsole. |
| **Provider** | LLM provider: `OpenAI`, `Anthropic`, `Gemini`, or trial variants. | Dropdown in settings. |
| **Model** | Specific model (e.g., `gpt-4o-2024-11-20`, `claude-3-5-sonnet-20240620`). | Auto-populated based on provider selection. |

Click **Apply** and **OK** to save.

---

## 📖 Usage Guide

### Opening the Plugin

1. Click the **"AutoCodeRover Assistant"** tab on the right-hand side of your IDE window.
2. The plugin interface has three sections:
   - **Chat History** — Displays the conversation between you and the ACR Agent with real-time streaming updates.
   - **Input Area** — Text field at the bottom for typing messages and descriptions.
   - **Button Panel** — Action buttons: Send / Stop, Reset Chat, and Run SonarLint.

---

### Fix a Bug / Add a Feature

1. Click **"Fix a Bug"** or **"Add a Feature"** in the chat panel.
2. Describe the issue or desired feature in the input area.
3. Click **Send** — the plugin will:
   - Enrich your description with detected code references (via PSI analysis), cursor history, and open files.
   - Submit the task to the ACR backend via HTTPS POST.
   - Subscribe to a real-time **Server-Sent Events (SSE)** stream for agent updates.
   - Display intermediate reasoning steps and the final patch with a typewriter animation.
4. After the run completes, review the patch and choose to **Apply** or **Not Apply**.

#### 🎥 Demo Video
Watch the full demonstration of prompting ACR Agent to fix a bug / add a feature:

https://github.com/user-attachments/assets/2f54b897-80f5-4396-a5ec-bd7429ba119b

---

### Build Failure Troubleshooting

When a build failure occurs inside your IDE, the plugin **automatically**:
1. Captures the build error log via IntelliJ's `BuildViewManager` listener.
2. Deduplicates and cleans the error messages (strips XML tags, normalizes whitespace).
3. Displays the structured error in the chat panel with **"Ask ACR"** / **"Not Ask"** buttons.
4. One click sends the error to ACR for autonomous resolution.

No manual intervention required — the plugin monitors build events in real-time.

---

### Test Failure Diagnosis

When a test failure occurs, the plugin **automatically**:
1. Detects the `FinishBuildEvent` with `FailureResult` status.
2. Extracts the Gradle HTML test report link from the build output.
3. Parses the HTML report to extract **class names**, **method names**, and **stack traces** (limited to 6 lines for clarity).
4. Displays the structured failure details in the chat with one-click ACR submission.

#### 🎥 Demo Video
Watch the full demonstration of prompting ACR Agent to diagnose a test failure:

https://github.com/user-attachments/assets/36c861f4-0541-475f-a8b0-e99471fa3601

---

### Code-Quality Checks (SonarLint)

The plugin embeds a full **SonarLint analysis engine** supporting both **Java** and **Python**.

1. **Run Analysis** — Click the **"Run SonarLint"** button to perform static analysis on all source files under the project's `src/` directory.

2. **View Results** — Issues are displayed in an expandable tree menu:
   - Grouped by file (clicking a file path opens it in the editor).
   - Each issue shows: violation message, line number, rule key, and suggested fix (if available).
   - Per-file and global **select all** checkboxes for batch operations.

3. **Fix Individual Issues** — Click the **"Run"** (▶) icon next to any issue to send it to ACR for a targeted fix.

4. **Batch Fix** — Use checkboxes to select multiple issues across files, then click **"Run All"** to queue sequential fixes.
   - Each issue shows a live **animated rainbow progress border** during processing.
   - Completion status is indicated via ✔️ (success) or ❌ (failure) icons next to each issue.

#### 🎥 Demo Video
Watch the full demonstration of prompting ACR Agent to fix SonarLint issues:

https://github.com/user-attachments/assets/7306e662-08b9-4030-a9a5-1043fd80c5d5

---

## 🔄 After an ACR Agent Run

Once AutoCodeRover completes its run, the plugin displays both the intermediate reasoning steps and the final patch in the chat interface. This section outlines all post-run capabilities.

### 1. Reviewing Agent Reasoning & Providing Feedback

- Click **"AutoCodeRover Response Details"** to expand the collapsible panel showing all intermediate agent outputs.
- Each step is displayed as a row with:
  - Agent name (e.g., `Context Retrieval Agent`)
  - Agent subtype (e.g., `Model response (API selection)`)
  - Full output text
- Click the **💬 feedback** icon next to any step to submit targeted feedback:
  - Feedback is tied to the specific agent step and model response ID.
  - Submitted feedback triggers a **guided re-run** — ACR re-executes with your instructions incorporated into the reasoning pipeline (via `/ide/jetbrains/repo/rerun`).

### 2. Applying the Patch

After reviewing the patch, two buttons appear:

- **Apply** — Applies the unified diff to your codebase:
  - Parses the diff and applies changes file-by-file.
  - Detects whether each file has local modifications since the latest pushed commit (using JGit).
  - **For unmodified files**: Applies the patch directly using `git apply` in a temporary repository, then writes the result to the working tree.
  - **For locally modified files**: Triggers **Patch Alignment** (three-way AST merge):
    1. Constructs three ASTs using GumTree: baseline (from latest pushed commit) → modified (current working copy) → patched (diff applied to baseline).
    2. Computes node mappings between all three versions using GumTree's `Matchers`.
    3. For each node, applies merge logic: if baseline = modified → accept patched (ACR's change); if baseline = patched → accept modified (your change); if modified = patched → accept either.
    4. New nodes from either version are attached to their nearest matched ancestor.
    5. The merged AST is converted back to source code using an LLM-assisted converter.
  - All modified files are opened automatically.
  - Changed lines are **temporarily highlighted in yellow** for quick visual reference. Highlights are dismissed on the first click in the editor.

- **Not Apply** — Dismisses the patch. It remains visible in chat history but does not modify your code.

#### 🎥 Demo Video
Watch the full demonstration of ACR Plugin performs Patch Alignment:

https://github.com/user-attachments/assets/41a5ac45-9812-40d2-bce9-3ae7de60bbc3

### 3. Rating the Patch

At the top-right of the patch bubble:
- Click **👍** to rate the patch as helpful (button turns blue, both buttons disabled).
- Click **👎** to indicate dissatisfaction (button turns red, both buttons disabled).

Ratings are sent to the ACR WebConsole via `/ide/jetbrains/repo/task/rate`, enabling continuous system improvement through aggregated user satisfaction tracking.

---

## 📁 Project Structure

```
jetbrains-plugin/
├── build.gradle.kts                 # Build config, dependencies, plugin metadata
├── settings.gradle.kts              # Gradle project settings
├── gradle.properties                # Kotlin compiler flags
├── gradlew / gradlew.bat            # Gradle wrapper scripts
│
├── src/
│   ├── main/
│   │   ├── kotlin/com/example/jetbrainsplugin/
│   │   │   ├── ACRToolWindowFactory.kt       # Main orchestrator (3000+ lines)
│   │   │   ├── ApplyCodeToEditorAction.kt    # Manual diff application via dialog
│   │   │   ├── ChatGPT.kt                    # Development notes / changelog
│   │   │   ├── OpenIDE.kt                    # JBProtocol handler for external diff
│   │   │   ├── ShowRepoInfoAction.kt         # Tools menu: show repo URL & commit
│   │   │   │
│   │   │   ├── actions/
│   │   │   │   ├── applyDiff.kt              # Patch application & merge trigger
│   │   │   │   ├── CursorTracker.kt          # Developer cursor position tracking
│   │   │   │   ├── getRepoURL.kt             # Git remote URL & commit retrieval
│   │   │   │   ├── LLMClient.kt              # OpenAI API client (auxiliary)
│   │   │   │   ├── retrieveFailureDetailsFromHtml.kt  # Gradle HTML report parser
│   │   │   │   ├── SonarLintReportCollector.kt        # Embedded SonarLint engine
│   │   │   │   ├── ZipProjectAction.kt       # Project zipping utility
│   │   │   │   └── git/
│   │   │   │       ├── GitDiffUtils.kt       # Changed files & baseline code
│   │   │   │       └── GitRepositoryUtils.kt # Repo resolution & remote tracking
│   │   │   │
│   │   │   ├── listeners/
│   │   │   │   └── BuildFailureListener.kt   # Build/test failure event listener
│   │   │   │
│   │   │   ├── parser/
│   │   │   │   ├── applyDiffToBaseline.kt    # Temp Git repo + JGit patch apply
│   │   │   │   ├── converter.kt              # GumTree AST → Java source code
│   │   │   │   ├── ReferenceFinder.kt        # PSI-based symbol reference search
│   │   │   │   └── gumtree/
│   │   │   │       ├── Matcher.kt            # Three-way AST merge engine
│   │   │   │       ├── Parser.kt             # GumTree JDT AST parser wrapper
│   │   │   │       └── ThreeVersionAST.kt    # Three-version AST construction
│   │   │   │
│   │   │   ├── settings/
│   │   │   │   ├── ACRToolWindowConfigurable.kt  # Settings UI panel
│   │   │   │   └── ACRToolWindowSettings.kt      # Persistent state component
│   │   │   │
│   │   │   └── ui/
│   │   │       ├── addACRMenu.kt             # ACR response details renderer
│   │   │       ├── addSonarLintMenu.kt       # SonarLint issue tree renderer
│   │   │       ├── MenuData.kt               # Menu/MenuItem data models
│   │   │       ├── ProgressBorder.kt         # Animated rainbow progress border
│   │   │       └── SharedItems.kt            # Shared UI builder functions
│   │   │
│   │   └── resources/
│   │       ├── icon/                          # SVG icons for the plugin UI
│   │       ├── META-INF/plugin.xml            # Plugin descriptor & extensions
│   │       ├── sonarlint-analysis-engine/     # SonarLint engine resources
│   │       └── sonarlint-intellij/plugins/    # SonarLint plugin JARs
│   │
│   └── test/
│       └── kotlin/
│           ├── ACRTest.kt                     # Core ACR integration tests
│           ├── BuildFailureListenerTest.kt    # Build listener unit tests
│           ├── ButtonTest.kt                  # UI button interaction tests
│           ├── ParserTest.kt                  # AST parser unit tests
│           ├── SonarLintTest.kt               # SonarLint analysis tests
│           ├── TypeWriterEffectTest.kt        # Typewriter animation tests
│           └── data/                          # Test fixtures (Java source files)
```

---

## 🔬 Technical Deep Dive

### API Communication

The plugin communicates with the AutoCodeRover backend over **HTTPS** with `acr-auth-token` header authentication using four REST endpoints:

| Endpoint | Method | Purpose |
|---|---|---|
| `/ide/jetbrains/repo/task` | POST | Submit a bug-fix or feature-addition task. Returns a `subscribe_link` for SSE streaming. |
| `/ide/jetbrains/repo/rerun` | POST | Submit user feedback to trigger a guided re-run of a specific agent step. |
| `/ide/jetbrains/repo/task/rate` | POST | Submit a thumbs-up (`1`) or thumbs-down (`2`) rating for a patch. |
| `/ide/jetbrains/repo/task/stop` | POST | Stop a running ACR task and cancel the SSE subscription. |

#### Task Submission Payload

```json
{
  "agent": "IntelliJ IDEA",
  "language": "Java",
  "description": "<user description + code references + cursor history + open files>",
  "type": "bug fixing | feature addition",
  "provider": "OpenAI",
  "model": "gpt-4o-2024-11-20",
  "keyName": "my-llm-key",
  "projectId": "project-123",
  "localPath": "/path/to/project",
  "branch": "main",
  "commit": ""
}
```

### Real-Time Streaming (SSE)

After task submission, the plugin subscribes to a **Server-Sent Events** stream via OkHttp's `EventSource`:
- Agent intermediate outputs (context retrieval, API selection, patch generation) are streamed in real-time.
- Messages are accumulated and parsed as JSON objects.
- When `status == "Finished"`, the plugin extracts the final patch from the second-to-last message.
- Connection timeouts: 120s connect, 120s read, 120s write. TLS 1.2 enforced.

### Three-Way AST Merge (Patch Alignment)

When the developer has made local changes that conflict with ACR's patch:

1. **Baseline AST** — Parsed from the latest pushed commit (retrieved via JGit) using GumTree's `JdtTreeGenerator`.
2. **Modified AST** — Parsed from the current working copy on disk.
3. **Patched AST** — Generated by applying the ACR diff to the baseline file in a temporary Git repository, then parsing the result.
4. **Merge** — GumTree's `Matchers` compute node mappings between all three versions. For each baseline node, a `NodeTriple(baseline, modified, patched)` is constructed, and the merge decision is:
   - `baseline == modified && baseline != patched` → **accept patched** (ACR's change)
   - `baseline == patched && baseline != modified` → **accept modified** (developer's change)
   - `modified == patched && modified != baseline` → **accept either** (both agree)
   - All three differ → **fallback to baseline**
5. New nodes (present in modified or patched but absent in baseline) are attached to their nearest matched ancestor in the merged tree.
6. The merged AST is converted back to source code using an **LLM-assisted AST-to-code converter** that first determines declaration order, then reconstructs the full Java source.

### SonarLint Engine Integration

The plugin embeds the **SonarLint Core 10.3.0** analysis engine:
- Loads SonarLint plugin JARs from the bundled resources directory via `PluginsLoader`.
- Supports **Java** and **Python** languages with language-specific configurations.
- Dynamically resolves source directories, binary paths, test paths, and library classpaths.
- Captures issues with rule keys, line numbers, primary messages, and **suggested fixes** (including text edit ranges and replacement text).
- Runs per-file analysis with configurable Sonar properties (JDK version, source level, Python version, etc.).
- Utilizes IntelliJ's `ModuleManager` and `OrderEnumerator` to resolve project dependencies for accurate analysis.

### Context Enrichment

Before submitting a task to ACR, the plugin enriches the user's description:
1. **Code References** — Uses `ReferenceFinder` (PSI-based traversal of Java and Kotlin files) combined with `LLMClient` to extract symbol names from the description, then locates all usages with line numbers, enclosing method/class names, and visibility modifiers.
2. **Cursor History** — The developer's last 10 cursor positions (file name, line, column) from `CursorTracker` are appended, providing implicit signal about the area of interest.
3. **Open Files** — All currently open editor tabs are listed for additional context.

---

## ❓ Troubleshooting

| Issue | Solution |
|---|---|
| Plugin not appearing after install | Ensure you're running IntelliJ IDEA 2023.1+ and restart the IDE. |
| "Setup Required" dialog on startup | Configure all required fields in **Settings → AutoCodeRover Plugin Settings**. |
| Build/test failures not captured | Ensure the project uses Gradle or IntelliJ's built-in build system. |
| SonarLint analysis fails | Verify `SONARLINT_HOME` and `JDK_HOME` environment variables are set correctly. |
| Patch application fails | Ensure your project has a Git remote configured and a valid pushed commit on the tracking branch. |
| SSE connection timeout | Check network connectivity to the ACR API domain. Verify the API URL in settings. |
| GumTree classes not found | The plugin auto-loads JARs from the sandbox `/lib` directory. Ensure the plugin build completed successfully. |
| Plugin logs | Check **Help → Show Log in Explorer/Finder** for detailed plugin diagnostic logs. |

For additional help, refer to the official [JetBrains Plugin Development Guide](https://plugins.jetbrains.com/docs/intellij/welcome.html).

---

## 📬 Contact

For questions, feedback, or collaboration inquiries:

- 📧 **Email**: [zanwen@u.nus.edu](mailto:zanwen@u.nus.edu)
- 🌐 **Website**: [https://autocoderover.dev/](https://autocoderover.dev/)
- 🔗 **GitHub**: [AutoCodeRoverSG](https://github.com/AutoCodeRoverSG)

---

<div align="center">
<i>Built with ❤️ at the National University of Singapore</i>
</div>
