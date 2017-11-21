# Amazon Mechanical Turk REST API Reference
This [suite of APIs](../readme.md#what-apis-are-available) simplifies how Requesters use Amazon Mechanical Turk (MTurk) by enabling you to automate most of the mechanics behind Human Intelligence Task (HIT) design, creation and approval.  The MTurk team has provided a [Python client], that simplifies using the single purpose APIs. Learn how to [set up and use the Python Client].

If you would prefer to use a different programming language you can build your own client in any language as long as your client uses [AWS’s Signature Version 4 Signing Process] to authentication your API requests.  These are REST style APIs using HTTP requests as defined in RFC 2616. 

## Making HTTP Requests
All operations on a Task follow the same REST URL structure:

#### HTTPS request - 
```
https://crowd.us-east-1.amazonaws.com/<AWS account number>/functions/<function name>/tasks/<task name>
```

| Name | Description | 
| ---- | ----------- | 
| AWS account number | User's 12-digit AWS account number, can be located under [AWS account settings] |
| Function name |  [API name from MTurk](../readme.md#what-apis-are-available) either `<api-name>` or `<api-name>-test`. |
| Task name | User-provided Task name This string  must consist of alphanumeric characters, period (`.`), hyphen (`-`), and underscore (`_`), and must be of a length between 1 and 64. |

Every Task within your account is uniquely identified by the combination of the API name and Task name.

---

## Create Task (PUT)
Tasks are created by calling PUT. Each API Task has a unique input formatted as a JSON object that has specific key value pairs as defined by the API you call. You will need to specify the API that you want to call, and give your Task a unique name which is also used to return the results later with a GET request.

### Request structure
This operation requires permission for the  action. 

#### HTTPS request - 

```
PUT https://crowd.us-east-1.amazonaws.com/<AWS account number>/functions/<function name>/tasks/<task name>

```

#### Request body - 
```
"input": <input provided by user, JSON object with required parameters defined by MTurk>
```


### Response structure

| Name | Description | Type | 
| ---- | ----------- | ---- |
|"taskName" | user-provided Task name  | String |
|"input" | input provided by user when Task was created  | JSON Object |
|"problemDetails" | if the "state" is "failed" - details about a failed Task | JSON Object or null |
|"state" | current Task state - one of "processing", "completed" or "failed" | string |
|"result" | if the "state" is "completed" - the results of the completed Task  | JSON Object or null |

> Note: The PUT operation is idempotent. Each call must have a unique Task Name
> 
#### Example response
```
{
  "taskName": <user-provided task name, string>,
  "input": <input provided by user when task was created, JSON object>,
  "problemDetails": null,
  "state": "processing",
  "result": null 
}
```

### HTTP Status Codes
| HTTP status code | Description | 
| ---- | ----------- | 
| 201&nbsp;Created | Request succeeds and new Task created. |
| 200&nbsp;OK | Request succeeds idempotently for a Task that already exists. |
| 400&nbsp;Bad&nbsp;Request |  Request input is syntactically invalid see message in the body describing the problem. |
| 400&nbsp;Bad&nbsp;Request |  Request is made using a Task name that’s already in use but with a different input than the existing Task. Remember the input of an existing Task cannot be modified. |
| 404&nbsp;Not&nbsp;Found |  The API name you provided is misspelled. |
| 429&nbsp;Too&nbsp;Many&nbsp;Requests | Request was throttled. If you receive a throttling error, you can backoff and retry your request. |

---

## Return Results (GET)
After creating a Task, users can call the GET operation to poll its current state. To call GET you will need to pass in the API name and the Task name used during the creation of the Task.

>Note: After 90 days, your Tasks may no longer be available when you call GET on a Task.

### Request structure

#### HTTPS request - 
```
GET https://crowd.us-east-1.amazonaws.com/<AWS account number>/functions/<function name>/tasks/<task name>
```

### Response structure

| Name | Description | Type | 
| ---- | ----------- | ---- |
|"taskName" | user-provided Task name  | String |
|"input" | input provided by user when Task was created  | JSON Object |
|"problemDetails" | if the "state" is "failed" - details about a failed Task | JSON Object or null |
|"state" | current Task state - one of "processing", "completed" or "failed" | string |
|"result" | if the "state" is "completed" - the results of the completed Task  | JSON Object or null |

#### Example response
```
{
  "taskName": <user-provided task name, string>,
  "input": <input provided by user when task was created, JSON object>,
  "problemDetails": <JSON object containing details about a failed task; 
                   populated if and only if the "state" is "failed", and null otherwise>,
  "state": <current task state; string; one of "processing", "completed", or "failed">,
  "result": <JSON object containing the results of the completed task; 
           populated if and only if the "state" is "completed", and null otherwise>
}
```

> Since Tasks are completed asynchronously by human Workers, they always start out in a “processing” state after being created. Eventually, they will move to a finished state, either “completed” or “failed”

| "state" | Description | 
| ------- | ----------- | 
| "processing" | Still waiting for Workers to complete a Task. |
| "failed" |  Workers did not respond within the allotted time, explanation under `"problemDetails"`. |
| "completed" | Workers have completed a Task. Your response will now also include a JSON object containing the results of the completed Task for the API you called. |

### HTTP Status Codes
| HTTP status code | Description | 
| ---- | ----------- | 
| 200 OK | Request succeeds idempotently for a Task that already exists. |
| 404 Not Found |  Request was for a non-existent Task name. |
| 429 Too Many Requests | Request was throttled. If you receive a throttling error, you can backoff and retry your request. |

---

## Delete Task (DELETE)
A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

Currently, the primary use case for deleting a Task is making the Task name available for re-use. For instance, if your Task using the Task name “my-natural-key-123” fails because of an issue with your [account setup], you may want to create a new Task using the same Task name and input to try again to get results. To do so, you could delete the existing Task and create a new one in its place. (Alternatively, you could pick a different Task name for the new Task instead of reusing the old Task name.)

### Request structure

#### HTTPS request - 
```
DELETE https://crowd.us-east-1.amazonaws.com/<AWS account number>/functions/<function name>/tasks/<task name>
```

### HTTP Status Codes
| HTTP status code | Description | 
| ---- | ----------- | 
| 204 No Content | Request succeeds, Task deleted |
| 404 Not Found |  Request was for a non-existent Task name including one that existed but was previously deleted. |
| 429&nbsp;Too&nbsp;Many&nbsp;Requests | Request was throttled. If you receive a throttling error, you can backoff and retry your request. |


[Python client]: https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html
[set up and use the Python Client]: step_2_first_task.md
[AWS’s Signature Version 4 Signing Process]: https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html
[AWS account settings]: https://console.aws.amazon.com/billing/home#/account
[account setup]: step_1_setup_aws_user.md
