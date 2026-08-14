# Setup State Delivery

**Setup State Delivery** is a configuration and environment delivery system for keeping working setups consistent across machines, projects, CI environments, containers, and AI agents.

CLI:

```bash
dlvy
```

Setup State Delivery manages not only configuration files, but the **state that makes an environment usable**:

- configuration files
- environment variables
- shell and CLI settings
- editor and IDE settings
- project-specific configuration
- tool profiles
- MCP configuration
- AI agent configuration
- credential and secret references
- protected metadata

Its goal is simple:

> **Define your setup once, understand what is effective, and deliver it safely wherever you work.**

---

## Why

A modern development environment is rarely defined by a single dotfiles repository.

The effective setup may depend on:

```text
Global
  ↓
User
  ↓
Profile
  ↓
Machine
  ↓
Workspace
  ↓
Project
  ↓
Local
```

At the same time, configuration is scattered across:

```text
~/.gitconfig
shell profiles
environment variables
.env files
editor settings
CLI configuration
SSH configuration
cloud credentials
MCP configuration
AI agent settings
containers
CI/CD variables
secret managers
```

Traditional synchronization tools are effective at copying or generating files, but the real questions are often:

- Which value is effective right now?
- Where did that value come from?
- Which setting is overriding another?
- What differs between these two machines?
- Which parts should be shared globally?
- Which parts belong only to this project?
- How can credentials be delivered without exposing plaintext secrets?
- How can an AI agent inspect and modify configuration safely?

Setup State Delivery treats these as first-class configuration-management problems.

---

## Core model

Setup State Delivery distinguishes four states.

### Declared State

What has been defined and at which scope.

```text
Profile/developer
  editor.fontSize = 14

Project/example
  editor.fontSize = 16
```

### Effective State

The resolved result after precedence and conditions are applied.

```text
editor.fontSize = 16
```

### Materialized State

The representation written for a specific target.

Examples:

```text
~/.gitconfig
.env
settings.json
MCP configuration
shell environment
CLI configuration
```

### Runtime State

What is actually active on the machine or inside the process.

Comparing the expected and observed state allows Setup State Delivery to detect drift.

---

## CLI

The command name is:

```bash
dlvy
```

Typical operations:

```bash
dlvy sync
dlvy status
dlvy diff
dlvy explain
dlvy plan
dlvy apply
dlvy capture
```

### Synchronize a setup

```bash
dlvy sync
```

Synchronization does not mean blindly overwriting files.

Conceptually:

```text
Fetch definitions
      ↓
Inspect target
      ↓
Resolve configuration
      ↓
Compare current state
      ↓
Create change plan
```

Changes can then be reviewed and applied.

```bash
dlvy apply
```

---

## Explain effective configuration

A central feature of Setup State Delivery is the ability to explain **why** a value is active.

```bash
dlvy explain OPENAI_MODEL
```

Example:

```text
Effective value:
  gpt-example

Sources:

  Project/example      gpt-example      ACTIVE
  Profile/developer    gpt-default      SHADOWED
  User                 gpt-base         SHADOWED
```

The same concept can apply to structured configuration:

```bash
dlvy explain git.user.email
dlvy explain mcp.github
dlvy explain editor.fontSize
```

The system therefore manages both:

```text
what is configured
```

and:

```text
why it is configured that way
```

---

## Configuration scopes

Setup State Delivery supports layered configuration rather than forcing everything into one flat master file.

Typical scopes include:

```text
Global
Organization
User
Profile
Machine
Workspace
Project
Local
```

A value may be inherited or overridden at a more specific scope.

The exact hierarchy is part of the configuration model rather than being encoded indirectly through file locations.

---

## Delivery models

Different targets need different forms of synchronization.

Setup State Delivery supports three conceptual delivery modes.

### Definition delivery

Deliver the configuration hierarchy itself.

Suitable for machines where configuration will continue to be inspected and edited.

### Resolved delivery

Resolve the hierarchy first and deliver only the effective configuration.

Suitable for:

- CI
- containers
- temporary VMs
- isolated build environments
- AI agent sandboxes

### Materialized delivery

Deliver generated target-specific artifacts.

This allows targets to consume the resulting configuration without running Setup State Delivery themselves.

---

## Capturing local changes

Not every change begins in the central definition.

Users may change a setting directly in an editor, CLI, or OS interface.

```bash
dlvy capture
```

Setup State Delivery can inspect those changes and turn them into candidate configuration updates.

Example:

```text
editor.fontSize

Declared:
  14

Observed:
  16
```

The user can then decide where the change belongs:

```text
User
Profile
Machine
Workspace
Project
Local
```

A local modification therefore does not automatically propagate to every machine.

---

## Drift detection

```bash
dlvy status
```

Example:

```text
Git             synced
Shell           synced
Editor          drift
Claude Code     synced
MCP             1 change
Environment     2 changes
```

Detailed comparison:

```bash
dlvy diff
```

Where possible, differences are represented semantically rather than only as raw file diffs.

---

## Adapters

Setup State Delivery works through adapters for individual tools and configuration systems.

