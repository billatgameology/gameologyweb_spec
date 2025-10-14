# User-Generated Content Pipeline (Version 1.0)

**Document Version:** 1.3 (Revised)  
**Date:** October 4, 2025  
**Author:** Project Lead

---

## 1. Overview & Goal
This document specifies the end-to-end technical architecture for handling all user-generated content (UGC) on the platform. This includes comments on devlogs, replies in the forum, and comments on bug reports.

The primary goal is to create a robust, scalable, and secure system that:

- Provides an instantaneous-feeling user experience for content creators.
- Ensures the safety of the community by preventing harmful content from being shown publicly.
- Gives administrators powerful, centralized tools for monitoring and moderation.
- Allows users to manage their own content.

This specification adopts the **Hybrid Moderation Approach**, balancing user experience with proactive content safety.

## 2. Core Firestore Schema

### 2.1. The Generic "Content" Document

Every piece of user-generated content, regardless of where it is posted, will share a standardized data structure.

**Fields:**

- **content:** (String) The body of the post, comment, or reply.
- **authorUid:** (String) The Firebase Auth uid of the user who created the content. This is the primary key for ownership.
- **authorDisplayName:** (String) The author's display name at the time of posting. Denormalized for efficient display.
- **createdAt:** (Timestamp) The server timestamp when the document was created.
- **updatedAt:** (Timestamp) Updated only if the content is edited by the author.
- **moderation:** (Object) A nested object containing all moderation-related data.
  - **status:** (String) The current visibility state of the content. Must be one of:
    - `under_review`: The default state upon creation. Visible only to the author.
    - `visible`: Approved by the LLM or a human moderator. Visible to all users.
    - `hidden_by_admin`: Flagged by the LLM or a human moderator as inappropriate. Not visible to anyone except moderators/admins.
    - `appealed_for_llm_review`: The author has requested a second-tier LLM review with higher reasoning capability.
    - `appealed_for_human_review`: The author has requested human review after failing the reasoning LLM check.
  - **initialReviewModel:** (String, Optional) The model used for initial moderation (e.g., "gpt-4o-mini").
  - **initialReviewResult:** (String, Optional) The result from initial review ("safe" or "unsafe").
  - **reasoningReviewModel:** (String, Optional) The reasoning model used for appeal (e.g., "o1-preview", "claude-3.5-sonnet").
  - **reasoningReviewResult:** (String, Optional) The result from reasoning LLM review ("safe" or "unsafe").
  - **violatedGuideline:** (String, Optional) Which community guideline the content violates (e.g., "Personal Attack", "Restricted Content").
  - **rejectionReason:** (String, Optional) Detailed explanation of why the content was rejected.
  - **reviewedBy:** (String, Optional) The uid of the admin/moderator who manually reviewed the content.
  - **lastReviewAt:** (Timestamp, Optional) The timestamp of the last manual review.

### 2.2. Contextual Placement (Subcollections)

This generic content document will be used within subcollections across the application to maintain context.

- **Devlog Comments:** `devlogPosts/{postId}/comments/{commentId}`
- **Forum Replies:** `forumTopics/{topicId}/replies/{replyId}`

This structure allows for efficient querying of content within its specific context (e.g., fetching all comments for a single devlog post).

## 3. Content Lifecycle & Moderation Pipeline

This section details the step-by-step flow from post creation to public visibility.

### Step 1: Client-Side Submission

- A logged-in user types a message and clicks "Post."
- The frontend application performs an immediate write to the relevant Firestore subcollection.
- The document is created with the default moderation status: `moderation: { status: "under_review" }`.

### Step 2: Instantaneous UI Feedback (Author & Public)

This step outlines the immediate UI changes for both the author and the public upon submission.

**Author's Perspective (Optimistic Update):**
- The user who submitted the content sees their full post appear instantly in the UI.
- The post is accompanied by a status tag displaying: `AI moderator reviewing...`
- This full-content view is visible only to the author.

**Public Perspective (Live Placeholder):**
- Simultaneously, for all other users, a generic placeholder appears. It will display a message like `[authorDisplayName] is posting...` but will not show the post's content.
- This placeholder ensures the feed feels active without exposing un-reviewed content.

### Step 3: Backend Trigger (Cloud Function - Thin Wrapper)

- The creation of the new document in Firestore automatically triggers a background Cloud Function (`onCreate`). This function is intentionally minimal - just 5-10 lines of code that acts as a lightweight forwarding layer.
- The Cloud Function's sole responsibility is to:
  1. Extract the document data and metadata from the Firestore event
  2. Forward this information to the Cloud Run container via an authenticated HTTP request
  3. Return immediately (no waiting for response)

### Step 4: Moderation Processing (Cloud Run Container)

The Cloud Run container receives the forwarded request and performs all the actual moderation work:

- It calls the LLM API for safety analysis of the content.
- Using community rules only, it processes the LLM response and determines the final classification (e.g., "Safe", "Unsafe").
- It directly updates the original Firestore document with the moderation result.

