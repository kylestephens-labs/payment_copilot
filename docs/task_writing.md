Review the business plan, architecture, mvp, full list of tasks, Structure using Gherkin syntax, and Task Classification to have complete context of the project. 

business plan, architecture, mvp, full list of tasks...

Everything above is sized to fit comfortably under a ~100 MB commit per task (mostly code/text/sql), single-concern, and testable in isolation.

Now write 

Structure using Gherkin syntax
Given: Set the context or preconditions. Use it for the state of the system before the user action. 
When: Describe the action the user takes. 
Then: Describe the expected outcome or result of the user's action. 
And / But: Add more conditions to Given, When, or Then steps to avoid repetition. 

Best practices for writing tasks
Focus on behavior: Scenarios should test a single, specific behavior or outcome, not multiple at once. 
Write from the user's perspective: Use clear, business-focused language that anyone can understand, rather than technical jargon. 
Keep it simple: Ensure scenarios are easy to read and maintain. Avoid complex or unnecessary steps. 
Use variables for data: Use Scenario Outline with an Examples table to run the same scenario with different data sets, which reduces redundancy. 
Write testable steps: Focus on the intention (the outcome) rather than the specific mechanics (how to achieve the precondition)
Make steps independent: Each scenario should be able to run on its own without relying on the successful completion of others. 

## 1) Task Classification
- **Functional** → needs testable business logic (clear inputs/outputs)
- **Non-Functional** → conflicts, config/env, one-time fixes, file edits
- **Decision rule:** "Does this require writing *testable* business logic?" If **yes → Functional**

Task template:

## Task [ID]: [Title]
- **Task Classification**: [Functional/Non-Functional]
- **Status**: [Ready/In Progress/Completed]

## Overview of the task

## Goal of the task

### BDD Scenario

Feature: [Feature Name]
  As a [user type]
  I want to [goal]
  So that [benefit]

  Scenario: [Main scenario]
    Given [precondition]
    When [action]
    Then [expected result]
### Acceptance Criteria
- [ ] [Specific, testable requirement 1]
- [ ] [Specific, testable requirement 2]
- [ ] [Specific, testable requirement 3]

### Files & Resources
- **Files Affected**: [Files to create/modify]
- **Dependencies**: [What must be done first]
- **External Resources**: [APIs, tools, documentation]

### Business Context
- **Value**: [Why this matters?]
- **Risk**: [What could go wrong?
- **Success**: [What needs to be tested and validated to know it's done with high quality?]

Use the task template to write the task for: