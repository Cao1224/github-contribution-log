# Contribution 1: [Feature request] Image zooming #701


**Contribution Number:** 1

**Student:** Yuanncheng Cao

**Issue:** [GitHub issue link](https://github.com/session-foundation/session-desktop/issues/701)

**Status:** Phase IV Complete

---

## Why I Chose This Issue

I chose issue #701 because it is a user-facing feature request that improves the usability of Session Desktop when viewing image attachments. Many users share screenshots containing text, and the current image viewer can make that text difficult to read without downloading the image and opening it in another application. Adding Zoom functionality would provide a smoother and more intuitive user experience.

This issue aligns well with my experience building frontend applications using React and TypeScript. The maintainer also mentioned that the work should be done in the Lightbox component, which provides a clear starting point and makes the scope manageable for a first open-source contribution.

I'm interested in this issue because:
1. I have experience working with React component state, event handling, and user interface interactions.
2. The feature is self-contained within the image viewing experience, making it a good opportunity to learn the Session Desktop codebase.
3. It directly impacts user experience and solves a practical problem that users have reported.

From reading the issue discussion, I understand that the current behavior automatically scales images to fit the window, which can make text in screenshots difficult to read. The proposed solution is to add zoom functionality within the Lightbox, allowing users to inspect images at full size without leaving the application.

Through this contribution, I hope to learn more about Session Desktop's frontend architecture, image rendering workflow, and the process of collaborating with maintainers through code reviews and pull requests.

## Understanding the Issue

### Problem Description

Currently, images opened in Session are automatically resized to fit the Lightbox window. This works well for most images, but images containing text can become difficult to read because the displayed size is too small. Users often have to maximize the application window or download the image and open it in an external viewer to inspect the content.

### Expected Behavior

Users should be able to zoom in when viewing an image in the Lightbox. Clicking the image could toggle between "fit to screen" and "actual size" modes, allowing users to inspect screenshots and text-heavy images without leaving the application. The image should return to the original fit-to-screen view when clicked again.

### Current Behavior

Images displayed in the Lightbox are always scaled to fit the current window size. Clicking an open image has no effect, and there is no built-in zoom capability. As a result, text inside screenshots or other detailed images may be unreadable unless the user enlarges the window or opens the image externally.

### Affected Components

The issue primarily affects the Lightbox component, which is responsible for displaying images and attachments in an enlarged view. The Zoom interaction and image scaling behavior will likely need to be implemented within this component.

---

## Reproduction Process

### Environment Setup

I set up the Session Desktop development environment on macOS. During the setup process, I encountered several issues with missing dependencies and version mismatches.

* Installed `pnpm` and enabled it through Corepack to manage project dependencies.
* Installed `CMake`, which was required to build native modules such as `libsession_util_nodejs`. The installation initially failed because of a Homebrew permission issue related to Docker, which I resolved by fixing the directory ownership.
* Installed and configured `nvm` to manage Node.js versions and ensure compatibility with the project's required Node version.
Verified that Xcode Command Line Tools were installed and rebuilt the dependencies successfully.

After resolving these issues, I was able to run `pnpm install`, build the project successfully, and launch multiple Session instances using separate profiles.

### Steps to Reproduce

1. Create two Session accounts and establish a conversation between them. I used two accounts because image attachments are only available after the recipient accepts the conversation request. 
2. 3. Send an image (for example, a screenshot containing text) from one account to the other. When the recipient receives the first image, it displays **"Click to download media"** and prompts:
```text
Would you like to automatically download all files from [contact]?
```
Select **Yes** so that the image can be downloaded and viewed.

3. Open the image in the Lightbox view. The interface only provides two controls: a close button and a download button. The image is scaled to fit the window, and there is currently no way to zoom in using the mouse or by clicking on the image.
4. Images are always resized to fit the current window. Screenshots containing text can be difficult to read, and users cannot zoom in within the application. To view the image at full size, they must download it and open it in an external image viewer.

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork](https://github.com/Cao1224/session-desktop/commit/cc27ea28af72f8e941da5a69af051b02dcc3172a)]
- **Screenshots/logs:**
  <img width="861" height="560" alt="Screenshot 2026-06-14 at 2 01 48 PM" src="https://github.com/user-attachments/assets/29eb62b0-21d1-4f4d-ad32-8dab3ffb6397" />

- **My findings:** The image displayed in the Lightbox is automatically scaled to fit the current window size. This behavior makes screenshots or images containing text difficult to read. Users currently have to maximize the application window or download the image and open it in an external viewer to inspect the image at full size. Based on feedback from a collaborator, the functionality should likely be implemented in the **Lightbox component**.

---

## Solution Approach

### Analysis

The issue is caused by the current Lightbox implementation always constraining images with:

* `objectFit: 'contain'`
* `maxWidth: '80vw'`
* `maxHeight: '80vh'`

As a result, large screenshots and text-heavy images are scaled down and can become difficult to read.

A naive attempt to remove the size limits (`maxWidth: none, maxHeight: none`) causes another problem: the image exceeds the viewport and obscures the Lightbox controls, making the interface difficult to use.

In addition, the `<img>` element does not implement any interaction handlers, so users cannot zoom or pan the image. The existing code explicitly prevents clicks on the image from closing the Lightbox:
```ts
const onContainerClick = (event: MouseEvent<HTMLDivElement>) => {
  if (renderedRef && event.target === renderedRef.current) {
    return;
  }

  handleClose();
};
```
This means image clicks are intentionally ignored and currently perform no action.

### Proposed Solution

Introduce a Zoom state inside the Lightbox component.

By default, images remain in the existing "fit-to-screen" mode. Clicking the image toggles between:

1. **Fit-to-screen mode**
   * Existing behavior.
   * Image constrained to 80% of the viewport.
2. **Zoomed mode**
   * Display the image at its actual size.
   * Allow the image container to scroll so that oversized images do not cover the close and download buttons.
   * Clicking the image again returns to fit-to-screen mode.

This approach preserves the current UI while providing a simple way to inspect screenshots without downloading them.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Images containing text are difficult to read because the Lightbox always scales them to fit the viewport. Users currently have no way to inspect an image at full resolution without saving it externally.

**Match:** 

The existing Lightbox already:
* distinguishes clicks on the image from clicks on the background;
* keeps the close and download buttons outside the image element;
* uses React state and props, making it straightforward to add a zoom mode.

**Plan:** 
1. Modify `LightboxObject` to support a zoom state.
2. Add an `onClick` handler to the `<img>` element.
3. Preserve the current 80vw × 80vh fit-to-screen behavior by default.
4. Update the image container to support scrolling when the image exceeds the viewport.
5. Ensure the close and download buttons remain visible.
6. Verify that videos and unsupported attachment types are unaffected.
7. Add or update tests if similar Lightbox tests already exist.

**Implement:** [[Link to your branch/commits as you work](https://github.com/Cao1224/session-desktop/commit/cc27ea28af72f8e941da5a69af051b02dcc3172a)]
* Modify `Lightbox.tsx`.
* Add a local `isZoomed` state.
* Add image click handling.
* Adjust container overflow behavior.

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]
- [x] Preserve existing behavior by default.
- [x] Do not affect video attachments.
- [x] Keep close and download buttons accessible.
- [x] Avoid introducing breaking UI changes.
- [x] Follow existing React and TypeScript patterns.

**Evaluate:** [How will you verify it works?]

Verify that:
* screenshots with text can be enlarged;
* clicking toggles between zoomed and fit-to-screen modes;
* large images remain navigable;
* close and download buttons stay visible;
* video attachments continue to behave normally;
* clicking outside the image still closes the Lightbox.

---

## Testing Strategy

### Unit Tests

- [x] `scale` clamps between `MIN_SCALE` (1) and `MAX_SCALE` (10) when wheel delta is large.
- [x] `translate` resets to `{x: 0, y:0} when scale returns to 1`
- [x] `isZoomed` state resets when `objectURL` prop changes

### Integration Tests

- [x] Pinch/Ctrl+scroll zooms toward pointer position, not center
- [x] Navigating to next/previous image resets zoom and pan

### Manual Testing

- [x] Mac trackpad — pinch to zoom in and out, image stays under fingers
- [x] Ctrl+scroll — zooms in/out on Windows and Mac
- [x] Drag while zoomed — pans the image, does not close the lightbox
- [x] Double-click — resets to fit-to-screen from any zoom level
- [x] Click dark background — closes the lightbox at any zoom level
- [x] Click image (no drag) — does not close the lightbox
- [x] Navigate prev/next — zoom resets on each new image
- [x] Video attachments — unaffected, no zoom UI appears
- [x] Unsupported file types — unaffected

---

## Implementation Notes

### Week 3 Progress

**What I built:**
- Added pinch-to-zoom and drag-to-pan to the Lightbox image viewer.
- Users can now pinch or Ctrl+scroll to zoom toward their pointers, drag to pan while zoomed, and double-click to rest.
- Zoom and pan also rest automatically when navigating between images.

**Challenges faced:**
- The main challenge was the zoom containers div collapsing to 0x0.
- The original approach used `position: absolute; inset: 0` while requiring a sized parent, but `objectContainer` is `inline-flex` and sizes to its content, so removing the image from the flow collapsed it.
- Solved by dropping absolute positioning entirely and letting the wrapper size naturally to the image, with `overflow: hidden` to clip the zoomed content.
- Also had to be careful with click event propagation; the zoom container was swallowing all clicks, including background clicks that should close the lightbox. Fixed with a `didDrag` ref to distinguish a pan gesture from a plain click.

**Decisions made:**
- Use CSS `transform: scale()` instead of a scrollable container, no scrollbar flash, background stays intact, no mode switching.
- Zoom toward the pointer rather than the center, matching native image viewer behavior
- No new UI chrome — kept the interaction implicit (cursor changes only)

### Code Changes

- **Files modified:** `Lightbox.tsx`
- **Key commits:**
  - [e906028](https://github.com/Cao1224/session-desktop/commit/e906028f4082668a172ffb27cbbd264626f2ae68) — fix: support pinch-to-zoom and drag-to-pan in lightbox
- **Approach decisions:** 
  - Kept the wrapper div in normal flow instead of `position: absolute` — lets it size naturally to the image, the same way the original `<img>` did, avoiding the 0×0 collapse
  - Covered Mac trackpad (pinch fires `wheel` with `ctrlKey`) and Windows (Ctrl+scroll, same event) with a single handler rather than separate touch/gesture APIs
  - Used a `didDrag` ref to distinguish a pan gesture from a plain click, so background-click-to-close still works correctly while zoomed
 
---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review

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
