# Contribution [1]: [Cannot parse custom colors whose .colorset uses Hex values]

**Contribution Number:** [1]  
**Student:** [Harsh Sabu]  
**Issue:** [[GitHub issue link](https://github.com/skiptools/skip-ui/issues/146)]  
**Status:** [Phase I] [Complete]

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

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