Potential adapters include:

- Git
- SSH
- shell environments
- environment variables
- VS Code
- Zed
- Claude Code
- Codex
- MCP clients
- Docker
- GitHub CLI
- cloud CLIs
- generic configuration files

An adapter conceptually provides operations such as:

```text
inspect
resolve
plan
apply
capture
verify
```

This allows users and agents to operate on configuration concepts instead of needing to know every underlying file path and format.

---

## Secret and credential delivery

A usable environment frequently requires credentials.

Setup State Delivery treats secret delivery as a core capability rather than an optional addition.

Definitions should normally contain logical references:

```yaml
github:
  token: secret://github/personal
```

instead of plaintext credentials.

Secret material can be backed by external providers such as:

- OS keychains
- 1Password
- Bitwarden
- HashiCorp Vault
- Infisical
- cloud secret managers
- SOPS-compatible storage

Setup State Delivery can also support an end-to-end encrypted store for environments where an external secret manager is undesirable.

---

## End-to-end encrypted synchronization

For confidential data, the synchronization backend should not need access to plaintext.

```text
Plaintext
   ↓
Client-side encryption
   ↓
Encrypted object
   ↓
Sync backend
   ↓
Encrypted object
   ↓
Client-side decryption
```

The sync service acts as a transport and storage layer rather than a trusted secret holder.

Sensitive metadata may also require encryption because information such as:

```text
production hostnames
account identifiers
internal service names
customer environment names
private repository locations
```

can itself be confidential.

---

## Device identity

Each authorized machine has its own cryptographic identity.

```text
Device Private Key
Device Public Key
```

A new device can request access:

```bash
dlvy device add
```

and an existing trusted device can approve it:

```bash
dlvy device approve
```

Secret material can then be made available only to authorized devices.

Lost or retired machines can be revoked:

```bash
dlvy device revoke
```

---

## Secret authorization is separate from configuration synchronization

A configuration referencing a secret does not automatically grant every target permission to decrypt it.

For example:

```yaml
secret:
  github.personal:
    available_on:
      - personal-machine

  github.work:
    available_on:
      - work-machine
```

Setup State Delivery therefore separates:

```text
Configuration synchronization
```

from:

```text
Secret authorization
```

This prevents a project definition from accidentally distributing sensitive credentials to every synchronized machine.

---

## AI agents

AI agents are first-class clients of Setup State Delivery.

The intended model is not:

```text
Agent
  ↓
Find configuration file
  ↓
Guess its format
  ↓
Edit it directly
```

Instead:

```text
Agent
  ↓
Setup State Delivery
  ↓
Inspect
Explain
Plan
Apply
Verify
```

The same configuration engine can be exposed through:

```text
CLI
API
MCP
```

Potential structured operations include:

```text
get_configuration
get_effective_configuration
explain_configuration
inspect_target
detect_drift
create_change_plan
apply_change_plan
```

---

## Secret access from AI agents

Agents should generally not receive plaintext credentials.

Instead of:

```text
get_secret_plaintext
```

prefer operations such as:

```text
check_secret_available
run_with_secret
materialize_for_target
request_secret_access
```

The agent receives the **capability to perform an authorized action**, not necessarily the underlying credential value.

---

## Example workflow

A new development machine can be prepared roughly as follows:

```bash
dlvy init
dlvy device add
```

Approve it from an existing trusted machine:

```bash
dlvy device approve
```

Then inspect and synchronize:

```bash
dlvy sync
dlvy diff
dlvy apply
```

Check the resulting state:

```bash
dlvy status
```

Investigate any unexpected value:

```bash
dlvy explain <configuration-key>
```

Later, if local changes should be incorporated:

```bash
dlvy capture
```

The normal lifecycle becomes:

```text
Define
  ↓
Deliver
  ↓
Use
  ↓
Observe
  ↓
Capture
  ↓
Deliver
```

---

## Architecture

```text
              Setup Definitions
                      │
              Configuration Model
                      │
               Resolution Engine
                      │
                Effective State
                      │
                  Sync Planner
                      │
                 Adapter Layer
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      macOS        Windows          CI
        │             │             │
        └─────────────┼─────────────┘
                      │
                 Runtime State
```

Cross-cutting capabilities include:

```text
Secret providers
End-to-end encryption
Device identity
Authorization
Provenance
Drift detection
Policy
Audit
```

---

## What Setup State Delivery is not

Setup State Delivery is not intended to replace:

- operating-system provisioning systems
- Kubernetes or infrastructure-as-code platforms
- MDM products
- password managers
- package repositories
- enterprise CMDB systems

It operates at the level of **user and developer working-state configuration** and integrates with existing systems where appropriate.

---

## Project identity

**Project:** Setup State Delivery  
**CLI:** `dlvy`

```text
Setup State Delivery
        ↓
       dlvy
```

The project name describes what the system does.

The CLI keeps everyday interaction short:

```bash
dlvy sync
```

The long-term objective is to make a working setup **portable, explainable, reproducible, and safely deliverable** across the environments where people and agents actually work.