### Step 5: Firestore Document Update (by Cloud Run)

The Cloud Run service updates the original document in Firestore based on the LLM classification:

- If "Safe", it sets `moderation.status` to `"visible"`.
- If "Unsafe" (or any other unsafe category), it sets `moderation.status` to `"hidden_by_admin"`.

### Step 6: Real-Time Visibility Update

When the backend function updates the post's status, real-time listeners in all clients trigger a UI update based on the outcome.

**Outcome 1: Moderation Pass (status -> "visible")**
- **For the author:** The `AI moderator reviewing...` tag briefly changes to `Pass` for 1-2 seconds, then fades out, leaving the clean post.
- **For all other users:** The `[authorDisplayName] is posting...` placeholder is replaced by the full, visible post content.

**Outcome 2: Moderation Fail (status -> "hidden_by_admin")**
- **For the author:** The post content remains visible to them, but the status tag changes to a message and a button: `Rejected by moderator. [Request AI Review]`
- **For all other users:** The `[authorDisplayName] is posting...` placeholder is removed entirely from their view.

## 4. Manual Reporting & Review

This section covers all manual intervention flows, both from users reporting others and authors appealing a decision.

### 4.1. contentReports Collection Schema

A top-level collection to queue manual reviews.

**`contentReports/{reportId}`**

- **contentPath:** (String) The full Firestore path to the reported document.
- **contentPreview:** (String) A snippet of the reported content.
- **contentAuthorUid:** (String) The UID of the content's author.
- **reporterUid:** (String) The UID of the user filing the report or appealing.
- **reportType:** (String) The origin of the report. Must be one of:
  - `user_report`: A user reported another user's content.
  - `author_appeal_llm`: An author is appealing to the reasoning LLM after initial rejection.
  - `author_appeal_human`: An author is appealing to human review after reasoning LLM rejection.
- **reason:** (String) A category for the report (e.g., "Spam", "Harassment").
- **status:** (String) The state of the report (`"open"`, `"resolved_action_taken"`, `"resolved_no_action"`).
- **createdAt:** (Timestamp)
- **resolvedBy:** (String, Optional) The uid of the admin/moderator who resolved the report.
- **resolvedAt:** (Timestamp, Optional) The timestamp when the report was resolved.

### 4.2. Manual Review Flow (User Reporting)

1. A user clicks a "Report" button on a piece of content.
2. The application creates a new document in the `contentReports` collection with `reportType: "user_report"`.
3. An administrator, using a dedicated dashboard, views all documents where `status == "open"`.
4. The dashboard provides a link to the content (using the `contentPath`) and tools for the admin to take action.
5. The admin can change the `moderation.status` of the original content document directly (e.g., from `"visible"` to `"hidden_by_admin"`).
6. After taking action, the admin updates the report document in `contentReports` to record the resolution. This includes setting the status to `"resolved_action_taken"`, adding their uid to `resolvedBy`, and setting the `resolvedAt` timestamp.

### 4.3. Author Appeal Flow (Two-Tier Review)

This flow minimizes human review by implementing a two-tier automated review system before escalating to human moderators.

#### 4.3.1. First Tier: Initial LLM Review (Fast Model)

- Content is reviewed by a fast, efficient LLM model (e.g., GPT-4o-mini)
- If rejected, the author sees: `Rejected by moderator. [Request AI Review]`

#### 4.3.2. Second Tier: Reasoning LLM Review

1. After an author's post is rejected by the initial LLM, they click the `[Request AI Review]` button.
2. This action triggers:
   - a. Updates the content document's `moderation.status` to `appealed_for_llm_review`
   - b. Creates a document in `contentReports` with `reportType: "author_appeal_llm"`
   - c. Triggers a Cloud Function that forwards the request to Cloud Run for reasoning LLM review
3. **Cloud Run Reasoning LLM Processing:**
   - Sends the content to a higher reasoning capability LLM (e.g., o1-preview, Claude 3.5 Sonnet)
   - The reasoning LLM evaluates against community guidelines and returns:
     - **Result:** `"safe"` or `"unsafe"`
     - **Violated Guideline:** (if unsafe) Which specific guideline was violated (e.g., "Personal Attack", "Restricted Content", "Dismissive Without Reason")
     - **Rejection Reason:** (if unsafe) Detailed explanation of the violation
4. **Reasoning LLM Pass (safe):**
   - Cloud Run updates the content document:
     - `moderation.status` → `"visible"`
     - `moderation.reasoningReviewModel` → model name
     - `moderation.reasoningReviewResult` → `"safe"`
   - **Author sees:** Status changes to `Approved on appeal` then fades out
   - **Public sees:** The full post content becomes visible
   - The `contentReports` document is updated: `status` → `"resolved_action_taken"`
