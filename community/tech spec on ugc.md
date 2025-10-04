Technical Specification: User-Generated Content Pipeline (Version 1.0)
Document Version: 1.3 (Revised)

Date: October 4, 2025

Author: Project Lead

1. Overview & Goal
This document specifies the end-to-end technical architecture for handling all user-generated content (UGC) on the platform. This includes comments on devlogs, replies in the forum, and comments on bug reports.

The primary goal is to create a robust, scalable, and secure system that:

Provides an instantaneous-feeling user experience for content creators.

Ensures the safety of the community by preventing harmful content from being shown publicly.

Gives administrators powerful, centralized tools for monitoring and moderation.

Allows users to manage their own content.

This specification adopts the Hybrid Moderation Approach, balancing user experience with proactive content safety.

2. Core Firestore Schema
2.1. The Generic "Content" Document
Every piece of user-generated content, regardless of where it is posted, will share a standardized data structure.

Fields:

content: (String) The body of the post, comment, or reply.

authorUid: (String) The Firebase Auth uid of the user who created the content. This is the primary key for ownership.

authorDisplayName: (String) The author's display name at the time of posting. Denormalized for efficient display.

createdAt: (Timestamp) The server timestamp when the document was created.

updatedAt: (Timestamp) Updated only if the content is edited by the author.

moderation: (Object) A nested object containing all moderation-related data.

status: (String) The current visibility state of the content. Must be one of:

under_review: The default state upon creation. Visible only to the author.

visible: Approved by the LLM or a human moderator. Visible to all users.

hidden_by_admin: Flagged by the LLM or a human moderator as inappropriate. Not visible to anyone except moderators/admins.

appealed_for_review: The author has requested a manual review after an automated rejection. Visible only to the author and moderators.

reviewedBy: (String, Optional) The uid of the admin/moderator who manually reviewed the content.

lastReviewAt: (Timestamp, Optional) The timestamp of the last manual review.

2.2. Contextual Placement (Subcollections)
This generic content document will be used within subcollections across the application to maintain context.

Devlog Comments: devlogPosts/{postId}/comments/{commentId}

Bug Report Comments: bugReports/{reportId}/comments/{commentId}

Forum Replies: forumTopics/{topicId}/replies/{replyId}

This structure allows for efficient querying of content within its specific context (e.g., fetching all comments for a single devlog post).

3. Content Lifecycle & Moderation Pipeline
This section details the step-by-step flow from post creation to public visibility.

Step 1: Client-Side Submission

A logged-in user types a message and clicks "Post."

The frontend application performs an immediate write to the relevant Firestore subcollection.

The document is created with the default moderation status: moderation: { status: "under_review" }.

Step 2: Instantaneous UI Feedback (Author & Public)
This step outlines the immediate UI changes for both the author and the public upon submission.

Author's Perspective (Optimistic Update):

The user who submitted the content sees their full post appear instantly in the UI.

The post is accompanied by a status tag displaying: AI moderator reviewing...

This full-content view is visible only to the author.

Public Perspective (Live Placeholder):

Simultaneously, for all other users, a generic placeholder appears. It will display a message like [authorDisplayName] is posting... but will not show the post's content.

This placeholder ensures the feed feels active without exposing un-reviewed content.

Step 3: Backend Trigger (Cloud Function - Thin Wrapper)

The creation of the new document in Firestore automatically triggers a background Cloud Function (onCreate). This function is intentionally minimal - just 5-10 lines of code that acts as a lightweight forwarding layer.

The Cloud Function's sole responsibility is to:
1. Extract the document data and metadata from the Firestore event
2. Forward this information to the Cloud Run container via an authenticated HTTP request
3. Return immediately (no waiting for response)

Step 4: Moderation Processing (Cloud Run Container)

The Cloud Run container receives the forwarded request and performs all the actual moderation work:

It calls the LLM API for safety analysis of the content.

It processes the LLM response and determines the final classification (e.g., "SAFE", "TOXIC").

It directly updates the original Firestore document with the moderation result.

Step 5: Firestore Document Update (by Cloud Run)

The Cloud Run service updates the original document in Firestore based on the LLM classification:

If "SAFE", it sets moderation.status to "visible".

If "TOXIC" (or any other unsafe category), it sets moderation.status to "hidden_by_admin".

Step 6: Real-Time Visibility Update
When the backend function updates the post's status, real-time listeners in all clients trigger a UI update based on the outcome.

Outcome 1: Moderation Pass (status -> "visible")

For the author: The AI moderator reviewing... tag briefly changes to Pass for 1-2 seconds, then fades out, leaving the clean post.

For all other users: The [authorDisplayName] is posting... placeholder is replaced by the full, visible post content.

Outcome 2: Moderation Fail (status -> "hidden_by_admin")

