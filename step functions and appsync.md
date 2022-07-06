# Step Functions and AppSync

## Step Functions
- Model your workflows as state machines (1 per workflow)
- Define workflow in JSON
- Visualize the workflow
- Every step could be Lambda function, DB insert ...
- To start a workflow, can use SDK, `API Gateway`, `EventBridge`

### Task States
- Each task state can invoke a service like Lambda, ECS, SQS ... or launch the next Step Function workflow
- A task can also be defined in an Activity
- An activity can poll step functions for work then sends result back to step functions

### Error Handling
- If an error happens we should not catch it inside the code to make the code simpler
- Should use Retry and Catch in the State Machine to handle instead
- Predefined error codes:
  - `State.ALL`: matches any errors
  - `State.Timeout`: task runs longer than `TimeoutSeconds`
  - `State.TaskFailed`: execution failure
  - `State.Permissions`: insufficient privileges
- `Retry`: defines how and number of times to retry
- `Catch`: handles errors after retries
  - `ErrorEquals`: match a specific type of error
  - `Next`: State to send to
  - `ResultPath`: specify what input is sent to the state of the `Next`
  - For example: `"ResultPath": "$.error"` will include the error in the input of the next step
  