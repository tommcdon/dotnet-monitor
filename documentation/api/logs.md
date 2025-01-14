# Logs - Get

Captures log statements that are logged to the [ILogger<> infrastructure](https://docs.microsoft.com/aspnet/core/fundamentals/logging) within a specified process.

> **NOTE:** The [`LoggingEventSource`](https://docs.microsoft.com/aspnet/core/fundamentals/logging#event-source) provider must be enabled in the process in order to capture logs.

## HTTP Route

```http
GET /logs/{pid}?level={level}&durationSeconds={durationSeconds}&egressProvider={egressProvider} HTTP/1.1
```

or 

```http
GET /logs/{uid}?level={level}&durationSeconds={durationSeconds}&egressProvider={egressProvider} HTTP/1.1
```

or

```http
GET /logs/{name}?level={level}&durationSeconds={durationSeconds}&egressProvider={egressProvider} HTTP/1.1
```

or

```http
GET /logs?level={level}&durationSeconds={durationSeconds}&egressProvider={egressProvider} HTTP/1.1
```

> **NOTE:** Process information (IDs, names, environment, etc) may change between invocations of these APIs. Processes may start or stop between API invocations, causing this information to change.

## Host Address

The default host address for these routes is `https://localhost:52323`. This route is only available on the addresses configured via the `--urls` command line parameter and the `DOTNETMONITOR_URLS` environment variable.

## URI Parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `pid` | path | false | int | The ID of the process. |
| `uid` | path | false | guid | A value that uniquely identifies a runtime instance within a process. |
| `name` | path | false | string | The name of the process. |
| `level` | query | false | [LogLevel](definitions.md#LogLevel) | The name of the log level at which log events are collected. For .NET Core 3.1, the default is `Warning`. For .NET 5+, logs are collected at the levels configured by the application for each logging category; this can be overriden by setting `level`, which will fallback to collecting all categories at or above the specified level. |
| `durationSeconds` | query | false | int | The duration of the log collection operation in seconds. Default is `30`. Min is `-1` (indefinite duration). Max is `2147483647`. |
| `egressProvider` | query | false | string | If specified, uses the named egress provider for egressing the collected logs. When not specified, the logs are written to the HTTP response stream. See [Egress Providers](../egress.md) for more details. |

See [ProcessIdentifier](definitions.md#ProcessIdentifier) for more details about the `pid`, `uid`, and `name` parameters.

If none of `pid`, `uid`, or `name` are specified, logs for the [default process](defaultprocess.md) will be captured. Attempting to capture logs of the default process when the default process cannot be resolved will fail.

## Authentication

Authentication is enforced for this route. See [Authentication](./../authentication.md) for further information.

Allowed schemes:
- `MonitorApiKey`
- `Negotiate` (Windows only, running as unelevated)

## Responses

| Name | Type | Description | Content Type |
|---|---|---|---|
| 200 OK | | The logs from the process formatted as [newline delimited JSON](https://github.com/ndjson/ndjson-spec). Each JSON object is a [LogEntry](definitions.md#LogEntry) | `application/x-ndjson` |
| 200 OK | | The logs from the process formatted as [server-sent events](https://www.w3.org/TR/eventsource). | `text/event-stream` |
| 400 Bad Request | [ValidationProblemDetails](definitions.md#ValidationProblemDetails) | An error occurred due to invalid input. The response body describes the specific problem(s). | `application/problem+json` |
| 401 Unauthorized | | Authentication is required to complete the request. See [Authentication](./../authentication.md) for further information. | |
| 429 Too Many Requests | | There are too many logs requests at this time. Try to request logs at a later time. | |

## Examples

### Sample Request

```http
GET /logs/21632?level=Information&durationSeconds=60 HTTP/1.1
Host: localhost:52323
Authorization: MonitorApiKey QmFzZTY0RW5jb2RlZERvdG5ldE1vbml0b3JBcGlLZXk=
```

or

```http
GET /logs/cd4da319-fa9e-4987-ac4e-e57b2aac248b?level=Information&durationSeconds=60 HTTP/1.1
Host: localhost:52323
Authorization: MonitorApiKey QmFzZTY0RW5jb2RlZERvdG5ldE1vbml0b3JBcGlLZXk=
```

### Sample Response

The log statements logged at the Information level or higher for 1 minute is returned as the response body.

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream

event: ProcessRequest
data: Information Agent.RequestProcessor[3]
data: Processing request 353f398a-dc74-4adc-b107-ec35edd09968.

event: ProcessRequest
data: Information Agent.RequestProcessor[3]
data: Processing request eeb18b82-5dfd-49e7-88a3-d0b7cbf2f4bc.

event: ProcessRequest
data: Information Agent.RequestProcessor[3]
data: Processing request 0b7ba879-fa80-4eb8-a87d-408f539952ca.
```

## Supported Runtimes

| Operating System | Runtime Version |
|---|---|
| Windows | .NET Core 3.1, .NET 5+ |
| Linux | .NET Core 3.1, .NET 5+ |
| MacOS | .NET Core 3.1, .NET 5+ |

## Additional Notes

### When to use `pid` vs `uid`

See [Process ID `pid` vs Unique ID `uid`](pidvsuid.md) for clarification on when it is best to use either parameter.