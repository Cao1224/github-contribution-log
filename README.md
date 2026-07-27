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

The root cause is the nested scrolling behavior in the JMX content layout.

`JmxContent.tsx` renders the main content inside a `PageSection` with the `hasOverflowScroll` prop, which creates an internal scroll container (`overflow: auto`). This results in two independent scroll containers: the page scroll and the JMX content scroll.

When an attribute is selected, the details drawer is rendered within the inner scroll container. If the page has already been scrolled, the details panel stays positioned relative to the internal scroll container instead of following the page scroll, which matches the behavior described in the issue.

Removing `hasOverflowScroll` eliminates the nested scroll container and allows the page to use a single scroll context, so the details view naturally follows the page scroll.

### Proposed Solution

Remove the `hasOverflowScroll` prop from the `PageSection` in `packages/hawtio/src/plugins/jmx/JmxContent.tsx`.

This removes the unnecessary nested scroll container and allows the JMX page to rely on the browser's page scroll. As a result, the attribute details drawer follows the page scroll naturally without modifying the drawer or attribute selection logic.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The Attributes tab currently creates two independent scroll containers. When a user scrolls down the page and selects an attribute, the details view remains tied to the inner scroll container instead of following the page scroll, making it cumbersome to view the attribute details.

**Match:** During investigation, the nested scrolling was traced to the `hasOverflowScroll` prop on the `PageSection` in `JmxContent.tsx`. Neither `Attributes.tsx`, `AttributeModal.tsx`, nor the `Drawer` component explicitly creates the nested scrolling behavior. Removing `hasOverflowScroll` resolves the issue while keeping the existing drawer implementation unchanged.

**Plan:** [Step-by-step implementation plan]
1. Modify `packages/hawtio/src/plugins/jmx/JmxContent.tsx`.
2. Remove the `hasOverflowScroll` prop from the `PageSection` with id `jmx-content-main`.
3. Verify that:
   - The Attributes tab uses a single-page scroll.
   - The details drawer follows the page scroll after selecting an attribute.
   - The Operations and Chart tabs continue to render and scroll correctly.
4. Run the existing test suite to ensure no regressions are introduced.

**Implement:** [[Link to your branch/commits as you work]](https://github.com/Cao1224/hawtio-react/tree/fix-issue-2014)
- Create a feature branch.
- Remove the `hasOverflowScroll` prop from `JmxContent.tsx`.
- Test the change locally across the JMX plugin.
- Commit the change and open a pull request.


**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]
- [x] The fix is minimal and only addresses the reported issue.
- [x] No unnecessary changes were made to `Attributes.tsx`, `AttributeModal.tsx`, or the `Drawer` components.
- [x] The Operations and Chart tabs continue to function correctly.
- [x] The implementation follows the project's coding style and contribution guidelines.

**Evaluate:** [How will you verify it works?]

Verify the fix by:

1. Launching the application and navigating to the JMX plugin.
2. Opening the **Attributes** tab.
3. Scrolling down the page.
4. Selecting an attribute near the bottom of the list.
5. Confirming that the details view follows the page scroll without requiring the user to scroll back to the top.
6. Verifying that the **Operations** and **Chart** tabs continue to behave correctly after the change.
---

## Testing Strategy

### Unit Tests

- [x] Verify `JmxContent` renders correctly after removing `hasOverflowScroll`.
- [x] Verify the Attributes tab still renders the attribute table correctly.
- [x] Verify the Operations and Chart tabs continue to render without layout regressions.

### Integration Tests

- [x] Navigate through the JMX plugin and verify all tabs function correctly.
- [x] Select an attribute after scrolling and confirm the details view follows the page scroll.

### Manual Testing

- Reproduced the issue using the provided Spring Boot sample application.
- Inspected the page layout using Chrome DevTools and identified nested scroll containers.
- Verified that the inner scroll container was introduced by `PageSection` with the `hasOverflowScroll` prop.
- Temporarily removed `hasOverflowScroll` and confirmed the nested scroll disappeared.
- Confirmed that the attribute details view now follows the page scroll as described in the issue.

---

## Implementation Notes

### Week 2 Progress

- Set up the Hawtio development environment and connected it to a Spring Boot application with Jolokia.
- Reproduced the issue consistently on the Attributes tab.
- Investigated `Attributes.tsx`, `AttributeModal.tsx`, and the drawer implementation to understand how the details panel is rendered.
- Explored the layout hierarchy and identified that the page contained both an outer page scroll and an inner content scroll.

### Week 3 Progress

- Investigated the JMX page layout and traced the nested scrolling behavior to `PageSection` in `JmxContent.tsx`.
- Used Chrome DevTools to inspect the generated DOM and confirmed that `hasOverflowScroll` creates an additional scroll container.
- Removed the `hasOverflowScroll` prop as a proof of concept and verified that the nested scroll was eliminated.
- Confirmed that the attribute details view now follows the page scroll, matching the expected behavior described in the issue.
- Planned to perform regression testing on the Operations and Chart tabs before submitting the final PR.

### Code Changes

- **Files modified:**
  - `packages/hawtio/src/plugins/jmx/JmxContent.tsx`
    - Removed the `hasOverflowScroll` prop from the `PageSection` with id `jmx-content-main`.
- **Key commits:** [Links to important commits]
  - [(1d5212e)](https://github.com/Cao1224/hawtio-react/commit/1d5212eff178e9b196eeab2ebeb927bc62717953) Remove `hasOverflowScroll` from `JmxContent.tsx` to eliminate the nested scroll container
- **Approach decisions:**
  - Initially investigated `Attributes.tsx`, `AttributeModal.tsx`, and the `Drawer` component since the issue appeared when selecting an attribute.
  - Used Chrome DevTools to inspect the generated DOM and trace the source of the nested scrolling behavior.
  - Determined that the nested scroll container originated from the `hasOverflowScroll` prop on `PageSection` rather than the drawer or attribute selection logic.
  - Chose to remove `hasOverflowScroll` because it is a minimal change that addresses the root cause instead of introducing additional scrolling logic or modifying the drawer implementation.
  - Verified that removing `hasOverflowScroll` resolves the issue by allowing the page to use a single scroll context, causing the details view to follow the page scroll as expected.

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
