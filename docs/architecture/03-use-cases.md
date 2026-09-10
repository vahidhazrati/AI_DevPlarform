# Use Cases

**Version:** 0.1  
**Status:** Draft  
**Author:** Vahid  
**Last Updated:** 2026-09-10

---

## Purpose

This document defines the primary application use cases for the AI-native Software Engineering Workspace.

The purpose of these use cases is to describe how users interact with the system to achieve meaningful engineering goals.

These use cases intentionally focus on business behavior rather than implementation details.

They do not define:

- REST endpoints.
- Database tables.
- Entity Framework models.
- React components.
- AI provider SDKs.
- Infrastructure technologies.

The use cases will later be used to derive:

- BDD scenarios.
- Acceptance criteria.
- Application services.
- Domain behavior.
- Automated tests.
- API contracts.

The initial focus is the MVP, while future product requirements remain important architectural considerations.

---

# Actors

## Software Engineer

The primary actor for the MVP.

A Software Engineer can:

- Create and manage projects.
- Manage engineering tasks.
- Create engineering documents.
- Interact with project-aware AI.
- Review AI-generated artifacts.
- Record technical decisions.
- Retrieve historical engineering context.

---

## Tech Lead

A Tech Lead may perform the same operations as a Software Engineer while also reviewing technical decisions and maintaining engineering consistency.

Advanced approval and permission workflows are expected in later versions.

---

## AI System

The AI System is a supporting actor.

It may:

- Generate content.
- Analyze project context.
- Suggest tasks.
- Generate documentation.
- Summarize information.
- Suggest implementation approaches.

The AI System does not have unrestricted authority to modify official project state.

---

# UC-001 — Register User

## Goal

Allow a new user to create an account and gain access to the platform.

## Primary Actor

Software Engineer.

## Preconditions

The user does not already have an account using the supplied identity.

## Trigger

The user chooses to create an account.

## Main Flow

1. The user provides the required registration information.
2. The system validates the supplied information.
3. The system creates the user account.
4. The system establishes the user's authenticated identity.
5. The user gains access to the application.

## Alternative Flows

### Invalid Registration Information

If required information is invalid, the system rejects the request and explains what must be corrected.

### Existing Account

If the identity is already registered, the system does not create a duplicate account.

## Postconditions

A valid user account exists and can authenticate with the platform.

## Related Requirements

- FR-001 — User Authentication.
- NFR-003 — Security.
- NFR-004 — Data Isolation.

---

# UC-002 — Authenticate User

## Goal

Allow an existing user to securely access the platform.

## Primary Actor

Software Engineer.

## Preconditions

The user has an existing account.

## Trigger

The user attempts to sign in.

## Main Flow

1. The user provides authentication credentials or uses a supported identity provider.
2. The system validates the authentication request.
3. The system identifies the user.
4. The system establishes an authenticated session.
5. The user gains access to authorized resources.

## Alternative Flows

### Authentication Failure

If authentication fails, access is denied without revealing sensitive security information.

## Postconditions

The authenticated user can access resources permitted by the authorization model.

## Future Considerations

Future versions may support enterprise authentication through providers such as:

- Microsoft Entra ID.
- Google.
- Okta.
- Other OpenID Connect or SAML providers.

Authentication and business authorization should remain separate concerns.

---

# UC-003 — Create Project

## Goal

Allow an authenticated engineer to create a new software engineering workspace.

## Primary Actor

Software Engineer.

## Preconditions

- The user is authenticated.
- The user is authorized to create projects.

## Trigger

The user chooses to create a new project.

## Main Flow

1. The user starts project creation.
2. The user provides a project name.
3. The user may provide an initial description or product context.
4. The system validates the project information.
5. The system creates the Project.
6. The system establishes ownership or access for the creating user.
7. The system records the project creation activity.
8. The project becomes available to the user.

## Alternative Flows

### Invalid Project Name

If the project name is empty or invalid, the project is not created.

### Unauthorized User

If the user is not authorized to create a project, the operation is rejected.

## Postconditions

- A new Project exists.
- The creating user can access it.
- The project has an initial lifecycle state of Active.
- Project activity contains a creation record.

## Domain Rules

- Every Project must have a valid identity.
- Every Project must have a valid name.
- Every Project must have an authorization boundary.
- Project-owned data must remain isolated from other Projects.

## Related Requirements

