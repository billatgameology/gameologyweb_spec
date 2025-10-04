Feature Specification: Devlogs & Comments (Version 1.0)
Document Version: 1.1 (Revised)

Date: October 3, 2025

Author: Project Lead

1. Overview & Goal
This document specifies the requirements for a new Devlog feature on the website. The primary goal is to provide official updates and content to the community and foster engagement by allowing authenticated users to comment on these posts.

This initial version (v1.0) focuses on the core functionality of an admin creating devlog posts and users viewing and commenting on them.

2. Key Terminology & Definitions
Devlog (Developer Log): A dedicated section of the website, managed by the site administrator, that serves as an official blog or news feed.

Devlog Post: A single, self-contained article or entry created by a site administrator. This is the core content artifact of the feature. It consists of a title, body content, author information, and timestamps.

Comment: A text-based response submitted by an authenticated user in relation to a specific Devlog Post.

Creator/Author: Refers exclusively to the Site Administrator(s) who have the ability to create and manage Devlog Posts.

3. Core Functions & Scope
The following functions are considered IN SCOPE for Version 1.0 of this feature.

3.1. Creating & Managing a Devlog Post (Admin Function)
Ownership: Only Site Administrators can create, edit, and delete devlog posts.

Creation is done in code, nothing required for web feature

3.2. Viewing Devlog Posts & Comments
Devlog Feed View:

A central page (/devlog) will display all published posts in reverse chronological order.

Each entry in the feed will be a "card" displaying the post's Title, Author, a content snippet, and the publication date.

Clicking a card navigates to the Single Post View.

Single Post View:

A dedicated page for viewing a post in its entirety.

Displays the full Title and Content Body.

Below the post content, there will be a Comment Section that displays all comments for that post chronologically.

Authenticated users will see a form to submit a new comment.

3.3. User Commenting
Action: An authenticated user can post comments on any Devlog Post.

Content: Comments are plain text.

Display: Each comment will display the user's Display Name, the comment text, and a timestamp.

Comment Visibility & Safety:

Comments go through an automated safety review before appearing publicly.

Authors see their comments immediately with a review status indicator (e.g., "AI moderator reviewing...").

Other users see a placeholder (e.g., "[Username] is posting...") while comments are being reviewed (typically 2-5 seconds).

Comments that don't pass safety review can be appealed for human review.

3.4. Comment Management
Editing: Users can edit their own comments after posting. Edited comments will display an updated timestamp.

Deleting: Users can delete their own comments at any time.

Appeals: If a comment is rejected during safety review, users can request a manual review by a moderator via a "Request Human Review" button.

4. What is OUT OF SCOPE for Version 1.0
To ensure a focused and timely initial release, the following features will be deferred to a future version:

Nested Comments (Replies): Users cannot reply directly to other comments. All comments will appear in a single, flat list.

Likes, Reactions, or Upvotes: There will be no system for users to "like" posts or comments.

Image & Media Uploads: The content body and comments will be text-only.

Post Drafts & Scheduling (Admin): Posts are published immediately upon creation.

Content Moderation Dashboard: Admin interface for reviewing flagged content and appeals (this is part of a platform-wide moderation system, not devlog-specific).

User Reporting: Ability for users to report other users' comments (deferred to future version).

5. Assumptions
A robust user authentication system (account creation, login, logout) already exists and is functional.

Every user has a unique ID and a publicly visible "Display Name."

A centralized content moderation system (as specified in the UGC Technical Spec) is implemented and operational.

Real-time updates via Firestore listeners are supported by the frontend framework.

Cloud infrastructure (Cloud Functions, Cloud Run) for content moderation is deployed and configured.

6. Technical Implementation Notes
Comments follow the standardized UGC pipeline as defined in the technical specification:

Storage: Comments are stored in the subcollection `devlogPosts/{postId}/comments/{commentId}`.

Schema: Each comment uses the generic content document schema with moderation status fields.

Moderation: All comments are subject to automated LLM moderation before public visibility.

Real-time Updates: The system provides instant UI feedback to authors while maintaining safety for the broader community through the hybrid moderation approach.