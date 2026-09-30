# Alert Retrieval

Retrieves active alerts from AlertManager instances (local and remote spoke clusters) so the adapter can translate them into AgenticRun resources.

## Behavioral Rules

### Local AlertManager

1. The adapter SHALL query the AlertManager API and return the set of currently active alerts from the AlertManager v2 API.
2. The adapter SHALL request only active, non-silenced, non-inhibited alerts so suppressed alerts are never processed.
3. The adapter SHALL authenticate using the pod's ServiceAccount bearer token, re-reading the token file on every call to handle rotation.
4. The adapter SHALL trust the OpenShift service CA certificate (`service-ca.crt`) for TLS verification.
5. When the ServiceAccount token file is not present, the adapter SHALL return an error indicating the token could not be loaded.
6. When no alerts are firing, the adapter SHALL return an empty list and no error.
7. When AlertManager is unreachable, the adapter SHALL return an error indicating the service could not be contacted.
8. When AlertManager rejects the request due to insufficient permissions or an invalid token, the adapter SHALL return an authentication or authorization error.

### Remote AlertManager (Spoke Clusters)

9. The adapter SHALL retrieve active, non-silenced, non-inhibited alerts for every configured spoke target by querying the remote AlertManager endpoint from the spoke's credential Secret (`alertmanager-url` data value).
10. The adapter SHALL authenticate every remote request with the bearer token from the spoke's credential Secret (`token` data value).
11. The adapter SHALL validate the TLS certificate of a remote AlertManager using the PEM-encoded CA bundle from the credential Secret (`ca-bundle` data value). TLS certificate validation SHALL NOT be disabled.
12. When a remote AlertManager endpoint URL does not use the `https` scheme, the adapter SHALL reject the endpoint before sending a bearer token.
13. When a remote AlertManager request fails, the adapter SHALL return the API client error with retrieval context.
13a. [PLANNED] Errors returned for non-2xx responses SHALL identify the HTTP status without exposing the response body. The adapter currently wraps the API client error directly; this redaction guarantee still needs verification or enforcement.
14. When the remote AlertManager endpoint is unreachable, the adapter SHALL return an error identifying remote AlertManager retrieval.
15. When the remote AlertManager rejects the bearer token, the adapter SHALL return an authentication or authorization error for that spoke target.
16. When a remote endpoint presents a certificate not trusted by the credential Secret's CA bundle, the adapter SHALL fail with a TLS validation error.

### Logging

17. After a target cycle completes, the adapter SHALL log the target and total, skipped, and created alert counts at Info level. Creation events SHALL include alert details at Info level; filter skips SHALL be logged at Debug level. The initial cycle SHALL use the same logging rules. A cycle that exits early on error or cancellation may have no totals log.
18. When alert retrieval fails during the initial reconcile cycle, the adapter SHALL log the error; the next poll retries.
19. When `AgenticOLSConfig.spec.suspended` is true, the adapter SHALL not fetch alerts and SHALL log that it is suspended.

## Configuration Surface

| Field | Default | Description |
|---|---|---|
| `ALERTMANAGER_URL` | `https://alertmanager-main.openshift-monitoring.svc:9094` | Local AlertManager endpoint (env var) |
| Spoke credential Secret `alertmanager-url` | (none) | Remote AlertManager endpoint per spoke |
| Spoke credential Secret `token` | (none) | Bearer token for remote AlertManager |
| Spoke credential Secret `ca-bundle` | (none) | PEM CA bundle for remote TLS validation |
