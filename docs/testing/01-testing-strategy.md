# Testing Strategy

**Version:** 0.1  
**Status:** Draft  
**Author:** Vahid  
**Last Updated:** 2026-09-10

---

## Purpose

This document defines the testing strategy for the AI-native Software Engineering Workspace.

Testing is part of the development process from the beginning rather than an activity performed after implementation is complete.

The strategy combines:

- Domain-Driven Design (DDD).
- Behavior-Driven Development (BDD).
- Test-Driven Development (TDD).
- Unit testing.
- Integration testing.
- Frontend component testing.
- End-to-end testing.

The objective is not to maximize the number of tests.

The objective is to create confidence that important product behavior works correctly while keeping the test suite maintainable, fast, and useful.

---

# Testing Philosophy

The project follows one primary principle:

> Test behavior and business rules, not implementation details.

Tests should protect meaningful system behavior.

They should not make normal refactoring unnecessarily difficult.

A good test should answer:

- What behavior are we protecting?
- Why would failure of this behavior matter?
- What business rule does this test represent?
- At which testing level should this behavior be verified?

---

# Relationship Between DDD, BDD, and TDD

DDD, BDD, and TDD solve different problems.

They are complementary rather than competing approaches.

---

## Domain-Driven Design

DDD helps us understand:

> What does the business domain mean?

DDD defines concepts such as:

- Project.
- Task.
- Technical Decision.
- AI Artifact.
- Approval.
- Project Knowledge.

It also defines the rules that must always remain true.

Example:

> An AI Artifact cannot become approved project knowledge without authorization from a human user.

This is a domain rule.

---

## Behavior-Driven Development

BDD helps us describe:

> How should the system behave from the user's or business perspective?

BDD scenarios use domain language and describe observable outcomes.

Example:

