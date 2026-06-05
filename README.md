**Contribution Number:** 1

**Student:** Sebastian Ramirez Vega

**Issue:** https://github.com/Yoast/wordpress-seo/issues/8502

**Status:** Phase II - Complete

---

## Why I Chose This Issue
I chose Yoast WordPress SEO issue #8502 because it is a clear user-facing dashboard issue involving how the plugin behaves when the Yoast blog feed cannot be loaded. The issue appeared well-scoped, involved a specific component of the application, and was labeled as a good first issue.

This issue seems like a good fit for AI301 because it should let me practice the full open source workflow: understanding an unfamiliar codebase, setting up a local development environment, reproducing the issue, finding the dashboard/widget feed logic, making a small focused change, and testing that the user-facing behavior works correctly. I chose it because the scope appears bounded and the issue is labeled as a good first issue.

---

## Understanding the Issue

### Problem Description

This issue concerns the Yoast SEO dashboard widget that displays recent blog posts from Yoast.com. Historically, when the widget could not load the blog feed because the request was blocked or failed, the dashboard could still display the feed heading or related content in an undesirable state.

A maintainer later changed the intended behavior: when the feed cannot be loaded, the blog feed section should not be displayed at all. My Phase II investigation focused on determining whether this issue can still be reproduced in the current codebase.

### Expected Behavior

When the dashboard widget cannot retrieve the blog feed from Yoast.com, the blog feed section should not be displayed. Users should continue to see the rest of the Yoast SEO Posts Overview widget without any broken or partially rendered feed content.

### Current Behavior

During local testing, the dashboard widget successfully loaded and displayed the "Latest blog posts on Yoast.com" section when the feed request succeeded.

After intentionally blocking requests to the Yoast feed endpoint and refreshing the dashboard, the blog feed section disappeared completely while the rest of the Yoast SEO Posts Overview widget continued to function normally.

Based on this investigation, the current behavior appears to match the intended behavior described by maintainers in the issue discussion.

### Affected Components

The affected components appear to be:

- `admin/class-yoast-dashboard-widget.php` — registers the WordPress dashboard widget, renders the empty widget container, and localizes text such as `feed_header` and `feed_footer`.
- `packages/js/src/dashboard-widget.js` — renders the React dashboard widget, calls `getPostFeed()` to retrieve posts from `https://yoast.com/feed/widget/`, and conditionally renders the Yoast blog feed section only when feed data is available.

---

## Reproduction Process

### Environment Setup

Set up a local WordPress development environment using LocalWP on Windows and created a WordPress site for testing Yoast SEO.

Forked and cloned the Yoast SEO repository from GitHub, then installed the required development dependencies. The project required Node.js 20, Composer, Yarn, Grunt CLI, PHP 8.2, and WordPress.

Challenges encountered during setup:

* Composer installation initially failed on Windows because the build scripts use the Unix `rm` command. This was resolved by making Git Bash's Unix tools available in the shell PATH so Composer could complete successfully.
* The initial `grunt build:dev` failed because Windows path length limitations prevented some files from being processed. This was resolved by enabling long path support in Windows and rebuilding the project.
* Yarn dependency installation experienced several network timeouts and required retries before completing successfully.

After resolving these issues, I successfully ran `composer install`, `yarn`, and `grunt build:dev`, activated the Yoast SEO plugin in WordPress, and confirmed that the dashboard widget was functioning in the local environment.

### Steps to Reproduce

1. Install and activate the Yoast SEO plugin in a local WordPress environment.
2. Navigate to the WordPress Dashboard page (`/wp-admin/index.php`).
3. Verify that the "Yoast SEO Posts Overview" widget is visible and that the "Latest blog posts on Yoast.com" feed section is displayed.
4. Open the browser Developer Tools and locate the request to:
   `https://yoast.com/feed/widget/...`
5. Block the feed request using the browser's network request blocking tools.
6. Refresh the WordPress Dashboard page.
7. Observe the contents of the "Yoast SEO Posts Overview" widget.

