# 📋 Agent Handover Document
**Date:** 2026-01-12
**Status:** In-Progress (Restructuring Phase)

## 1. Project Overview
We are in the middle of restructuring the `n8n-agent` system. The goal is to move from a chaotic, ad-hoc structure to a standardized **3-Phase Workflow** system.

### New Directory Structure
- **`00_System/`**: Principles, Growth Guide.
- **`01_Workflows/`**: Standard procedures for the AGENT to follow.
- **`02_Projects/`**: Active projects (each contains Scenario, Spec, Guide).
- **`.agent/rules/`**: System prompts & strict rules (e.g. MCP usage).

## 2. Current Status
### ✅ Completed
- [x] **Universal Standards**: Defined `01_Scenario_Design.md` (Logic Architecture) and `02_Node_Spec.md` (Developer-Ready Spec).
- [x] **Rule Enforcement**: Created `.agent/workflows/n8n-mcp-design.md` to automate MCP usage.
- [x] **Project Migration**: Migrated `Kmong_Luction` and `Kmong_China_Translation` to the new structure.
- [x] **Cleanup**: Deleted legacy `기존_agent` folder.

### 🚧 In-Progress: `Hospital_Admin` (요양병원 원무 자동화)
- **Scenario**: `01_Scenario.md` is rewritten as a **Logic Architecture**. It contains strict I/O definitions and the complex "Matching & Aging" algorithms.
- **Spec**: `02_Spec.md` exists but needs to be **synchronized** with the new Scenario deep logic.
- **Guide**: Not started yet.

## 3. Next Actions (For the Next Agent)
Please follow this sequence immediately upon resuming:

1.  **Sync Hospital_Admin Spec**:
    - Open `02_Projects/Hospital_Admin/02_Spec.md`.
    - Update the **Code Node (Logic A, Logic B)** sections to implement the specific algorithms defined in `01_Scenario.md` (Step 2.2).
    - Ensure all parameters are MCP-verified.
2.  **Create Hospital_Admin Guide**:
    - Write `03_Guide.md` focusing on user operations (how to handle "Mismatch" errors, etc.).
3.  **Deploy**:
    - Ask the user to test the `n8n-mcp-design` workflow with a new random request to verify the system works.

## 4. Environment Notes
- **Git**: Initialized. You may need to `git remote add origin <URL>` and push.
- **MCP**: n8n server is active. Always use it for validation.