- FR-002 — Project Management.
- FR-010 — Project Activity History.

---

# UC-004 — Manage Engineering Task

## Goal

Allow an engineer to create and manage a unit of engineering work.

## Primary Actor

Software Engineer.

## Preconditions

- The user is authenticated.
- The Project exists.
- The user has access to the Project.

## Trigger

The user creates or modifies a Task.

## Main Flow — Create Task

1. The user opens a Project.
2. The user chooses to create a Task.
3. The user provides a title.
4. The user may provide a description and priority.
5. The system validates the input.
6. The Task is created within the Project.
7. The system records the activity.

## Main Flow — Change Task Status

1. The user opens an existing Task.
2. The user selects a new status.
3. The system verifies that the transition is valid.
4. The Task status changes.
5. The change is preserved.
6. Relevant activity is recorded.

## Possible Initial Statuses

- Backlog.
- In Progress.
- Blocked.
- Completed.

## Alternative Flows

### Invalid Status Transition

If a requested transition violates a domain rule, the operation is rejected.

### Project Archived

If the Project is archived, normal Task modifications should not be allowed.

## Postconditions

The Task accurately reflects its latest valid state.

## Domain Rules

- A Task must belong to exactly one Project.
- A Task must have a title.
- A completed Task remains historically discoverable.
- External task systems must not determine the internal domain model.

## Related Requirements

- FR-003 — Task Management.

---

# UC-005 — Create Engineering Document

## Goal

Allow an engineer to create durable engineering documentation.

## Primary Actor

Software Engineer.

## Preconditions

- The user is authenticated.
- The Project exists.
- The user has access to the Project.

## Trigger

The user chooses to create an engineering document.

## Main Flow

1. The user opens a Project.
2. The user chooses to create a Document.
3. The user selects a document type.
4. The user provides a title and content.
5. The system validates the information.
6. The Document is saved within the Project.
7. The Document becomes available for future retrieval.

## Example Document Types

- Requirement.
- User Story.
- Feature Specification.
- Technical Note.
- Implementation Plan.
- Test Scenario.

## Alternative Flow — AI-Assisted Creation

Instead of writing the complete document manually, the user may request AI assistance.

This invokes UC-007 — Generate AI Artifact.

## Postconditions

A durable engineering Document exists within the Project.

## Domain Rules

- A Document must belong to a Project.
- Document origin should remain traceable.
- AI-generated origins must not be silently removed.

## Related Requirements

- FR-005 — AI-Assisted Document Generation.
- FR-007 — Context Preservation.

---

# UC-006 — Ask Project-Aware AI

## Goal

Allow an engineer to interact with AI without repeatedly explaining the entire project context.

## Primary Actor

Software Engineer.

## Supporting Actor

AI System.

## Preconditions

- The user is authenticated.
- The Project exists.
- The user has access to the Project.

## Trigger

The user submits a question or request to the AI assistant from within a Project.

## Main Flow

1. The user opens the AI assistant within a Project.
2. The user submits a request.
3. The system determines what project context may be relevant.
4. The system retrieves authorized project information.
5. The system constructs the AI request.
6. The request is sent to the configured AI provider.
7. The AI provider generates a response.
8. The system returns the response to the user.
9. The interaction is associated with the AI Conversation.

## Example Requests

> What authentication decisions have already been made?

> Break this feature into implementation tasks.

> Generate acceptance criteria for this requirement.

> What are the major risks associated with this implementation?

> Summarize the important decisions made for this project.

## Alternative Flows

### AI Provider Unavailable

If the AI provider is unavailable:

1. The system handles the failure gracefully.
2. Existing project data remains unaffected.
3. The user receives an understandable failure message.

### Insufficient Context

If relevant context cannot be found, the system should avoid pretending that unavailable information is known.

## Postconditions

- The conversation is preserved according to application policy.
- No authoritative project information changes automatically.

## Domain Rules

- AI providers cannot bypass application authorization.
- Only authorized context may be supplied to the AI.
- AI output is treated as generated information rather than trusted fact.
- AI provider failure must not corrupt Project state.

## Related Requirements

- FR-004 — Project-Aware AI Assistant.
- FR-007 — Context Preservation.
- NFR-010 — AI Provider Independence.
- NFR-011 — Human Control of AI.

---

# UC-007 — Generate AI Artifact

## Goal

Allow AI to create a structured engineering artifact that can be reviewed by a human.