```gherkin
Feature: AI artifact approval

  Scenario: An authorized engineer approves an AI-generated artifact
    Given an AI-generated artifact is awaiting review
    And an authorized engineer has access to the project
    When the engineer approves the artifact
    Then the artifact should become approved
    And the engineer should be recorded as the approver
    And the artifact should become available as project knowledge
BDD scenarios should remain understandable without knowledge of:

SQL.
Entity Framework.
React.
HTTP.
Cloud infrastructure.
Test-Driven Development

TDD helps us decide:

How do we safely implement the behavior?

The development cycle is:

RED
Write a failing test.

    ↓

GREEN
Write the minimum implementation required to make it pass.

    ↓

REFACTOR
Improve the design while keeping the tests passing.

TDD is primarily an implementation technique.

BDD operates at a higher behavioral level.

DDD provides the domain language and rules used by both.

Development Flow

For important features, the expected workflow is:

Product Requirement
        ↓
Use Case
        ↓
BDD Scenario
        ↓
Domain Rule
        ↓
Acceptance Criteria
        ↓
Failing Test
        ↓
Implementation
        ↓
Passing Test
        ↓
Refactoring
        ↓
Integration Test
        ↓
End-to-End Validation

Not every small code change requires this complete process.

The amount of ceremony should match the importance and complexity of the behavior.

Testing Pyramid

The project should generally follow this structure:

             /\
            /  \
           / E2E\
          /------\
         /Integration\
        /------------\
       / Unit Tests   \
      /----------------\

The majority of tests should be fast unit tests.

A smaller number should verify integration boundaries.

Only critical user journeys should require end-to-end tests.

1. Domain Unit Tests

Domain tests verify business rules without involving:

Database.
HTTP.
External APIs.
File systems.
AI providers.
React.
Infrastructure.

These should normally be the fastest tests in the project.

Examples include:

A Project cannot be created with an invalid name.
An archived Project cannot accept normal modifications.
A Task must belong to a Project.
An AI Artifact cannot approve itself.
A rejected AI Artifact cannot become project knowledge.
A Technical Decision can supersede another decision without deleting history.
Example

Conceptually:

[Fact]
public void Approve_ShouldMarkArtifactAsApproved_WhenReviewerIsAuthorized()
{
    // Arrange

    // Act

    // Assert
}

At this stage, the exact class design is intentionally undefined.

The domain model and tests should help shape that design.

2. Application-Level Tests

Application tests verify use-case orchestration.

Examples include:

Creating a Project.
Creating a Task.
Approving an AI Artifact.
Recording a Technical Decision.
Retrieving Project Knowledge.

These tests may verify that the application correctly coordinates:

Domain objects.
Authorization.
Persistence abstractions.
External-service abstractions.

They should avoid unnecessary dependency on real infrastructure where a simpler substitute provides enough confidence.

3. Backend Integration Tests

Integration tests verify that important infrastructure components work together correctly.

For the .NET backend, these tests may verify:

ASP.NET Core endpoints.
Dependency injection.
Authentication.
Authorization.
Entity Framework Core configuration.
PostgreSQL persistence.
Transactions.
API serialization.
Error handling.

Where practical, integration tests should use infrastructure close to production behavior.

A real PostgreSQL test container is preferred for important persistence behavior rather than relying entirely on an in-memory database with different semantics.

4. API Tests

Important API behavior should be verified through HTTP.

Examples:

POST /projects
GET /projects/{id}
POST /projects/{id}/tasks
POST /ai/artifacts/{id}/approve
GET /projects/{id}/knowledge

API tests should verify:

HTTP status codes.
Request validation.
Authorization.
Response contracts.
Error responses.
Business behavior.

Not every backend test should go through HTTP.

Domain rules should remain testable independently.

5. Frontend Unit and Component Tests

The React frontend should include tests for meaningful component behavior.

Potential tools:

Vitest.
React Testing Library.

Tests should focus on behavior visible to the user.

Example:

Instead of testing:

The component's internal state variable becomes true.

Prefer testing:

When the user clicks Approve, the confirmation interface appears.

Good Frontend Test Targets

Examples include:

Form validation.
Loading states.
Error states.
Conditional rendering.
User interaction.
Approval workflows.
Authentication behavior.
Task state changes.
6. End-to-End Tests

End-to-end tests verify complete user journeys through the real application interface.

Potential tooling:

Playwright.

Because E2E tests are slower and more fragile than lower-level tests, they should be reserved for critical workflows.

Initial Critical E2E Scenario

The most important MVP journey is:

Register
    ↓
Sign In
    ↓
Create Project
    ↓
Create Task
    ↓
Generate AI Artifact
    ↓
Review Artifact
    ↓
Approve Artifact
    ↓
Record Technical Decision
    ↓
Leave Project
    ↓
Return Later
    ↓
Retrieve Project Knowledge

We do not need dozens of E2E tests for the first version.

A few high-value scenarios provide more value than a large fragile suite.

7. AI Testing Strategy

AI functionality requires a different testing approach because AI output is probabilistic.

We should not write fragile tests that expect exact generated sentences.

Instead, AI testing should be separated into multiple layers.

AI Orchestration Unit Tests

Core application behavior should be testable without calling a real AI provider.

The AI provider should be represented behind an abstraction.

Tests can then verify:

Correct context selection.
Correct request construction.
Correct authorization.
Correct handling of provider responses.
Correct artifact creation.
Correct approval behavior.

Example:

Given the AI provider returns generated requirements
When the generation use case completes
Then an AI Artifact should be created
And the artifact should remain unapproved
AI Provider Integration Tests

A smaller number of tests may verify communication with the real AI provider.

These tests should not run with every normal unit-test execution because they may:

Cost money.
Be slower.
Fail due to external availability.
Produce non-deterministic output.
AI Evaluation Tests

As the AI capabilities become more important, we should create evaluation scenarios.

For example:

Input:

Create acceptance criteria for password reset.

Evaluation may check whether the result contains relevant concepts such as:

Expiration.
Invalid token handling.
Successful password update.
Security considerations.

Evaluation should focus on quality criteria rather than exact wording.

8. External Integration Testing

Future integrations such as GitHub, Jira, and Slack should be tested at explicit boundaries.

Core application tests should not require those external systems to be available.

For example:

Core Domain
    ↓
Integration Interface
    ↓
GitHub Adapter
    ↓
GitHub API

The integration interface can be replaced with a test implementation during core application testing.

Separate integration tests can verify the real provider adapter.

Test Doubles

Different types of test doubles may be used where appropriate.

These include:

Stub.
Fake.
Mock.
Spy.

They should be used intentionally.

Excessive mocking should be avoided because it often creates tests that verify implementation details rather than behavior.

Prefer testing real domain objects whenever practical.

Mocking Guidelines

Mock dependencies when the dependency represents an external boundary such as:

AI provider.
Email service.
External API.
Time provider.
External integration.

Avoid mocking simple domain objects.

For example:

Avoid:

Mock<Project>
Mock<Task>
Mock<TechnicalDecision>

Prefer using real domain objects when testing business behavior.

Database Testing

Database behavior should not rely exclusively on mocked repositories.

Important persistence scenarios should eventually run against PostgreSQL.

Potential approach:

Test
 ↓
ASP.NET Core Application
 ↓
Entity Framework Core
 ↓
PostgreSQL Test Container

This allows tests to detect problems involving:

Constraints.
Transactions.
Migrations.
Relationships.
PostgreSQL-specific behavior.
Query behavior.
Test Isolation

Tests should be independent.

A test should not require another test to run first.

Each test should establish the state it needs.

Tests should not depend on:

Execution order.
Shared mutable global state.
Existing developer databases.
Previous test execution.
Test Naming

Test names should describe behavior clearly.

One possible convention is:

MethodUnderTest_ShouldExpectedBehavior_WhenCondition

Example:

Approve_ShouldRejectRequest_WhenUserIsUnauthorized

Another valid style is behavior-oriented naming:

CannotApproveArtifactWithoutProjectAccess

Consistency and readability are more important than enforcing one naming format everywhere.

Arrange, Act, Assert

Most unit tests should clearly separate:

Arrange
Act
Assert

Example:

[Fact]
public void Create_ShouldRejectEmptyProjectName()
{
    // Arrange

    // Act

    // Assert
}

This structure helps make tests easier to understand.

What Should Be Tested

Tests should prioritize risk.

Highest-value targets include:

Business invariants.
Authorization.
Authentication.
Project isolation.
AI approval workflows.
Data integrity.
Technical decision history.
Important API contracts.
Critical user workflows.
Failure handling.
What Should Not Be Tested Excessively

Avoid spending large amounts of time testing:

Framework behavior.
Trivial property getters.
Third-party library internals.
Exact HTML implementation details.
Exact AI wording.
Private methods directly.
Implementation details with no user or business consequence.
Coverage

Code coverage may be measured, but it should not become the primary quality goal.

For example:

90% code coverage

does not necessarily mean:

90% confidence in the application

A smaller set of meaningful tests is more valuable than hundreds of tests that execute code without validating useful behavior.

Backend Testing Tooling

The initial backend testing stack is expected to include:

xUnit
ASP.NET Core testing infrastructure
Entity Framework Core integration testing
PostgreSQL
Testcontainers

Additional libraries may be introduced when justified.

Tool selection will be documented through architecture decisions where appropriate.

Frontend Testing Tooling

The initial frontend testing stack is expected to include:

Vitest
React Testing Library
Playwright

Expected responsibilities:

Vitest + React Testing Library
    → component and frontend behavior

Playwright
    → critical end-to-end workflows
CI Testing

Automated tests should eventually run as part of Continuous Integration.

A typical pipeline may become:

Pull Request
    ↓
Restore Dependencies
    ↓
Build
    ↓
Backend Unit Tests
    ↓
Frontend Tests
    ↓
Backend Integration Tests
    ↓
Quality Checks
    ↓
Build Result

Critical failures should prevent unsafe changes from being merged.

E2E execution frequency may be adjusted later based on execution cost and speed.

First TDD Feature

We should not begin TDD with authentication.

Authentication contains significant framework and infrastructure behavior and is not the best place to learn the TDD cycle.

The first feature used to practice TDD should be:

Create Project

This feature has useful domain behavior while remaining small enough to understand clearly.

The initial implementation sequence should be approximately:

ProjectName
    ↓
Project Creation Domain Rules
    ↓
CreateProject Use Case
    ↓
Persistence
    ↓
API Endpoint
    ↓
Frontend

At each step, appropriate tests should be written before or alongside implementation.

Example First TDD Cycle

The first domain rule may be:

A Project cannot be created without a valid name.

RED

Write a test:

[Fact]
public void Create_ShouldFail_WhenProjectNameIsEmpty()
{
    // Test implementation will be added during development.
}

Run the test.

The test should fail because the behavior does not exist yet.

GREEN

Implement the minimum domain behavior necessary to reject an invalid project name.

Run the test again.

The test should pass.

REFACTOR

Improve:

Naming.
Encapsulation.
Domain structure.

Run the tests again.

Everything must remain green.

Then move to the next behavior.

BDD Scenario Selection

We will not write BDD scenarios for every possible operation.

Initially, we should document BDD scenarios for the most important domain behaviors.

The first candidates are:

Create Project.
Prevent unauthorized Project access.
Create Task.
Approve AI Artifact.
Reject AI Artifact.
Record Technical Decision.
Supersede Technical Decision.
Retrieve approved Project Knowledge.

These scenarios will later become part of the executable testing strategy where practical.

Definition of Done

A feature should not be considered complete merely because the UI appears to work.

For important MVP functionality, Definition of Done includes:

Required behavior implemented.
Domain rules enforced.
Relevant unit tests passing.
Relevant integration tests passing.
Frontend behavior tested where appropriate.
Error cases handled.
Authorization verified.
Documentation updated if behavior changed.
Code reviewed.
No known critical defects.
Testing Responsibility During AI-Assisted Development

AI tools such as GitHub Copilot and Claude may help:

Generate initial tests.
Suggest edge cases.
Review tests.
Identify missing scenarios.
Generate test data.
Analyze failures.

However, AI must not determine the testing strategy independently.

The engineer remains responsible for:

Understanding the behavior being tested.
Validating generated tests.
Detecting meaningless tests.
Ensuring important edge cases exist.
Understanding why tests pass or fail.

A generated test that the engineer cannot explain should not be accepted into the codebase.

Learning Objective

Testing is also a deliberate learning objective for this project.

By the end of the MVP development process, the developer should be comfortable explaining:

Why automated tests are necessary.
Unit vs integration vs E2E testing.
TDD.
BDD.
Red-Green-Refactor.
Arrange-Act-Assert.
Test doubles.
Mocking.
Dependency injection and testability.
Testing domain behavior.
Testing ASP.NET Core APIs.
Testing EF Core with PostgreSQL.
Testing React components.
Testing user workflows with Playwright.
Testing AI-dependent functionality.
CI test automation.

The goal is not simply to use these techniques.

The developer should be able to explain the engineering trade-offs behind them during an interview.

Guiding Principle

The testing strategy follows one central rule:

Tests should give us enough confidence to change the system safely.

Testing exists to support engineering evolution.

It should not become a bureaucratic obstacle to development.