**Expected Behavior:** When the feed request cannot be loaded, the blog feed section should not be displayed.

**Actual Behavior Observed:** After blocking the feed request and refreshing the dashboard, the "Latest blog posts on Yoast.com" section disappeared completely while the rest of the Yoast SEO Posts Overview widget continued to function normally.


### Reproduction Evidence

* **Branch used for investigation:** https://github.com/sebramvega/wordpress-seo/tree/issue-8502-investigation

* **Screenshots/logs:** Captured screenshots of the dashboard widget in both scenarios:

  * Feed request succeeds: "Latest blog posts on Yoast.com" section is displayed.
  * Feed request blocked: blog feed section disappears while the SEO statistics remain visible.

* **My findings:** During investigation, I located the dashboard widget implementation in:

  * `admin/class-yoast-dashboard-widget.php`
  * `packages/js/src/dashboard-widget.js`

  The dashboard widget requests blog posts from:

  `https://yoast.com/feed/widget/`

  When the request succeeds, the blog feed is displayed normally.

  When the request is blocked, the feed data remains null and the React component does not render the blog feed section. This behavior appears consistent with the maintainer's 2019 change of plans and a 2024 maintainer comment indicating that the heading should no longer be displayed when the feed cannot be loaded.


---

## Solution Approach

### Analysis

My initial assumption was that the dashboard widget still displayed the "Latest blog posts on Yoast.com" heading when the feed request failed. However, after investigating the current implementation and reproducing both successful and failed feed requests locally, I found that the behavior appears to have already been changed.

The dashboard widget fetches feed data in:

`packages/js/src/dashboard-widget.js`

If the feed request succeeds, the feed data is stored in the component state and the blog feed is rendered.

If the feed request fails, the feed data remains `null`. The widget then returns `null` for the blog feed component, causing the entire feed section (including the heading) to be hidden.

This behavior appears consistent with the maintainer's 2019 change of plans and the follow-up maintainer comment from 2024 indicating that the heading no longer appears when feed requests are blocked.

### Proposed Solution

Before implementing any code changes, I plan to confirm whether the issue is still considered unresolved by the Yoast maintainers.

My investigation suggests that the current implementation already matches the intended behavior described in the issue discussion. If maintainers confirm that the issue has effectively been resolved, I will document my findings and select a new issue for contribution.

If maintainers identify a remaining edge case where the feed section is still displayed incorrectly when feed loading fails, I will reproduce that specific scenario, identify the relevant code path, and implement a focused fix that follows the project's existing patterns.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The issue originally described a problem where the Yoast dashboard blog feed did not fail gracefully when the feed from Yoast.com could not be loaded. However, the issue discussion later changed the desired behavior: instead of showing an error message, the blog feed section should be hidden when the feed request fails.

**Match:** The current implementation in `packages/js/src/dashboard-widget.js` already follows this pattern. The widget only renders the `WordpressFeed` component if `this.state.feed` is not `null`. When `getPostFeed()` fails, the error is logged and `feed` remains `null`, so the blog feed section is not rendered.

**Plan:**
1. Document my local reproduction findings in the Contribution README.
2. Comment on the GitHub issue with the reproduction results and ask whether there is still an unresolved scenario maintainers want addressed.
3. If maintainers confirm the issue is already resolved, select a new issue for the build phase.
4. If maintainers identify a remaining failing scenario, reproduce that specific case and implement a focused fix in `packages/js/src/dashboard-widget.js`.

**Implement:** Investigation branch: https://github.com/sebramvega/wordpress-seo/tree/issue-8502-investigation

**Review:** Before making any code changes, I will confirm with maintainers whether the issue still needs implementation work. If a code change is needed, I will review Yoast's contribution guidelines and keep the change scoped to the dashboard widget/feed-loading behavior.

**Evaluate:** I verified the current behavior manually by testing the dashboard widget with the Yoast feed request allowed and then blocked. If further work is needed, I will validate by repeating those steps and running the relevant build/test commands before opening a pull request.

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
