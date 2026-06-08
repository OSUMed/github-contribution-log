# Contribution 1: Filter changes from Slack notifications

**Contribution Number:** 1
**Student:** Srikanth Medicherla
**Issue:** https://github.com/graphql-hive/console/issues/3730 
**Status:** Phase I Complete

---

## Why I Chose This Issue

[1-2 paragraphs explaining why this issue interests you, how it matches your skills/learning goals, what you hope to learn]

I chose this issue since it aligns with my experience working with Typescript and Slack notifications in a professional environment. This request is clearly defined, the scope is clear, and it also deals with real user workflow. It also matches my current skill level and gives me an oppurtunity to learn from open source Typescript codebases. I was looking for a backend or product focused issue with an active maintainer with clear instructions on what the definition of done is. GraphQL Hive has great documentation, an active community, and the feature is an excellent choice for my first contribution since I worked on something very similar at my work. 

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

The current Slack integration sends notifications for all supported schema changes, including description-only changes. Some teams use these notifications to monitor important schema updates such as breaking or dangerous changes, but are not interested in description changes. Currently, there is no way to filter which change types are included in Slack notifications.

### Expected Behavior

[What should happen?]

Users then should be able to filter out specific type of slack notifications so there is less noise and they can focus on the notifications that are important to them.

### Current Behavior

[What actually happens?]

Right now the users are not able to filter slack notifications. They get all of the notifications including the description changes.

### Affected Components

[Which parts of the codebase are involved?]

Affected components include slack integration, notification generation logic, and also any configs or settings that are used to determine which schema changes are included in outgoing notifications.

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