## Primary Actor

Software Engineer.

## Supporting Actor

AI System.

## Preconditions

- The user has access to the Project.
- An AI capability is available.
- The requested operation is permitted.

## Trigger

The user requests AI-generated engineering content.

## Main Flow

1. The user selects the type of artifact to generate.
2. The user provides instructions.
3. The system gathers relevant Project context.
4. The system submits the request to the AI provider.
5. The provider generates content.
6. The platform creates an AI Artifact.
7. The artifact is marked as generated or awaiting review.
8. The user is shown the result.
9. The artifact remains non-authoritative until approved.

## Example AI Artifacts

- Requirements.
- User stories.
- Acceptance criteria.
- Implementation plans.
- Task suggestions.
- Test scenarios.
- Architecture suggestions.
- Technical documentation.

## Alternative Flows

### AI Generation Failure

The system reports the failure and does not create an approved artifact.

### User Cancels

The user may discard unwanted AI output without adding it to permanent project knowledge.

## Postconditions

An AI Artifact exists in a non-authoritative state.

## Domain Rules

- AI-generated content must retain provenance.
- AI-generated content is not automatically Project Knowledge.
- Generated content should be reviewable and editable.

## Related Requirements

- FR-005 — AI-Assisted Document Generation.
- FR-009 — Human Review and Control.

---

# UC-008 — Review and Approve AI Artifact

## Goal

Allow a human engineer to decide whether AI-generated information should become trusted project information.

## Primary Actor

Software Engineer.

## Preconditions

- An AI Artifact exists.
- The artifact has not already reached a terminal review state.
- The user is authorized to review the artifact.

## Trigger

The user opens an AI Artifact for review.

## Main Flow — Approve

1. The user reviews the generated content.
2. The user may modify the content.
3. The user chooses Approve.
4. The system verifies authorization.
5. The artifact state changes to Approved.
6. The system records who approved it.
7. The system records when it was approved.
8. The approved information becomes eligible to participate in Project Knowledge.
9. The approval activity is recorded.

## Alternative Flow — Reject

1. The user reviews the artifact.
2. The user chooses Reject.
3. The artifact state changes to Rejected.
4. The rejection is recorded.
5. The artifact does not become authoritative Project Knowledge.

## Postconditions — Approved

- The artifact is Approved.
- Approval information is preserved.
- Approved content can participate in Project Knowledge.

## Postconditions — Rejected

- The artifact is Rejected.
- It does not become trusted Project Knowledge.

## Domain Rules

- AI cannot approve its own artifact.
- Approval requires an authorized human.
- Approval history must remain traceable.
- Original AI provenance must remain available after approval.
- Rejection must not accidentally publish the artifact as trusted knowledge.

## Related Requirements

- FR-009 — Human Review and Control.
- NFR-011 — Human Control of AI.
- NFR-012 — AI Traceability.

---

# UC-009 — Record Technical Decision

## Goal

Preserve an important engineering decision and the reasoning behind it.

## Primary Actor

Software Engineer or Tech Lead.

## Preconditions

- The user is authenticated.
- The Project exists.
- The user has permission to record technical decisions.

## Trigger

An important technical decision is made.

## Main Flow

1. The user chooses to create a Technical Decision.
2. The user describes the problem or context.
3. The user records the considered alternatives.
4. The user records the selected approach.
5. The user records the reasoning.
6. The user records relevant trade-offs or consequences.
7. The system validates the information.
8. The Technical Decision is saved.
9. The decision becomes part of Project Knowledge.
10. The activity is recorded.

## Example

### Decision

Use PostgreSQL as the primary relational database.

### Reasoning

The domain contains strongly related transactional data and requires reliable relational consistency.

### Alternatives

- SQL Server.
- MongoDB.

### Consequences

The development team must maintain PostgreSQL expertise and deployment infrastructure.

## Alternative Flow — Supersede Existing Decision

If a previous decision is no longer valid:

1. A new Technical Decision is created.
2. The new decision references the previous decision.
3. The previous decision is marked as superseded.
4. Both remain historically discoverable.

## Domain Rules

- Historical decisions must not be silently overwritten.
- Important decisions should preserve reasoning.
- Superseded decisions remain discoverable.
- Decisions belong to a Project.

## Related Requirements

- FR-006 — Technical Decision Logging.
- FR-008 — Context Retrieval.

---

