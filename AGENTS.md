# AGENTS.md

## Debugging and search-space pruning

Treat debugging and implementation as search-space reduction, not speculative patching.

Use this loop:

**Observe → Hypothesize → Prune → Localize → Patch → Verify**

### 1. Separate facts from assumptions

Keep observed facts distinct from hypotheses and assumptions.

Do not promote an assumption to a fact because it appears plausible or because an earlier attempt was based on it.

When evidence is missing, gather evidence first. Prefer logs, reproduction, targeted code inspection, or a focused test over additional speculation.

### 2. Maintain a small live hypothesis set

Keep only a small number of plausible explanations active at once.

For each hypothesis, identify what evidence would support or contradict it.

Prune contradicted hypotheses immediately. Do not repeatedly revisit an eliminated branch unless new evidence invalidates the earlier conclusion.

Avoid stacking speculative fixes for multiple hypotheses at the same time.

### 3. Choose the cheapest high-information evidence

Before editing code, choose the cheapest observation, search, build, or test that eliminates the most uncertainty.

Prefer focused evidence over broad exploration.

Do not run a repository-wide search, full build, full test suite, formatter, or unrelated validation when a smaller check can answer the current question.

Evidence gathering is a measurement step. It should reduce the search space.

### 4. Localize before changing

Identify the smallest component, state transition, invariant, or boundary responsible for the observed behavior.

Do not fix symptoms in downstream components while the causal location is still uncertain.

Once the responsible location is known, stop expanding the search unless new evidence requires it.

### 5. Make the smallest causal patch

Change only what is necessary to address the identified cause.

Avoid opportunistic refactoring, cleanup, formatting, or unrelated improvements during the fix.

A patch should correspond to the current causal hypothesis and be easy to falsify through verification.

### 6. Verify and update the search frontier

Run the cheapest focused verification that can establish whether the patch fixed the identified cause.

If verification passes, stop unless broader validation is explicitly required.

If verification fails, do not stack another speculative patch on top.

Return to the live hypothesis set, incorporate the new evidence, prune invalid branches, and choose the next cheapest discriminating action.

Preserve evidence conservatively; prune hypotheses aggressively.

## Workspace scope

The current working directory is the hard workspace boundary.

Search, planning, implementation, build, and test execution must remain inside the current workspace unless the user explicitly expands the scope.

Before planning or editing, confirm that the target project and relevant files are inside the current workspace.

If the relevant project or files appear to exist outside the current workspace:

1. report the scope mismatch immediately;
2. stop that branch of investigation;
3. do not search neighbouring directories or other workspaces;
4. do not generate a plan that depends on files outside the current workspace.

Do not continue exploratory searches inside an apparently incorrect workspace merely because no matching code was found there.

A scope violation is a reason to stop and correct the workspace, not a reason to broaden the search.

## Agent roles and verification delegation

Use model capability according to the remaining search space.

- **Astra**: use when core assumptions may be wrong, the search space has not converged, or the problem needs to be remodelled or replanned.
- **Sol**: owns the search frontier, hypotheses, uncertainty reduction, implementation decisions, and interpretation of evidence.
- **Luna**: performs bounded mechanical execution once the path is known, such as an explicitly specified build, targeted test, or other deterministic verification.

Do not use a stronger model as a substitute for missing evidence.

When evidence is insufficient, first gather logs, reproduce the problem, inspect the relevant code, or run a focused test.

### Delegated build and test execution

Treat build and test execution as evidence gathering, not reasoning work.

When Luna is available and the command and working directory are already known, delegate build and test execution to Luna by default. This includes bounded mechanical verification such as targeted tests, type checks, lint checks, and builds. Sol or Astra should not spend higher-cost reasoning capacity running deterministic verification loops themselves.

If Luna reports a failure, return control to Sol for interpretation and search-space reduction. Luna must not debug, edit code, or broaden the task unless explicitly reassigned. If Luna is unavailable, the parent agent may run the same bounded verification directly.

When delegating mechanical verification:

- specify the exact command and working directory;
- specify the expected scope;
- do not ask the execution agent to discover scope;
- do not allow it to edit code, debug, or expand the task;
- start with the cheapest focused verification and broaden only when justified;
- on success, return a concise PASS result;
- on failure, return the command, exit status, and the minimum relevant causal error or failing test;
- do not return large undigested logs when a small error excerpt is sufficient.

The parent reasoning agent owns interpretation of failures and decides the next search-space reduction step.
