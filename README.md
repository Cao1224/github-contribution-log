# Contribution 2: Attributes tab - Make details view after clicking in an attribute follow the page scroll

**Contribution Number:** 2

**Student:** Yuancheng Cao

**Issue:** [GitHub issue link](https://github.com/hawtio/hawtio-react/issues/2014)

**Status:** Phase II Complete

---

## Why I Chose This Issue

I chose issue #2014 because it aligns with my experience building React frontend applications and my goal of improving my understanding of UI behavior and user experience. The issue has a clear problem description and expected behavior, making it a good opportunity to contribute while learning more about the project's frontend architecture.

I'm interested in this because:
1. I've built React interfaces where component state, layout, and user interactions are important for creating a smooth user experience.
2. The issue appears to be focused on a specific part of the UI, making it approachable while still requiring me to understand how the components work together.
3. It gives me a chance to learn how this project handles page scrolling, component positioning, and layout behavior.
4. Solving this issue will improve usability by making the attribute details easier to access without unnecessary scrolling.

From reading the issue description, I understand that the current problem is that the attribute details remain at the top of the page after an attribute is selected, even if the user has scrolled further down. My contribution will help make the details panel stay visible near the user's current position, resulting in a smoother and more intuitive experience.

---

## Understanding the Issue

### Problem Description

When a user clicks an attribute in the Attributes tab after scrolling down the page, the attribute details panel stays fixed at the top of the page. As a result, the selected attribute appears to have no details unless the user manually scrolls back up.

### Expected Behavior

The attribute details panel should follow the user's current scroll position so that, after selecting an attribute, the details remain visible without requiring the user to scroll back to the top of the page.

### Current Behavior

The details panel remains positioned at the top of the page regardless of the user's scroll position. If the user is far down the page, the details are outside the visible area, making it seem like nothing happened after clicking an attribute.

### Affected Components

Based on the issue description, the affected components are likely:
* The Attributes tab UI, where attributes are listed.
* The Attribute Details panel that displays information about the selected attribute.
* The layout or styling responsible for positioning the details panel during page scrolling (possibly using CSS positioning such as `sticky` or `fixed`, or the parent container's layout).

---

## Reproduction Process

### Environment Setup

Working branch: https://github.com/Cao1224/hawtio-react/tree/fix-issue-2014

#### Setup Steps

- Cloned the `hawtio-react` repository and installed project dependencies with `yarn install`.
- Started the frontend development server using `yarn start`.
- Set up a local Spring Boot application with a Jolokia endpoint to provide JMX data for testing.
- Verified the Jolokia endpoint was running using `curl` before connecting it to the Hawtio frontend.

#### Challenges & Solutions

- **Challenge:** Initially tried to run `mvn spring-boot:run` from the `hawtio-react` repository, which resulted in a Maven plugin error because the repository is a React/Yarn workspace rather than a Spring Boot project.
  - **Solution:** Started the frontend with `yarn start` and ran the Spring Boot application separately.

- **Challenge:** The latest Spring Boot example depended on unpublished `5.3-SNAPSHOT` artifacts, causing Maven dependency resolution failures.
  - **Solution:** Switched to a compatible Spring Boot example that uses released dependencies.

- **Challenge:** The frontend initially failed to connect to the backend due to an incorrect Jolokia endpoint configuration.
  - **Solution:** Verified the correct endpoint with `curl` and updated the connection settings to use `HTTP`, port `10001`, and `/actuator/jolokia`.

### Steps to Reproduce

1. Start the `hawtio-react` frontend and connect it to a Spring Boot application with a Jolokia endpoint.
2. Navigate to **JMX** and select an MBean with multiple attributes (e.g., `java.lang > Memory` or `java.lang > OperatingSystem`).
3. Open the **Attributes** tab.
4. Scroll down the page.
5. Click on an attribute near the bottom of the list.
6. **Observed Result:**
   - The attribute details panel remains at the top of the page instead of appearing within the current viewport.
   - Users must manually scroll back to the top to view the selected attribute's details, which makes the interface appear unresponsive when scrolled further down.
7. **Expected Result:**
   - The attribute details panel should appear within the user's current viewport after an attribute is selected.
   - Users should be able to view the selected attribute's details without needing to scroll back to the top of the page.

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/Cao1224/hawtio-react)
- **Screenshots/logs:** ![Issue #2014 Reproduction](issue2014-issue.gif)
- **My findings:**
  - The issue is reproducible on the latest `main` branch.
  - Clicking an attribute updates the selected attribute, but the details panel stays fixed at the top of the page instead of following the current scroll position.

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
