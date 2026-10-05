# AWS Step Functions — State Machine Structure

## Basic Structure

```json
{
  "Comment": "Description of the state machine",
  "StartAt": "First State",
  "TimeoutSeconds": 3600,

  "States": {
    "First State": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",

      "Parameters": {
        "FunctionName": "my-function",
        "Payload.$": "$"
      },

      "InputPath": "$",

      "ResultSelector": {
        "result.$": "$.Payload"
      },

      "ResultPath": "$.lambda",

      "OutputPath": "$",

      "Next": "Next State"
    },

    "Next State": {
      "Type": "Pass",
      "End": true
    }
  }
}
```

---

## State Machine

Top-level fields:

| Field            | Purpose                          |
| ---------------- | -------------------------------- |
| `Comment`        | Description of the state machine |
| `StartAt`        | Name of the first state          |
| `TimeoutSeconds` | Maximum execution time           |
| `States`         | All states in the state machine  |

`States` contains the individual states:

```text
Task
Pass
Choice
Wait
Parallel
Map
Succeed
Fail
```

---

## State

A state generally looks like:

```json
"State name": {
  "Type": "Task",
  "Next": "Another State"
}
```

### Main State Types

| Type       | Purpose                        |
| ---------- | ------------------------------ |
| `Task`     | Perform work                   |
| `Pass`     | Pass or transform state data   |
| `Choice`   | Conditional branching          |
| `Wait`     | Wait for a period/time         |
| `Parallel` | Run branches in parallel       |
| `Map`      | Process multiple items         |
| `Succeed`  | Successfully finish execution  |
| `Fail`     | Finish execution with an error |

---

# Task

A `Task` performs some work.

```json
{
  "Type": "Task",
  "Resource": "...",
  "Parameters": {},
  "Next": "Next State"
}
```

## Resource

Defines the service or operation to invoke.

### Lambda

```json
"Resource": "arn:aws:states:::lambda:invoke"
```

### SNS

```json
"Resource": "arn:aws:states:::sns:publish"
```

### SQS

```json
"Resource": "arn:aws:states:::sqs:sendMessage"
```

---

# Service Integration Patterns

There are three main integration patterns.

| Pattern             | Meaning                                                  |
| ------------------- | -------------------------------------------------------- |
| Request Response    | Call the service and receive its response                |
| `.sync`             | Start a job and wait until it completes                  |
| `.waitForTaskToken` | Wait until an external process sends the task token back |

## Request Response

This is the default behavior.

```text
Step Functions
      ↓
    Service
      ↓
   response
      ↓
    Next
```

Example:

```json
"Resource": "arn:aws:states:::sns:publish"
```

---

## `.sync`

Start a job and wait until it completes.

Example:

```text
ecs:runTask.sync
```

```text
Step Functions
      ↓
    ECS task
      │
      │ running...
      │
      ▼
   completed
      ↓
Step Functions
      ↓
    Next
```

**Mental model:**

> Start the work and wait for AWS to tell Step Functions that it is finished.

---

## `.waitForTaskToken`

Send a task token to an external worker and wait for a callback.

Example:

```text
sqs:sendMessage.waitForTaskToken
```

```text
Step Functions
      ↓
     SQS
      ↓
    Worker
      ↓
SendTaskSuccess(token)
      ↓
Step Functions
      ↓
    Next
```

The worker must call:

```text
SendTaskSuccess
```

or:

```text
SendTaskFailure
```

**Mental model:**

> Give the external process a token and wait until it gives the token back.

---

# State Data Flow

The most useful mental model is:

```text
                 STATE INPUT
                     │
                     ▼
                 InputPath
                     │
                what to take
                     │
                     ▼
                 Parameters
                     │
              what to send
                     │
                     ▼
                  ┌─────┐
                  │ Task│
                  └──┬──┘
                     │
                  result
                     │
                     ▼
              ResultSelector
                     │
             what to take
             from the result
                     │
                     ▼
                ResultPath
                     │
              where to put it
                     │
                     ▼
                OutputPath
                     │
              what to pass on
                     │
                     ▼
               NEXT STATE
```

---

# InputPath

Selects what part of the input state should be used.

```json
"InputPath": "$.user"
```

Input:

```json
{
  "user": {
    "id": 42,
    "name": "Igor"
  },
  "debug": true
}
```

Selected input:

```json
{
  "id": 42,
  "name": "Igor"
}
```

### Remember

> `InputPath` = **What do I take from the input?**

---

# Parameters

Defines what is sent to the service being called.

```json
"Parameters": {
  "FunctionName": "my-function",
  "Payload.$": "$"
}
```

Example:

```json
"Parameters": {
  "FunctionName": "my-function",
  "Payload": {
    "userId.$": "$.user.id"
  }
}
```

Input:

```json
{
  "user": {
    "id": 42
  }
}
```

Lambda receives:

```json
{
  "userId": 42
}
```

### Remember

> `Parameters` = **What do I send to the Task?**

---

# ResultSelector

Transforms or selects data from the result returned by the Task.

Suppose Lambda returns:

```json
{
  "Payload": {
    "id": 42,
    "name": "Igor"
  },
  "StatusCode": 200
}
```

Use:

```json
"ResultSelector": {
  "user.$": "$.Payload"
}
```

Result:

```json
{
  "user": {
    "id": 42,
    "name": "Igor"
  }
}
```

### Remember

> `ResultSelector` = **What do I take from the Task result?**

---

# ResultPath

Defines where the Task result is placed in the state.

Input:

```json
{
  "userId": 42
}
```

Task result:

```json
{
  "permissions": ["read", "write"]
}
```

With:

```json
"ResultPath": "$.result"
```

The state becomes:

```json
{
  "userId": 42,
  "result": {
    "permissions": ["read", "write"]
  }
}
```

You can also place the result deeper:

```json
"ResultPath": "$.user.permissions"
```

### Remember

> `ResultPath` = **Where do I put the Task result?**

---

# OutputPath

Selects what is passed out of the state to the next state.

Given:

```json
{
  "userId": 42,
  "result": {
    "permissions": ["read", "write"]
  }
}
```

With:

```json
"OutputPath": "$.result"
```

The next state receives:

```json
{
  "permissions": ["read", "write"]
}
```

### Remember

> `OutputPath` = **What do I pass to the next state?**

---

# Error Handling

A `Task` can use `Retry` and `Catch`.

```json
"Retry": [
  {
    "ErrorEquals": ["States.ALL"],
    "MaxAttempts": 3,
    "BackoffRate": 2
  }
],

"Catch": [
  {
    "ErrorEquals": ["States.ALL"],
    "Next": "Handle Error"
  }
]
```

Flow:

```text
Task
 │
 ├── success ──────────────→ Next
 │
 └── error
      ↓
    Retry
      │
      ├── success ─────────→ Next
      │
      └── error
           ↓
         Catch
           ↓
      Handle Error
```

---

# State Transitions

## `Next`

Moves execution to another state:

```json
"Next": "Next State"
```

Flow:

```text
State A
   ↓
State B
   ↓
State C
```

## `End`

Marks the state as the final state:

```json
"End": true
```

---

# Quick Reference

| Field            | Simple question                          |
| ---------------- | ---------------------------------------- |
| `InputPath`      | **What do I take from the input?**       |
| `Parameters`     | **What do I send to the Task?**          |
| `ResultSelector` | **What do I take from the Task result?** |
| `ResultPath`     | **Where d**                              |
