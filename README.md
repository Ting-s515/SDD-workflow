<p dir="auto">
  <a href="./README.md"><img src="https://img.shields.io/badge/docs-English-blue" alt="English" /></a>
  <a href="./README.zh-TW.md"><img src="https://img.shields.io/badge/docs-%E7%B9%81%E9%AB%94%E4%B8%AD%E6%96%87-yellow" alt="繁體中文" /></a>
</p>

# SDD Workflow

A skill-based workflow built around Specification-Driven Development (SDD). This repository explains how product requirements can be transformed into structured flows, acceptance criteria, task lists, TDD implementation, review, and synced completion status.

This repo is useful if you want to:

- turn natural-language requirements into executable development tasks
- use a structured workflow to constrain AI implementation, keep the process controllable, and continuously align requirements with delivery
- connect specs, implementation, review, testing, and delivery status in one workflow
- practice document as code so that spec documents do not stay as passive documentation, but directly drive development

## What This Repo Provides

- a reusable SDD core workflow
- a set of skills mapped to each stage of the workflow
- optional extensions for AC-first, executable Feature File generation, React design, and code review guidance

## Workflow At A Glance

```text
Requirements spec
  ↓
propose
  ├─ clarify-flow -> 01-flow.md
  ├─ export-gherkin -> 02-gherkin.md
  └─ 03-tasks.md
  ↓
apply
  ├─ bdd-unit-test -> [BDD]
  ├─ implementation and test validation
  └─ [x][BDD]
  ↓
code-reviewer (when review is needed)
  ↓
propose-sync
  ↓
sync completion status back to the spec
```

## Quick Start

### 1. Prepare a spec document

Start with a requirement document such as:

```md
## Discount Code Feature

Users can enter a discount code on the checkout page.
The system must validate the code and update the order total.
Codes below the threshold must be rejected, and expired codes must return an error.
```

### 2. Run `propose`

Use the workflow to break the requirement into implementation-ready intermediate documents.

```text
propose docs/spec.md
```

Expected output:

```text
docs/propose/<feature-name>/
  01-flow.md
  02-gherkin.md
  03-tasks.md
```

### 3. Run `apply`

Run the generated task list through the TDD flow. `apply` first writes BDD tests for all normal tasks and marks them as `[BDD]`, then implements each task, runs the relevant tests, and marks passing tasks as `[x][BDD]`.

```text
apply docs/propose/<feature-name>
```

### 4. Review or sync completion status

- review the diff against the spec with `code-reviewer`
- add extra file-level unit tests with `bdd-unit-test`
- sync finished work back to the source spec with `propose-sync`

## Repository Structure

```text
skills/
  propose/              entry point for proposal generation
  clarify-flow/         convert requirements into structured flow documents
  export-gherkin/       convert flows into Gherkin acceptance criteria
  apply/                run TDD tests and implementation from task lists
  bdd-unit-test/        generate BDD unit tests
  code-reviewer/        review git diff against specs
  propose-sync/         sync completed features back to the spec

  export-ac/            extension: generate AC documents first
  ac-to-test/           extension: generate red tests from AC
  export-feature-file/  extension: output executable .feature files
  react-design/         extension: React design and review principles

docs/
  document.md           workflow overview document
```

## Core Workflow

The core workflow starts from a spec document, generates proposal artifacts, moves into TDD implementation and validation, runs review when needed, and finally syncs completion status back to the source document.

| Stage                   | Skill            | Output / Task                                   |
| ----------------------- | ---------------- | ----------------------------------------------- |
| 1. Proposal planning    | `propose`        | identify features and generate three documents  |
| 1a. Structured flow     | `clarify-flow`   | `01-flow.md`                                    |
| 1b. Acceptance criteria | `export-gherkin` | `02-gherkin.md`                                 |
| 1c. Task list           | `propose` itself | `03-tasks.md`                                   |
| 2. TDD implementation   | `apply`          | write tests first, then complete `03-tasks.md`  |
| 2a. Test generation     | `bdd-unit-test`  | called by `apply`, or used manually for files   |
| 2b. Code review         | `code-reviewer`  | review git diff against specs and save reports  |
| 3. Completion sync      | `propose-sync`   | write back to the spec's `## 已完成` section    |

### Notes On The Core Flow

- `propose` writes `> propose:` markers back into the spec so that `propose-sync` can locate related feature folders later.
- `propose` can handle normal feature requests and `## bug fix list` items marked as `[quick-fix]`, `[propose]`, or left unmarked for automatic routing.
- `apply` only executes normal tasks automatically; tasks marked as `[manual]` are left for a new session.
- `apply` uses `[BDD]` for tasks with tests written and `[x][BDD]` for tasks whose tests, implementation, and validation are complete.
- `propose-sync` treats non-`[manual]` tasks as complete when they are marked `[x][BDD]`, `[x][widget-test]`, or `[x]`; it no longer depends on `[x][cr]`.
- `code-reviewer` is the review-stage skill. It checks the git diff against the spec and writes reports under `docs/code-review-report/`; it does not update task checkboxes.

## Extension Skills

If you only want to understand the main flow, you can skip this section first. These skills are not mandatory, but they extend the workflow with stronger acceptance and testing strategies.

| Skill                 | Role                                                   | When To Use It                                                              |
| --------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------- |
| `export-ac`           | generate an `AC.md` acceptance-criteria document first | when you want to define completion criteria before implementation           |
| `ac-to-test`          | turn `AC.md` into red test skeletons                   | when you want an AC-first or test-first workflow                            |
| `export-feature-file` | turn specs or Gherkin into executable `.feature` files | when integrating with Reqnroll, Cucumber, Behave, or similar BDD frameworks |
| `react-design`        | provide React architecture and best-practice guidance  | during frontend design, React implementation, or React code review          |

### Extension Flow Examples

#### AC-first path

```text
Requirements spec
  ↓
export-ac -> AC.md
  ↓
ac-to-test -> test skeletons
  ↓
propose / apply -> implementation and validation
```

#### Executable BDD path

```text
Requirements spec / 01-flow.md
  ↓
export-gherkin -> 02-gherkin.md
  ↓
export-feature-file -> .feature
```

## Minimal Example

A typical path looks like this:

1. Prepare `docs/spec.md`
2. Run `propose docs/spec.md`
3. Generate `docs/propose/<feature>/01-flow.md`
4. Generate `docs/propose/<feature>/02-gherkin.md`
5. Generate `docs/propose/<feature>/03-tasks.md`
6. Run `apply docs/propose/<feature>`
7. Let `apply` write tests as `[BDD]`, then implement and mark tasks as `[x][BDD]`
8. Run `code-reviewer`, manual `bdd-unit-test`, or `propose-sync` if needed

## Who This Is For

- individual developers building stable AI-assisted development workflows
- teams that want specs and implementation artifacts tied together
- developers and non-technical stakeholders trying to reduce the gap between requirement understanding and direct implementation

## Related Files

- Workflow overview: [docs/document.md](docs/document.md)
- Skill definitions: [skills](skills)
- License: [LICENSE](LICENSE)
