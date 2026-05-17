---
name: frontend-develop
description: Enhance frontend code analysis and development. Trigger when users write, modify, or review React/JS/TS code. Helps optimize code logic, clarify code structure, and improve rendering performance. Suitable for: code review and refactoring suggestions, component splitting design, rendering performance analysis and optimization, code quality issue diagnosis, project structure optimization, state management recommendations, hook logic extraction. Whether fixing bugs, adding features, or optimizing existing code, use this skill to improve code quality and maintainability.
---

# Frontend Skill

When analyzing frontend code, proceed from "overall structure to specifics" to narrow down gradually. Avoid giving too many suggestions at once.

## Analysis Flow

### 1. Overall Structure

Start from the file list or directory structure to understand how the project is organized.

- Are files organized by feature domain?
- Are there any files exceeding 300 lines?
- Is data fetching logic inside components or extracted into hooks/services?

### 2. Component Level

Once the target file is located, check the following dimensions:

| Dimension | Quick Check |
|-----------|-------------|
| Props types | Complete TS types? Interface precise enough? |
| Component size | Keep under 300 lines unless truly inseparable |
| Single file responsibility | Don't declare multiple unrelated components in one file; split into separate files |
| Responsibility boundary | Does it mix data fetching, routing, or other non-rendering concerns? |
| State placement | Is state at the right level? (Push downward when possible) |
| State-component binding | When a state changes, does it only trigger re-renders in components that depend on it? Unrelated components should be isolated |
| Redundant state | Is there state that could be derived from existing data? |

### 3. Logic & Hooks

- Are `useEffect` deps complete? Can it be replaced by computed values or event handlers?
- Should multiple coordinated `useState` calls be extracted into a custom hook?
- Is there a cleanup function (cancel request, clear timer)?

### 4. Rendering & Performance

Only intervene when there are perceptible performance issues. Don't optimize prematurely.

- Are list keys stable and unique?
- Are there inline objects/functions passed to child components?
- Is `React.memo` used ineffectively? (memo is meaningless if props change every time)

### 5. Post-Completion Review

After finishing a component or feature, review two things:

- **Rendering performance**: Any unnecessary re-renders? Are list keys, inline props, and memo usage reasonable?
- **Type checking**: Do props have complete types? Can `any` be replaced?

## Core Principles

- **Single Responsibility**: One component/function does one thing
- **UI & Logic Separation**: Hooks manage logic, components handle rendering
- **Minimal State**: Don't store what can be derived, don't use global when local suffices
- **Progressive Optimization**: Make it correct first, then optimize. Use the profiler to confirm bottlenecks before acting

## Reference Files

Read on demand during analysis — no need to read everything:

| Scenario | File |
|----------|------|
| Project structure, API layer conventions | `references/frontend-code-standards.md` |
| Component splitting, Props design, JSX conventions | `references/react-component-standards.md` |
| Full reference patterns | `examples/` directory |