# UC-010 — Retrieve Project Knowledge

## Goal

Allow engineers to recover important project information without manually searching across multiple disconnected systems.

## Primary Actor

Software Engineer.

## Preconditions

- The user is authenticated.
- The Project exists.
- The user has access to the Project.

## Trigger

The user searches for information or asks a project-related question.

## Main Flow

1. The user opens a Project.
2. The user enters a search query or question.
3. The system searches authorized Project Knowledge.
4. Relevant information is identified.
5. The system returns the relevant results.
6. Source information is preserved where possible.

## Example Queries

> Why did we choose PostgreSQL?

> What authentication approach was selected?

> Show me the requirements for project creation.

> What tasks are currently blocked?

> Which technical decisions have changed?

## Future AI-Assisted Flow

The system may later use semantic retrieval and AI to synthesize answers from multiple knowledge sources.

Where AI is used, the system should preserve enough source information for the user to understand where the answer came from.

## Postconditions

The user obtains relevant historical project information without modifying Project state.

## Domain Rules

- Users may retrieve only information they are authorized to access.
- Search results must not cross Project or tenant boundaries incorrectly.
- Generated summaries should remain distinguishable from original source information.

## Related Requirements

- FR-008 — Context Retrieval.
- NFR-004 — Data Isolation.

---

# UC-011 — Archive Project

## Goal

Allow a Project to become inactive without destroying its engineering history.

## Primary Actor

Software Engineer or authorized Project owner.

## Preconditions

- The Project exists.
- The user is authorized to archive it.

## Trigger

The user chooses to archive the Project.

## Main Flow

1. The user requests project archival.
2. The system verifies authorization.
3. The Project lifecycle state changes from Active to Archived.
4. Existing project information remains available according to access rules.
5. Normal modification operations are restricted.
6. The system records the archival activity.

## Postconditions

The Project remains historically discoverable but is no longer considered active.

## Domain Rules

- Archiving must not delete Project history.
- Archived Projects should reject normal modifications.
- A future restore capability may return the Project to Active state.

---

# UC-012 — View Project Activity

## Goal

Allow users to understand significant events that occurred within a Project.

## Primary Actor

Software Engineer or Tech Lead.

## Preconditions

The user has access to the Project.

## Main Flow

1. The user opens the Project activity view.
2. The system retrieves recent meaningful Project activities.
3. The system orders activities chronologically.
4. The user can inspect relevant activity information.

## Example Activities

- Project created.
- Task created.
- Task completed.
- Document created.
- Technical decision recorded.
- AI artifact generated.
- AI artifact approved.
- Project archived.

## Postconditions

No Project state is changed.

## Future Considerations

Activity history may later contribute to:

- Notifications.
- Audit trails.
- Analytics.
- Compliance.
- Agent monitoring.

The MVP activity model should therefore avoid assumptions that make richer auditing impossible later.

---

# Future Use Cases

The following use cases are intentionally outside the first MVP but remain part of the long-term product direction.

They include capabilities such as:

- Connect GitHub repository.
- Import GitHub issues.
- Analyze pull requests.
- Synchronize Jira work items.
- Capture engineering decisions from Slack.
- Import Confluence documentation.
- Monitor CI/CD pipelines.
- Analyze failed deployments.
- Invite Project members.
- Create Organizations and Teams.
- Assign roles and permissions.
- Configure enterprise identity.
- Configure billing and subscription plans.
- Create organization-wide engineering policies.
- Run autonomous AI agents.
- Approve agent actions.
- Create multi-agent engineering workflows.
- Share knowledge across Projects.

These use cases should influence architectural boundaries but should not be prematurely implemented.

---

# Use Case Relationships

The core MVP workflow can be represented as:

```mermaid
flowchart TD

    A[Register / Authenticate]
    B[Create Project]
    C[Create Project Context]
    D[Manage Tasks]
    E[Ask Project-Aware AI]
    F[Generate AI Artifact]
    G[Human Review]
    H{Approved?}
    I[Add to Project Knowledge]
    J[Reject Artifact]
    K[Record Technical Decision]
    L[Retrieve Project Knowledge]

    A --> B
    B --> C
    C --> D
    C --> E
    E --> F
    F --> G
    G --> H

    H -->|Yes| I
    H -->|No| J

    C --> K

    I --> L
    K --> L
    D --> L

    I --> L
    K --> L
    D --> L
