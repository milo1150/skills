---
name: angular-test
description: 'Fix, write, or improve Angular/Karma/Jasmine unit tests. Use when fixing failing spec files, writing new specs for Angular 18 standalone components, improving test coverage, or when a coverage gate is below 90%. Edits spec files only â€” never modifies source, config, or build files.'
argument-hint: 'Target source file path (e.g. src/app/components/my.component.ts)'
---

# Angular Test Skill

## 1. Constraints

| Rule                       | Detail                                                                       |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Editable files**         | `**/*.spec.ts` only                                                          |
| **Read-only files**        | All non-spec source files (`.ts`, `.html`, `.scss`, â€¦)                       |
| **Never modify**           | `karma.conf.js`, `angular.json`, `tsconfig*.json`, `package.json`            |
| **Animation setup**        | In tests, use `NoopAnimationsModule` instead of `BrowserAnimationsModule`    |
| **Final test mode**        | Full suite must run headless with `--watch=false`                            |
| **Execution guard**        | Final run must not execute more tests than the discovered suite/test set     |
| **Post-test quality gate** | After tests pass, run lint and Prettier checks, then fix all detected issues |

> If a test cannot be fixed by editing only the spec file, **stop and report** the issue to the developer instead of touching source files.

---

## 2. Test Runner

**Framework:** Karma + Jasmine (Angular 18, standalone components)

### Run all tests (single pass, headless, no watch)

```bash
ng test --watch=false --browsers=ChromeHeadless
```

### Run tests for a specific spec file

```bash
ng test --watch=false --browsers=ChromeHeadless --include='**/<filename>.spec.ts'
```

### Run tests matching a path pattern

```bash
ng test --watch=false --browsers=ChromeHeadless --include='**/kyc/**/*.spec.ts'
```

### Generate coverage report

```bash
ng test --watch=false --browsers=ChromeHeadless --code-coverage
```

---

## 3. Workflow â€” Fix Loop

```
1. Run tests
2. Parse failures â†’ collect failing spec files
3. For each failing spec file:
   a. Read the spec file
   b. Read the corresponding source file (read-only â€” do not edit)
   c. Identify the root cause
   d. Edit ONLY the spec file to fix it
4. Re-run tests
5. Generate / refresh the coverage report
6. From the user-provided source file path, open the matching HTML file under coverage/, for example:
   a. source: `src/app/components/upload-file/upload-file.component.ts`
   b. report: `coverage/app/components/upload-file/upload-file.component.ts.html`
7. Confirm the target source file coverage is greater than 90%
8. Re-check the final Karma/browser console output for any `WARNING`, `WARN`, `ERROR`, or uncaught error lines
9. Verify the final headless `--watch=false` run did not execute more test cases than expected from the suite under test
10. Run lint and Prettier checks after the final passing test run
11. Fix all lint/format issues detected by those checks
12. Repeat until tests pass, target file coverage is greater than 90%, the final console output is clean, the test count is not exceeded, and lint/Prettier checks are clean
```

**Stop condition:** all relevant tests pass, the target file shown in the final coverage HTML report is above `90%`, the final passing run has no unresolved warning/error console output, the final test count does not exceed the expected cases, and lint/Prettier checks have no remaining issues.

---

## 4. Coverage Gate

- The agent must determine the target source file from the user's prompt.
- The agent must translate that source path to the matching per-file HTML report under `coverage/`.
- The agent must inspect the final report file before finishing the task.
- Example mapping:
  - `src/app/components/upload-file/upload-file.component.ts`
  - `coverage/app/components/upload-file/upload-file.component.ts.html`
- The task is not complete unless the target file coverage is strictly more than `90%`.
- If tests pass but the target file coverage is `90%` or lower, continue adding or improving test cases in the related spec file.
- This rule is per target file. For each new user prompt, use the file named in that prompt to locate the correct coverage HTML report.

---

## 5. Parsing Test Output

Look for lines matching:

```
FAILED <DescribeBlock> <ItBlock>
Error: <message>
    at <stack trace>
```

Map each failure to its spec file path to know which file to edit.

After the final passing test run, also inspect the same terminal output for console log lines such as:

```
WARN <message>
WARNING <message>
ERROR <message>
Unhandled Promise rejection: <message>
```

Treat any warning/error line as a blocker unless it is explicitly known to be unrelated and documented by the developer. If the log cannot be cleaned by editing only the spec file, stop and report it instead of changing source code.

Also compare the final Karma summary against the expected suite size. Treat output such as `Executed 125 of 123` or any rerun/duplicate execution beyond the expected case count as a blocker. The final validation run must be a single headless `--watch=false` pass with no excess executions.

---

## 6. Console Log Gate