5. **Reasoning LLM Fail (unsafe):**
   - Cloud Run updates the content document:
     - `moderation.status` → `"hidden_by_admin"`
     - `moderation.reasoningReviewModel` → model name
     - `moderation.reasoningReviewResult` → `"unsafe"`
     - `moderation.violatedGuideline` → specific guideline
     - `moderation.rejectionReason` → detailed explanation
   - **Author sees:** 
     - The violated guideline and rejection reason are displayed
     - A new button appears: `[Request Human Review]`
   - **Public sees:** The placeholder remains removed
   - The `contentReports` document remains with `status` → `"open"` for potential human escalation

#### 4.3.3. Third Tier: Human Review (Manual Escalation)

1. After failing the reasoning LLM review, the author can click `[Request Human Review]`.
2. This action:
   - a. Updates `moderation.status` to `appealed_for_human_review`
   - b. Creates a new document in `contentReports` with `reportType: "author_appeal_human"`
3. The appeal enters the admin moderation queue.
4. An administrator reviews the content and can:
   - Override the LLM decisions by changing `moderation.status` to `"visible"`
   - Confirm the rejection by leaving it as `"hidden_by_admin"`
5. After making a decision, the administrator updates the `contentReports` document:
   - Sets `status` to `"resolved_action_taken"` or `"resolved_no_action"`
   - Adds their uid to `resolvedBy`
   - Adds a `resolvedAt` timestamp

## 5. Architectural Decisions & Rationale

This section documents the reasoning behind the key technical choices in this architecture.

### 5.1. The Hybrid Moderation Approach

This approach was chosen as it provides the optimal balance between user experience and community safety.

- **Pure Pre-Moderation (blocking):** Making the user wait for the LLM review to complete before their post is even accepted would result in a slow, frustrating user experience (2-5 seconds of loading).
- **Pure Post-Moderation (reactive):** Making content instantly public to everyone creates a window of vulnerability where harmful content can be seen before it's removed.
- **The Hybrid Approach (our choice):** Gives the author the instant gratification of seeing their post ("optimistic update") while ensuring no other user sees the content until it has been vetted by the automated system. This provides the best of both worlds.

### 5.2. Cloud Function as a Minimal Trigger Adapter

Cloud Functions serve as ultra-lightweight adapters (5-10 lines each) that simply forward Firestore events to Cloud Run. This thin wrapper pattern provides several key benefits:

- **Event-Driven Reliability:** The Firestore trigger mechanism guarantees execution in response to database writes, even if the user closes their app. This decouples the moderation process from the user's session.
- **Security Layer:** The Cloud Run service endpoint is not exposed to the public internet. It only accepts authenticated requests from our trusted Cloud Functions, preventing denial-of-service attacks and malicious probing.
- **Development Velocity:** All actual business logic lives in the Cloud Run container within the main monorepo. Developers can work in VS Code with full debugging, hot reload, and version control without dealing with the Cloud Functions deployment cycle.
- **Clean Architecture:** The function's role is purely that of a "trigger adapter" - it translates Firestore events into HTTP requests. This is a simple, maintainable pattern with minimal code to maintain.

### 5.3. Cloud Run for Core Moderation Logic

The decision to place the LLM interaction inside a Cloud Run container instead of the Cloud Function itself was deliberate.

- **Flexibility & Dependencies:** Cloud Run provides a full container environment, allowing for custom system dependencies, specific library versions, and larger binaries that might be required by advanced LLM SDKs.
- **Timeout Control:** LLM calls can sometimes be slow. Cloud Run offers much longer request timeout limits (up to 60 minutes) compared to Cloud Functions (max 9 minutes), preventing the process from failing on complex moderation tasks.
- **Cost & Performance:** For consistent traffic, a provisioned Cloud Run instance can be more cost-effective and have lower "cold start" latency than a Cloud Function. It's better suited for a service that is expected to be called frequently.

### 5.4. Two-Tier LLM Review System

The two-tier LLM review system was designed to minimize human moderator workload while maintaining high accuracy.

- **Cost Optimization:** The first tier uses a fast, inexpensive model for initial screening. Only rejected posts that authors appeal require the more expensive reasoning model.
- **Reduced False Positives:** The reasoning LLM (second tier) has higher cognitive capability and can better understand context, nuance, and edge cases (e.g., distinguishing "You're the shit" from "You're shit").
- **Transparency:** By providing the specific violated guideline and detailed rejection reason from the reasoning LLM, authors understand exactly why their content was rejected, reducing frustration and unnecessary human appeals.
- **Human Review as Last Resort:** Most appeals are resolved by the reasoning LLM, reserving human moderator time for only the most complex or contested cases.
- **Progressive Cost Model:** 
  - Tier 1 (Fast LLM): ~$0.0001 per post → runs on every post
  - Tier 2 (Reasoning LLM): ~$0.01-0.05 per appeal → only runs when author appeals
  - Tier 3 (Human): Most expensive → only escalated after two automated reviews fail