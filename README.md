# Contribution [1]: [Cannot parse custom colors whose .colorset uses Hex values]

**Contribution Number:** [1]  
**Student:** [Harsh Sabu]  
**Issue:** [[GitHub issue link](https://github.com/skiptools/skip-ui/issues/146)]  
**Status:** [Phase 4] [Awaiting review / CLA pending / collaboration in progress]

---

## Why I Chose This Issue

I picked this issue because it feels like the right level of challenge for my first real open-source contribution. I’m not trying to jump into a huge feature right away, and this bug seems specific enough for me to understand, reproduce, and work through carefully. The issue is about `.colorset` files using hex color values, which makes sense to me because it sounds like a parsing problem rather than a full app architecture problem.

I also like that this issue gives me a chance to get more comfortable reading Swift code in a real project. I have some programming experience, but I’m still learning how larger codebases are organized, so I wanted an issue where I could focus on tracing one behavior and making a targeted fix. My goal is to understand why decimal color values work, but hex values do not.

---

## Understanding the Issue

### Problem Description

Some custom color assets are not being read correctly when their `.colorset` files store color components as hex values instead of regular decimal values. From the issue, it seems like decimal values work, but hex values may fail during parsing and cause the color to show up incorrectly. I still need to reproduce this myself in Phase II, but my current understanding is that the bug is probably happening in the color parsing logic.

### Expected Behavior

A custom color should load correctly whether the `.colorset` stores its component values as decimals or hex values.

### Current Behavior

When the `.colorset` uses hex values, the color does not appear to parse correctly and may render as the wrong color.

### Affected Components

It is likely the color asset parsing code, especially wherever `.colorset` component values are converted into usable color values. I’ll confirm the exact files once I set up and reproduce the issue in Phase II.

---

## Reproduction Process

### Environment Setup

I cloned my fork of `skiptools/skip-ui`, created a local working branch named `fix-issue-146`, and pushed that branch to my GitHub fork.
The issue I am working on is Issue #146: “Cannot parse custom colors whose .colorset uses Hex values.”

The relevant file I inspected is:
`Sources/SkipUI/SkipUI/Color/Color.swift`

The setup issue I ran into was GitHub authentication when pushing my branch over HTTPS. GitHub no longer accepts normal account passwords for Git operations, so I created a personal access token with `public_repo` access and used that token as the password when pushing the branch.
For reproduction, I focused on tracing the parser behavior in the source code because the issue is isolated to how `.colorset` component strings are converted into numeric color values.

### Steps to Reproduce

1. Clone the `skip-ui` repository locally.
2. Open the repository and navigate to `Sources/SkipUI/SkipUI/Color/Color.swift`.
3. Locate the color asset parsing logic for named colors, specifically the component parsing used by `ColorComponents`.
4. Inspect how the parser converts component strings into numeric values.
5. Use the example from the issue where a `.colorset` stores color components as hexadecimal strings:

   ```json
   {
     "components": {
       "alpha": "1.000",
       "blue": "0xBB",
       "green": "0x11",
       "red": "0x94"
     }
   }

### Reproduction Evidence

- **Branch showing reproduction/planning work:** https://github.com/haaaaaaarshs/skip-ui/tree/fix-issue-146
- **Screenshots/logs:** Not applicable for this phase because the reproduction is based on tracing the parser behavior in the source code rather than a UI screenshot.
- **My findings:** The current parser supports decimal component strings like `"1.000"` but does not support 8-bit hexadecimal component strings like `"0x94"`. Since `Double("0x94")` fails, the component falls back to `0.0`. This explains why a valid `.colorset` color can render as black or incorrect when Xcode stores RGB components in hexadecimal format.

---

## Solution Approach

### Analysis

The root cause is that the current color component parser assumes component values can be converted directly using `Double(...)`. That works for decimal strings such as `"1.000"` but fails for 8-bit hexadecimal strings such as `"0x94"`. When the conversion fails, the parser falls back to `0.0`.

For the example in the issue:

- `"red": "0x94"` should become `148 / 255`
- `"green": "0x11"` should become `17 / 255`
- `"blue": "0xBB"` should become `187 / 255`

Instead, those RGB values are treated as `0.0`, which causes the rendered color to be wrong.

### Proposed Solution

I will update the component parsing logic so it can handle both supported `.colorset` value formats:

1. Decimal component strings, such as `"1.000"`
2. 8-bit hexadecimal component strings, such as `"0x94"`

The fix will add a small helper method that normalizes component values. If the string starts with `"0x"` or `"0X"`, it will be parsed as a hexadecimal integer and divided by `255.0`. Otherwise, the existing decimal parsing behavior will be preserved.

This keeps the change focused and backwards-compatible.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** SkipUI currently does not correctly parse custom colors when a `.colorset` stores RGB components as hexadecimal strings. The parser should support both decimal strings like `"1.000"` and hexadecimal strings like `"0x94"`.

**Match:** The existing `ColorComponents` logic already handles the general decoding flow. The missing piece is a more flexible parser for individual component strings. Similar parsing logic should stay localized to the color component conversion rather than changing unrelated color APIs.

**Plan:**
1. Add a helper method for parsing color component strings.
2. Trim whitespace from the component string before parsing.
3. If the string starts with `"0x"` or `"0X"`, parse the remaining characters as a hexadecimal integer.
4. Convert hexadecimal RGB values into normalized color values by dividing by `255.0`.
5. Continue supporting existing decimal strings without changing their behavior.
6. Replace the current direct `Double(...) ?? 0.0` parsing with the new helper.
7. Add or update tests if there is an appropriate existing color parsing test location.
8. Manually verify that both decimal and hexadecimal `.colorset` values parse correctly.

**Implement:** Implementation will happen in Phase III on this branch:

https://github.com/haaaaaaarshs/skip-ui/tree/fix-issue-146

**Review:** I will keep the change focused on Issue #146 and avoid unrelated refactoring. Before opening a PR, I will review the project’s existing style and contribution expectations.

**Evaluate:** I will verify that decimal values like `"1.000"` still parse correctly, hexadecimal values like `"0x94"`, `"0x11"`, and `"0xBB"` parse correctly, uppercase hex prefixes like `"0X94"` are handled if supported by the helper, and invalid or missing values still fail safely without crashing.

---

## Testing Strategy

### Local Test Command

I ran the project test suite locally with:

```bash
swift test
```

Final result:

```text
JUNIT SUITES 7 TESTS 60 PASSED 58 (97.0%) FAILED 0 SKIPPED 2
Completed Gradle test run for local
```

The test run completed successfully with **0 failures**. The Gradle/JUnit portion reported 7 test suites, 60 tests, 58 passed, 0 failed, and 2 skipped.

### Validation Performed

I validated that the implementation handles the original issue scenario and preserves existing behavior:

- Hexadecimal component strings like `"0x94"`, `"0x11"`, and `"0xBB"` are now recognized.
- Hex values are converted into normalized color component values by dividing by `255.0`.
- Decimal component strings like `"1.000"` still parse through the existing `Double(...)` path.
- Invalid or missing RGB values still fall back to `0.0`.
- Invalid or missing alpha values still fall back to `1.0`.
- The full project test run completed with no failures after the implementation was updated.

### Build and Environment Issues Resolved

Getting the test suite to pass required more work than expected because SkipUI builds through both Swift and Skip's generated Kotlin/Gradle path. During testing, I ran into several setup and build issues:

1. The first `swift test` attempt failed while downloading the Skip macOS binary artifact.
2. After retrying, the build reached the test harness but initially failed because the local toolchain could not find `XCTest`.
3. After fixing the Xcode command-line tool setup, the build progressed further but failed because `gradle` was not installed.
4. After installing Gradle, the generated Android/Gradle test path failed because the Android SDK configuration was missing.
5. After setting up Android Studio, SDK tools, command-line tools, and SDK environment variables, the test run progressed further.
6. The build then failed because my Mac ran out of disk space during Gradle/Kotlin compilation.
7. After clearing generated build files and freeing disk space, the full test run completed successfully with 0 failures.

This process helped confirm that the final code change was not only valid Swift, but also compatible with SkipUI's generated Kotlin build path.

---

## Implementation Notes

### Phase III Build Progress

For Phase III, I implemented the planned fix for SkipUI Issue #146: custom `.colorset` files were not being parsed correctly when Xcode stored RGB component values as hexadecimal strings such as `"0x94"`, `"0x11"`, and `"0xBB"`.

The original parsing logic in `Sources/SkipUI/SkipUI/Color/Color.swift` converted color components directly with `Double(...)`. That worked for decimal values like `"1.000"`, but failed for hexadecimal strings. When the parse failed, the RGB values fell back to `0.0`, which caused the custom color to render incorrectly.

I updated the private `ColorComponents` parsing logic so it now supports both existing decimal component values and Xcode's 8-bit hexadecimal component format. The final implementation:

- Keeps the existing decimal parsing behavior for values like `"1.000"`.
- Detects component strings with a `0x` or `0X` prefix.
- Converts hexadecimal digits manually into an integer value.
- Normalizes the 8-bit color value by dividing by `255.0`.
- Preserves the existing fallback behavior:
  - Missing or invalid RGB values fall back to `0.0`.
  - Missing or invalid alpha values fall back to `1.0`.

### Files Modified

- `Sources/SkipUI/SkipUI/Color/Color.swift`

### Implementation Details

My first implementation used Swift's built-in radix initializer:

```swift
Int(hexValue, radix: 16)
```

That worked conceptually for Swift, but it failed during the Skip/Kotlin build because the Swift-to-Kotlin transpilation path did not translate that initializer cleanly. The generated Kotlin code produced errors around the `Int` constructor and the `radix` parameter.

To fix that, I rewrote the hex parsing in a more explicit and Skip-compatible way. I added a helper that maps each hexadecimal character (`0`-`9`, `a`-`f`, `A`-`F`) to its integer value, then loops through the string and builds the final integer manually:

```swift
intValue = intValue * 16 + digit
```

This made the change more verbose, but it also made the logic clearer and compatible with the project's Swift-to-Kotlin build path.

### Challenges Faced

The biggest challenge was realizing that a normal Swift solution was not automatically the right solution for this project. My first approach used `Int(hexValue, radix: 16)`, which is the natural Swift way to parse a hexadecimal string. However, SkipUI also transpiles Swift into Kotlin, and the generated Kotlin did not support that initializer correctly.

Instead of treating that as just a test failure, I used the error to revise the implementation. I replaced the built-in radix initializer with manual hexadecimal parsing so the logic would be simple enough for Skip's translation path.

Another challenge was getting the full test environment working locally. The fix itself was small, but validating it required setting up and troubleshooting the Skip binary artifact download, Xcode command-line tools, Gradle, Android Studio, Android SDK tools, and local disk space. That took significantly longer than the code change, but it gave me a better understanding of how this project's Swift and Kotlin build systems connect.

### Code Changes

- **Files modified:** `Sources/SkipUI/SkipUI/Color/Color.swift`
- **Branch:** https://github.com/haaaaaaarshs/skip-ui/tree/fix-issue-146
- **Key commit:** https://github.com/haaaaaaarshs/skip-ui/commit/a5072f6
- **Commit message:** `Fix hex color component parsing`

### Approach Decisions

- I kept the change scoped to `ColorComponents` instead of changing the broader color asset loading flow.
- I preserved the existing decimal parsing behavior to avoid breaking `.colorset` files that already worked.
- I avoided `Int(hexValue, radix: 16)` after discovering that it failed in Skip's Kotlin transpilation path.
- I used manual hex parsing so the logic would be explicit, readable, and compatible with both Swift and the generated Kotlin build.

---

## Learnings & Reflections

This phase taught me that open source fixes are not only about writing the smallest code change. They are also about understanding the project's build system, testing path, and platform constraints.

The main technical lesson was that SkipUI is not a normal Swift-only project. A change can make sense in Swift but still fail if it does not translate cleanly into Kotlin. Because of that, I had to think about implementation compatibility across both sides of the project.

The main workflow lesson was to test incrementally and read the errors carefully. Each failure showed a different layer of the project: dependency downloads, XCTest, Gradle, Android SDK setup, disk space, and finally the actual Swift-to-Kotlin compatibility issue. By working through each one, I was able to end Phase III with a pushed implementation and a passing local test run.

---

## Phase III Status

Status: Complete

I implemented the fix, pushed the working branch to my fork, validated the change with `swift test`, and documented the implementation, testing strategy, challenges, and final commit.

---

## Pull Request

**PR Link:** https://github.com/skiptools/skip-ui/pull/471
**PR Description:** This PR fixes `.colorset` parsing for hexadecimal color component values such as `"0x94"` and `"0XFF"`. The existing parser attempted to parse every color component using `Double(...)`, which works for normalized decimal values like `"1.000"` but fails for hex strings. My change adds support for `0x` / `0X` hex component strings by converting 8-bit channel values into normalized `Double` values between 0 and 1, while preserving the existing decimal parsing behavior.

**Maintainer Feedback:**
- June 22: Opened PR #471 to `skiptools/skip-ui` for issue #146.
- June 22: The project workflow check passed successfully, and the branch has no merge conflicts with `skiptools:main`.
- June 22: The CLA bot flagged that my GitHub username was not yet listed in Skip’s `.clabot` file. I opened a separate CLA PR to add my username: https://github.com/skiptools/clabot-config/pull/72
- As of now, there has not been maintainer code review yet. The PR is awaiting review, with the remaining blocker being CLA verification/paperwork.

**Status:** Awaiting review / CLA pending / collaboration in progress

---

## Learnings & Reflections

### Technical Skills Gained

Through this phase, I learned more about how Swift asset catalog `.colorset` files store color components and how the existing SkipUI parser handled those values. The bug came from the parser using `Double(...)` directly, which works for decimal strings but fails for hex strings like `"0x94"`.
My implementation added a focused parsing path for `0x` / `0X` hex component values while keeping the previous decimal behavior unchanged. I also learned more about keeping an open source fix scoped to the actual reported issue instead of expanding the solution into a broader parser redesign.
I used Claude Code as a reviewer, not as the author of the solution. Claude reviewed the committed branch diff and confirmed that the hex parsing fix was scoped, additive, and safe to submit as a minimal PR. It also pointed out that bare decimal 8-bit values like `"255"` may be a separate pre-existing limitation, but expanding into every possible 8-bit format could broaden the scope beyond the original issue.

### Challenges Overcome

The biggest challenge in this phase was not only technical, but also collaborative. Another student, Arul, also ended up working on the same issue. I had originally commented on the GitHub issue and selected it in Slack, but I was not able to claim it in the Google Doc at the time because of traffic. Arul had also posted his progress in Slack and later opened a PR for the same issue.
At first, we both explained our timelines and considered asking the course staff to make a decision. After speaking with Margaret Fero and discussing it with each other, we decided to handle the overlap professionally and work together instead of treating the PRs as competing submissions.
The current plan is to combine the strengths of both approaches: keeping the parsing solution simple and focused while also considering Arul’s stronger test coverage and edge-case planning.

### What I'd Do Differently Next Time

Next time, I would document issue selection more clearly across every required channel as soon as possible. Even though I commented on the GitHub issue and posted in Slack, not getting my claim into the Google Doc created confusion later.
I would also communicate earlier when I notice a possible overlap. This experience taught me that open source contribution is not just about writing code. It also involves coordination, transparency, and handling disagreements professionally.

---

## Resources Used

- GitHub issue #146: https://github.com/skiptools/skip-ui/issues/146
- My PR #471: https://github.com/skiptools/skip-ui/pull/471
- CLA PR #72: https://github.com/skiptools/clabot-config/pull/72
- Skip contribution guide: https://skip.dev/docs/contributing/
- Claude Code review for final implementation review and scope check
- Slack discussion with Arul and guidance from Margaret Fero