For the author: The post content remains visible to them, but the status tag changes to a message and a button: Rejected by moderator. [Request Human Review]

For all other users: The [authorDisplayName] is posting... placeholder is removed entirely from their view.

4. Manual Reporting & Review
This section covers all manual intervention flows, both from users reporting others and authors appealing a decision.

4.1. contentReports Collection Schema
A top-level collection to queue manual reviews.

contentReports/{reportId}

contentPath: (String) The full Firestore path to the reported document.

contentPreview: (String) A snippet of the reported content.

contentAuthorUid: (String) The UID of the content's author.

reporterUid: (String) The UID of the user filing the report or appealing.

reportType: (String) The origin of the report. Must be one of:

user_report: A user reported another user's content.

author_appeal: An author is appealing an automated rejection of their own content.

reason: (String) A category for the report (e.g., "Spam", "Harassment").

status: (String) The state of the report ("open", "resolved_action_taken", "resolved_no_action").

createdAt: (Timestamp)

resolvedBy: (String, Optional) The uid of the admin/moderator who resolved the report.

resolvedAt: (Timestamp, Optional) The timestamp when the report was resolved.

4.2. Manual Review Flow (User Reporting)
A user clicks a "Report" button on a piece of content.

The application creates a new document in the contentReports collection with reportType: "user_report".

An administrator, using a dedicated dashboard, views all documents where status == "open".

The dashboard provides a link to the content (using the contentPath) and tools for the admin to take action.

The admin can change the moderation.status of the original content document directly (e.g., from "visible" to "hidden_by_admin").

After taking action, the admin updates the report document in contentReports to record the resolution. This includes setting the status to "resolved_action_taken", adding their uid to resolvedBy, and setting the resolvedAt timestamp.

4.3. Author Appeal Flow
After an author's post is rejected by the LLM, they are presented with a "Request Human Review" button.

Clicking this button performs two actions:
a. Updates the original content document's moderation.status to appealed_for_review.
b. Creates a new document in the contentReports collection with reportType: "author_appeal".

This appeal enters the same moderation queue in the admin dashboard as user reports.

An administrator reviews the content and can override the LLM's decision by manually changing the moderation.status of the original content document to "visible" or confirming the rejection by leaving it as "hidden_by_admin".

After making a decision on the content, the administrator updates the corresponding document in the contentReports collection to record their action. This includes:
a. Setting the status to "resolved_action_taken".
b. Adding their uid to the resolvedBy field.
c. Adding a resolvedAt timestamp.

5. Architectural Decisions & Rationale
This section documents the reasoning behind the key technical choices in this architecture.

5.1. The Hybrid Moderation Approach
This approach was chosen as it provides the optimal balance between user experience and community safety.

Pure Pre-Moderation (blocking): Making the user wait for the LLM review to complete before their post is even accepted would result in a slow, frustrating user experience (2-5 seconds of loading).

Pure Post-Moderation (reactive): Making content instantly public to everyone creates a window of vulnerability where harmful content can be seen before it's removed.

The Hybrid Approach (our choice): Gives the author the instant gratification of seeing their post ("optimistic update") while ensuring no other user sees the content until it has been vetted by the automated system. This provides the best of both worlds.

5.2. Cloud Function as a Minimal Trigger Adapter
Cloud Functions serve as ultra-lightweight adapters (5-10 lines each) that simply forward Firestore events to Cloud Run. This thin wrapper pattern provides several key benefits:

Event-Driven Reliability: The Firestore trigger mechanism guarantees execution in response to database writes, even if the user closes their app. This decouples the moderation process from the user's session.

Security Layer: The Cloud Run service endpoint is not exposed to the public internet. It only accepts authenticated requests from our trusted Cloud Functions, preventing denial-of-service attacks and malicious probing.

Development Velocity: All actual business logic lives in the Cloud Run container within the main monorepo. Developers can work in VS Code with full debugging, hot reload, and version control without dealing with the Cloud Functions deployment cycle.

Clean Architecture: The function's role is purely that of a "trigger adapter" - it translates Firestore events into HTTP requests. This is a simple, maintainable pattern with minimal code to maintain.

5.3. Cloud Run for Core Moderation Logic
The decision to place the LLM interaction inside a Cloud Run container instead of the Cloud Function itself was deliberate.

Flexibility & Dependencies: Cloud Run provides a full container environment, allowing for custom system dependencies, specific library versions, and larger binaries that might be required by advanced LLM SDKs.

Timeout Control: LLM calls can sometimes be slow. Cloud Run offers much longer request timeout limits (up to 60 minutes) compared to Cloud Functions (max 9 minutes), preventing the process from failing on complex moderation tasks.

Cost & Performance: For consistent traffic, a provisioned Cloud Run instance can be more cost-effective and have lower "cold start" latency than a Cloud Function. It's better suited for a service that is expected to be called frequently.