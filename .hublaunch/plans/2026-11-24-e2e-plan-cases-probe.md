# Fork-only probe: add one marked line to README.md so the sandbox runs this plan's own e2e cases

## Plan Summary

- **What and why:** This is a **fork-only probe** on `thabungm/excalidraw`. It must **never be submitted upstream** (do not run `hula submit-upstream`; the draft PR stays on the fork). It exists to check, on a real repository other than hula-server, that the HubLaunch sandbox e2e stage runs a plan's own `## End-to-End Test Cases` out of the box (no e2e commands configured for this project).
- **The change:** append exactly one line to the end of the repository's `README.md`:

  ```
  <!-- hublaunch-e2e-probe: fork-only line, not for upstream -->
  ```

  Nothing else in the repository changes. No code, no tests, no config, no lockfile.
- **Case files:** the agent writes the two case files named in the End-to-End Test Cases block below, in `.hublaunch/e2e/cases/e2e-plan-cases-probe/`.
- **Priority/complexity:** trivial. Keep the session short.

## Problem Statement

HubLaunch's built-in plan case runner (hula-server PR #1359) has only been tested by unit and shell tests. This probe gives it one real run in a repository that has no e2e commands, no Playwright and no base URL.

## Detailed Requirements

#### Functional Requirements

1. `README.md` ends with the exact line `<!-- hublaunch-e2e-probe: fork-only line, not for upstream -->` (one occurrence, followed by a newline).
2. `.hublaunch/e2e/cases/e2e-plan-cases-probe/readme-line.sh` exists and is executable (`chmod +x`). It is a bash script that:
   - takes the repository root as `$1` (fall back to the current directory when `$1` is empty);
   - checks with `grep -Fxq` that `README.md` in that root contains the exact line above;
   - on success prints exactly `README LINE OK` and exits 0;
   - otherwise prints `README LINE MISSING` and exits 1.
3. `.hublaunch/e2e/cases/e2e-plan-cases-probe/home.mjs` exists. It default-exports
   `async function ({ page, expect })` that opens `https://github.com/thabungm/excalidraw`, waits until `page.locator('body')` is visible, and returns the string `fork page visible`.

#### Out of Scope

- Any change to Excalidraw's source, tests, tooling, CI or dependencies.
- Adding Playwright or any package. The browser case is expected to be reported "not run" by the sandbox; that is part of what the probe checks.

## Implementation Steps

1. Append the line to `README.md`.
2. Create the two case files above; make `readme-line.sh` executable.
3. Run `bash .hublaunch/e2e/cases/e2e-plan-cases-probe/readme-line.sh "$(pwd)"` and confirm it prints `README LINE OK`.
4. Do not run the repository's full test suite for this change; it touches only README.md.

## Acceptance Criteria

- AC1: `README.md` ends with the exact fork-only probe line, and the plan's own script case confirms it from the repository root.
- AC2: The fork's GitHub page is reachable over HTTPS (status 200) from the sandbox.

## End-to-End Test Cases

```json
{
  "version": 1,
  "cases": [
    {
      "index": 1,
      "item": "The README probe line is present (script case run by the sandbox's built-in plan case runner)",
      "driver": "cli",
      "script": ".hublaunch/e2e/cases/e2e-plan-cases-probe/readme-line.sh",
      "expectRegex": "^README LINE OK$",
      "expected": "The script exits 0 and prints exactly `README LINE OK` because README.md contains the exact probe line.",
      "needsRealClaude": false,
      "criterion": "AC1"
    },
    {
      "index": 2,
      "item": "The fork's GitHub page answers 200 over HTTPS (absolute http case)",
      "driver": "http",
      "url": "https://github.com/thabungm/excalidraw",
      "method": "GET",
      "expect": "200",
      "expected": "GET https://github.com/thabungm/excalidraw returns status 200.",
      "needsRealClaude": false,
      "criterion": "AC2"
    },
    {
      "index": 3,
      "item": "Browser case in a project without Playwright or a base URL is reported 'not run' with a reason, never pass or fail",
      "driver": "browser",
      "case": ".hublaunch/e2e/cases/e2e-plan-cases-probe/home.mjs",
      "expected": "The sandbox reports this case 'not run' and names the missing piece (base URL or @playwright/test); it never counts as a pass or a fail.",
      "needsRealClaude": false,
      "criterion": "AC1"
    }
  ]
}
```
