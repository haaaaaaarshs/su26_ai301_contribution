# Contribution [1]: [Cannot parse custom colors whose .colorset uses Hex values]

**Contribution Number:** [1]  
**Student:** [Harsh Sabu]  
**Issue:** [[GitHub issue link](https://github.com/skiptools/skip-ui/issues/146)]  
**Status:** [Phase I] [Complete]

---

## Why I Chose This Issue

[I picked this issue because it feels like the right level of challenge for my first real open-source contribution. I’m not trying to jump into a huge feature right away, and this bug seems specific enough for me to understand, reproduce, and work through carefully. The issue is about `.colorset` files using hex color values, which makes sense to me because it sounds like a parsing problem rather than a full app architecture problem.

I also like that this issue gives me a chance to get more comfortable reading Swift code in a real project. I have some programming experience, but I’m still learning how larger codebases are organized, so I wanted an issue where I could focus on tracing one behavior and making a targeted fix. My goal is to understand why decimal color values work, but hex values do not.]

---

## Understanding the Issue

### Problem Description

[Some custom color assets are not being read correctly when their `.colorset` files store color components as hex values instead of regular decimal values. From the issue, it seems like decimal values work, but hex values may fail during parsing and cause the color to show up incorrectly. I still need to reproduce this myself in Phase II, but my current understanding is that the bug is probably happening in the color parsing logic.]

### Expected Behavior

[A custom color should load correctly whether the `.colorset` stores its component values as decimals or hex values.]

### Current Behavior

[When the `.colorset` uses hex values, the color does not appear to parse correctly and may render as the wrong color.]

### Affected Components

[It is likely the color asset parsing code, especially wherever `.colorset` component values are converted into usable color values. I’ll confirm the exact files once I set up and reproduce the issue in Phase II.]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
