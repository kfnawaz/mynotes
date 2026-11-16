
# Meeting Summary – Tableau SDR Workflow Troubleshooting

## 1. Key Points Discussed
- Investigated Tableau connector workflow failures in the SDR environment.
- Reviewed Atlan application health, container status, proxy configuration, GSM secret retrieval, and Tableau connectivity.
- Discussed recurring application downtime and the need for better error handling, logging, and notifications.

## 2. Items Proved Out
- **GSM Secret Retrieval:** Confirmed the application can access the secret store.
- **Container Health:** Container appeared healthy despite misleading UI status.
- **Proxy Service:** Successfully restarted and confirmed running.
- **Network Diagnosis:** Direct TCP connectivity test timed out, helping isolate the failure.

## 3. Issues Found / Blockers
- **Primary Blocker:** Tableau connector appears to ignore configured HTTPS proxy settings and attempts direct connectivity, resulting in timeouts.
- **Secret Configuration:** Correct GSM private-key path remains unresolved.
- **Application Stability:** Tableau application unexpectedly became inactive, requiring manual redeployment.
- **Error Handling:** Need clearer diagnostics distinguishing application failures from customer environment issues.

## 4. Next Steps

| Action Item | Owner |
|---|---|
| Fix proxy handling in Tableau connector and publish updated image | Connector Team |
| Check Snowflake, Databricks, and other connectors for similar proxy issues | Connector Team |
| Identify and validate correct GSM secret/private-key path | Joint Team |
| Investigate unexpected application downtime using logs | Connector Team |
| Improve error handling, notifications, and troubleshooting documentation | Connector Team |
| Procure updated connector images, preferably together, and rerun workflows | SDR/DataCompass Team |

## 5. Overall Outcome
**Productive troubleshooting session:** Confirmed secret-store access and narrowed the primary failure to connector proxy handling. A new connector image is likely required. Full workflow execution remains blocked pending the proxy fix and correct GSM secret-path configuration.