- The agent must inspect the final successful test run output after the coverage gate has passed.
- The task is not complete if the final run still emits `WARNING`, `WARN`, `ERROR`, uncaught exception, or unhandled rejection logs.
- If those logs can be removed by improving mocks, spies, or expectations in the spec file, do that and rerun the tests.
- If those logs originate from application code or test infrastructure and cannot be fixed in the spec file only, stop and output a report.
- The task is not complete if the final headless `--watch=false` run reports more executed tests than expected for the suite under test.

---

## 7. Common Angular Testing Patterns

### 6.1 Standalone component setup

```typescript
import { NoopAnimationsModule } from '@angular/platform-browser/animations';

await TestBed.configureTestingModule({
  imports: [
    MyComponent, // standalone component under test
    NoopAnimationsModule,
    TranslateModule.forRoot({
      loader: { provide: TranslateLoader, useFactory: HttpLoaderFactory, deps: [] },
    }),
  ],
  providers: [provideHttpClient(withInterceptorsFromDi()), provideHttpClientTesting()],
}).compileComponents();
```

Use this replacement whenever a spec currently imports `BrowserAnimationsModule` to avoid animation-related console warnings during Karma runs.

### 6.2 Mocking a service

```typescript
const mockService = jasmine.createSpyObj('MyService', ['methodA', 'methodB']);
mockService.methodA.and.returnValue(of(fakeData));

TestBed.configureTestingModule({
  providers: [{ provide: MyService, useValue: mockService }],
});
```

### 6.3 Mocking an Observable input / signal

```typescript
// Use BehaviorSubject to control emitted values in tests
const subject = new BehaviorSubject<MyType>(initialValue);
mockService.data$ = subject.asObservable();
```

### 6.4 Triggering change detection

```typescript
fixture.detectChanges(); // sync
await fixture.whenStable(); // wait for async ops
fixture.detectChanges(); // re-check after async
```

### 6.5 Spying on component methods

```typescript
spyOn(component, 'myMethod').and.callThrough();
// or prevent execution:
spyOn(component, 'myMethod').and.returnValue(undefined);
```

---

## 8. Common Failure Causes and Fixes

| Symptom                                                 | Root Cause                    | Fix (spec only)                                                          |
| ------------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------ |
| `NullInjectorError: No provider for XService`           | Missing provider              | Add `{ provide: XService, useValue: mockService }` to `providers`        |
| `Can't bind to 'input' since it isn't a known property` | Missing import                | Add the standalone component/directive/module to `imports`               |
| `TypeError: Cannot read properties of undefined`        | Spy not set up before call    | Set up spy return value in `beforeEach` before `fixture.detectChanges()` |
| `Error: NG0100: ExpressionChangedAfterItHasBeenChecked` | Change detection timing       | Call `fixture.detectChanges()` again after async operations              |
| `Expected spy X to have been called`                    | Method not triggered          | Trigger the correct DOM event or call the method explicitly              |
| `Zone.js` / async timeout                               | Unresolved promise/observable | Use `fakeAsync` + `tick()` or `async`/`await` + `whenStable()`           |

---

## 9. Reporting

When the task is complete, the final report must include:

```md
## Test Completion Report

**Target source file:** src/.../<name>.ts
**Target coverage report:** coverage/.../<name>.ts.html
**Coverage result:** <statement/branches/functions/lines percentages from the report>
**Console log check:** Clean (no WARNING/ERROR lines after the final passing run)
**Lint check:** Clean (no remaining lint errors/warnings)
**Prettier check:** Clean (no formatting violations)
**Status:** Passed (> 90% coverage, tests passing, clean console output, clean lint, clean Prettier)
```

If any test remains failing after **3 fix attempts** on the same spec, output a report in this format and stop:

```
## Unfixable Test Report

**Spec file:** src/.../<name>.spec.ts
**Failing test:** <describe> > <it>
**Error:** <error message>
**Reason cannot fix in spec:** <explanation â€” e.g., requires source file change>
**Suggested action for developer:** <concrete suggestion>
```

If tests pass but the target file coverage cannot be pushed above `90%` by editing the spec file only, output this report and stop:

```
## Coverage Blocked Report

**Target source file:** src/.../<name>.ts
**Target coverage report:** coverage/.../<name>.ts.html
**Current coverage:** <statement/branches/functions/lines percentages from the report>
**Reason cannot reach > 90% in spec only:** <explanation>
**Suggested action for developer:** <concrete suggestion>
```

If tests pass and the target file coverage is above `90%`, but warning/error console logs remain after the final run, output this report and stop:

```
## Console Log Blocked Report

**Target source file:** src/.../<name>.ts
**Target coverage report:** coverage/.../<name>.ts.html
**Console log lines:** <relevant WARNING/ERROR lines>
**Reason cannot clean console in spec only:** <explanation>
**Suggested action for developer:** <concrete suggestion>
```
