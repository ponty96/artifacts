# Coding standards

These standards apply to TypeScript in Voice Incident Commander. Use them for new code, refactors, and code reviews. Follow the project architecture in `docs/architecture.md` when choosing where code belongs.

## Put behavior with the code that owns it

Prefer a class when several operations share state, dependencies, or a lifecycle. Put those operations on the class as named methods. Keep rules that govern state changes with the class that owns the state. For example, `InvestigationRecorder` in `apps/agent/src/reporting/recorder.ts` owns the write queue, call IDs, and the decision to end a recording.

Use an interface to describe what a caller needs from a collaborator. Keep it small and name its methods in the language of the feature. For example, `ToolRecorder` describes the operations for recording the start and end of a tool call. A class can implement that contract and keep its storage details private. [Azure SDK API design](https://azure.github.io/azure-sdk/typescript_design.html)

Smell

```ts
export interface RecordingState {
  ended: boolean;
  writeMessage(text: string): void;
}

export function recordMessage(recording: RecordingState, text: string): void {
  if (!recording.ended) recording.writeMessage(text);
}

export function endRecording(recording: RecordingState): void {
  recording.ended = true;
}
```

Acceptable

```ts
export interface Recording {
  recordMessage(text: string): void;
  end(): void;
}

export class InvestigationRecording implements Recording {
  constructor(writeMessage: (text: string) => void) {
    this.writeMessage = writeMessage;
  }

  recordMessage(text: string): void {
    if (!this.ended) this.writeMessage(text);
  }

  end(): void {
    this.ended = true;
  }

  // Private

  private ended = false;
  private readonly writeMessage: (text: string) => void;
}
```

The class controls when recording ends. Callers can record a message or end the recording through the `Recording` interface.

## Group stateless functions in a module

When related functions can take all their inputs as arguments, prefer an Elixir-style module. Group the functions in a TypeScript file named for the feature or domain concept. Use named exports for the operations that other modules call. If the module name clarifies the call, import it as a namespace, as in `ScenarioGrading.gradeToolRuns(...)`. The file provides that namespace. [Elixir modules and functions](https://hexdocs.pm/elixir/modules-and-functions.html) · [TypeScript modules](https://www.typescriptlang.org/docs/handbook/2/modules.html)

Smell

```ts
// transcript.ts
export class Transcript {
  static includesPhrase(text: string, phrase: string): boolean {
    return text.toLowerCase().includes(phrase.toLowerCase());
  }
}
```

Acceptable

```ts
// transcript.ts
export function includesPhrase(text: string, phrase: string): boolean {
  return text.toLowerCase().includes(phrase.toLowerCase());
}
```

With `import * as Transcript from './transcript.js'`, callers can still write `Transcript.includesPhrase(text, phrase)`.

## Keep related code together

Group code by feature, such as evaluations, investigations, or reporting. Keep small types, rules, and helpers together when they describe the same concept and change for the same reason. If two files still share one responsibility and have no useful boundary, combine them. Split a file when a distinct responsibility has a clear name and can be understood through its own inputs and outputs. [VS Code source organization](https://github.com/microsoft/vscode/wiki/Source-Code-Organization) · [Angular style guide](https://angular.dev/style-guide#one-concept-per-file)

Keep the public API, which is the set of declarations that other modules use, small. Give callers an operation that expresses what they want done. Keep the steps of that operation with its implementation. [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html#export-visibility)

Smell

```text
eval/
  run.ts
  scenario.ts
  grading/
    grade-tool-runs.ts
    find-missing-tool-runs.ts
    find-forbidden-tool-runs.ts
```

Acceptable

```text
eval/
  run.ts
  scenario.ts
  scenario-grading.ts
```

When only grading uses the two tool checks, keep both in `scenario-grading.ts` with their caller.

## Put the public API above `// Private`

Start each file with its imports. Then put the public functions, classes, types, and constants that other modules use. Put file-local declarations below a `// Private` comment and leave them unexported. TypeScript modules already keep unexported declarations local. The comment marks where the implementation starts.

Inside a class, list its public members before its private members. Put private members below their own `// Private` comment. Use `private` or `#` to restrict access. Keep field initialization in dependency order. If a module or class has no private section, omit the marker. [TypeScript modules](https://www.typescriptlang.org/docs/handbook/2/modules.html)

Smell

```ts
export function normalizeToolName(server: string, name: string): string {
  return `${server}__${name}`.toLowerCase();
}

export function isTicketCreation(server: string, name: string): boolean {
  return normalizeToolName(server, name) === 'tickets__create_ticket';
}
```

Acceptable

```ts
export function isTicketCreation(server: string, name: string): boolean {
  return normalizeToolName(server, name) === 'tickets__create_ticket';
}

// Private

function normalizeToolName(server: string, name: string): string {
  return `${server}__${name}`.toLowerCase();
}
```

`isTicketCreation` is the operation callers use. The helper can stay local when only this file uses it.

## Name the rule inside a complex loop

Use `for...of` when the body performs ordered work, awaits each step, or records events. For example, in `apps/agent/src/eval/run.ts`, each eval turn waits for the same conversation session. Prefer a collection method when its name makes the intent clear. Use `find` for a lookup, `some` or `every` for a condition, `filter` for selection, and `map` for a transformation. [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html#iterating-containers)

If nested loops and conditions make a rule hard to see, extract the rule into a small named function or method. Use a private method when the rule uses state that a class owns. Use a file-local function when the arguments supply everything the rule needs. Use a name that describes the result or decision, such as `findForbiddenToolRuns`. Keep a small loop inline if its purpose is already clear. Preserve order, duplicate results, and early exits when they affect behavior. [Google code review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html#complexity)

Smell

```ts
// scenario-grading.ts
import { matches, type Scenario } from './scenario.js';

export function gradeToolRuns(
  expectations: Pick<Scenario['expect'], 'mustRun' | 'mustNotRun'>,
  executedTools: readonly string[],
): string[] {
  const reasons: string[] = [];
  for (const pattern of expectations.mustRun) {
    if (!executedTools.some((tool) => matches(tool, pattern))) {
      reasons.push(`${pattern} did not run`);
    }
  }
  for (const pattern of expectations.mustNotRun) {
    for (const tool of executedTools) {
      if (matches(tool, pattern)) {
        reasons.push(`${tool} ran`);
      }
    }
  }
  return reasons;
}
```

Acceptable

```ts
// scenario-grading.ts
import { matches, type Scenario } from './scenario.js';

export function gradeToolRuns(
  expectations: Pick<Scenario['expect'], 'mustRun' | 'mustNotRun'>,
  executedTools: readonly string[],
): string[] {
  return [
    ...findMissingToolRuns(expectations.mustRun, executedTools),
    ...findForbiddenToolRuns(expectations.mustNotRun, executedTools),
  ];
}

// Private

function findMissingToolRuns(patterns: readonly string[], executedTools: readonly string[]): string[] {
  return patterns
    .filter((pattern) => !executedTools.some((tool) => matches(tool, pattern)))
    .map((pattern) => `${pattern} did not run`);
}

function findForbiddenToolRuns(patterns: readonly string[], executedTools: readonly string[]): string[] {
  return patterns.flatMap((pattern) =>
    executedTools.filter((tool) => matches(tool, pattern)).map((tool) => `${tool} ran`),
  );
}
```

`gradeToolRuns` now reads as two named rules. A forbidden pattern still produces one result per matching run.

## Let types express domain rules

Name operations and types after the concepts that callers work with, such as starting an investigation or recording a tool call. Give a function the inputs it uses. For example, scenario grading can take expectations and observations without taking a live session or database connection.

When a value has several valid states with different fields, use a discriminated union. This is a type with a named tag for each state. For example, if a value represents a tool call from start to finish, give skipped, running, and finished calls their own states. Put the completion status on the finished state. Derive a flag such as `ran` from the state if a report needs it. [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)

Smell

```ts
import type { Approval } from '@vic/shared';
import type { ToolCallStatus } from '../reporting/schema.js';

export interface ToolCallResult {
  approval: Approval | null;
  ran: boolean;
  status?: ToolCallStatus;
}
```

Acceptable

```ts
import type { Approval } from '@vic/shared';
import type { ToolCallStatus } from '../reporting/schema.js';

export type ToolCallResult =
  | { state: 'skipped'; approval: Exclude<Approval, 'approved'> }
  | { state: 'running'; approval: 'approved' | null }
  | { state: 'finished'; approval: 'approved' | null; status: ToolCallStatus };
```

The first type accepts `ran: false` with `status: 'success'`. The second requires a status for a finished call and reserves `skipped` for declined or expired approvals.

## Keep expressions and type contracts readable

### Name an awaited result before combining it

When a call combines an awaited result with other work, assign the result to a named variable first. This makes the wait and the next use visible. Keep the order of requests, checks, and writes when you split an expression. A direct `await` is clear when it stands on its own or supplies a single result.

Smell

```ts
reasons.push(
  ...gradeObservations(expectations, observations),
  ...(await checkRootCauseClaims(recordDir, reportModel)),
);
```

Acceptable

```ts
const observationReasons = gradeObservations(expectations, observations);
const claimReasons = await checkRootCauseClaims(recordDir, reportModel);

reasons.push(...observationReasons, ...claimReasons);
```

### Show the order of business decisions

When several conditions decide a result, use guards or a `switch` to make their priority visible. Use a named function when the decisions form one rule. A short conditional expression works well for a single choice.

Smell

```ts
const status = live ? 'in_call' : storedStatus === 'in_call' ? 'paused' : storedStatus;
```

Acceptable

```ts
function displayCallStatus(storedStatus: DisplayStatus, live: boolean): DisplayStatus {
  if (live) {
    return 'in_call';
  }
  if (storedStatus === 'in_call') {
    return 'paused';
  }
  return storedStatus;
}

const status = displayCallStatus(storedStatus, live);
```

### Use the domain type in the signature

Use an existing domain type when it describes what the function needs. If a derived type hides that meaning, give it a name. A generic return type such as `S` promises to preserve every constraint of the caller's type. When the function replaces a field that the caller can narrow, return the result type that the function can guarantee.

Smell

```ts
function replaceStatements<S extends Brief[keyof Brief]>(
  section: S,
  statements: Statement[],
): S {
  return { ...section, statements };
}
```

Acceptable

```ts
function replaceStatements(section: BriefSection, statements: Statement[]): BriefSection {
  return { ...section, statements };
}

const rootCause: Brief['rootCause'] = {
  ...replaceStatements(brief.rootCause, checkedStatements),
  status: brief.rootCause.status,
};
```

### Give non-trivial options a name

If a signature is hard to scan, give its options a domain name. This is especially useful when the signature also destructures the options or sets defaults. Keep short shapes inline when their fields make the contract clear.

Smell

```ts
declare function generateReport(
  dir: string,
  options?: { complete?: Complete; model?: string; force?: boolean },
): Promise<GenerateResult>;
```

Acceptable

```ts
interface GenerateReportOptions {
  complete?: Complete;
  model?: string;
  force?: boolean;
}

declare function generateReport(
  dir: string,
  options?: GenerateReportOptions,
): Promise<GenerateResult>;
```

### Show where input gets checked

When reading JSON from a file, request, or model, use a schema or a runtime check to confirm the expected shape. Handle invalid input according to the caller's error contract. When constructing a known object in code, use its declared type so TypeScript can check the fields. For a nullable value, show the guard near its use when the type still allows it to be absent.

Smell

```ts
const report = JSON.parse(await readFile(reportPath, 'utf8')) as Report;
```

Acceptable

```ts
const reportText = await readFile(reportPath, 'utf8');
const parsedReport: unknown = JSON.parse(reportText);
const report = Report.parse(parsedReport);
```
