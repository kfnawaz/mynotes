# Meeting Summary

## 1. Production Snowflake / LRI Workflow Issue

The team compared the failing newer **LRI Snowflake workflow** with the older **CCB Snowflake workflow** that is working successfully.

### Key Findings

- The issue does **not appear to be related to S3, Snowflake connectivity, or the JPMC Edge/agent side**.
- The newer LRI workflow is taking a **different execution path** than the older working workflow.
- The older workflow appears to run correctly through the expected **secure agent / self-deployed runtime path**.
- The newer workflow, even though configured similarly, appears to pick up a **credential GUID** and routes through a different template path.
- A newly created workflow using custom OAuth still followed the same unexpected path.
- This ruled out the theory that the first workflow was misconfigured only because of an earlier key-pair selection.
- Owen will investigate how to make the credential GUID blank, or how to make the workflow template treat the credential as blank so it uses the expected agent path.

---

## 2. UAT Agent / k3s Instability Issue

The team investigated why the **UAT agent** had been offline or unstable for over a week.

### Key Findings

- Offline cron jobs were taking much longer than expected, sometimes over an hour.
- This delay likely caused health check failures because the agent was not reporting back within the expected window.
- The team noticed a large number of `fuse-overlayfs` defunct/zombie processes.
- The parent process appeared to be related to the `k3s server`.
- CPU, memory, and disk availability looked acceptable.
- The concern shifted toward process buildup, k3s/rootless behavior, or stale/zombie process handling.
- The team restarted the k3s service and planned to monitor whether the zombie `fuse-overlayfs` processes reappear.
- If k3s does not recover cleanly, the next step is to reboot the UAT VSI/server.
- Rebooting is considered a temporary workaround; the real root cause still needs to be identified if the processes continue accumulating.

---

## Action Items

| Owner | Action Item |
|---|---|
| Owen | Investigate why the LRI workflow is picking up a credential GUID and taking a different template path. |
| Owen | Determine whether the workflow can be forced to use a blank credential/default path. |
| Team | Monitor whether k3s restarts cleanly in UAT. |
| Team | If k3s does not recover, reboot the UAT server. |
| Team | Track whether `fuse-overlayfs` zombie processes reappear after restart/reboot. |
| Team | Review successful offline cron logs to understand why processing is taking so long. |
| Team | Reduce unnecessary cron/job history if it is contributing to clutter or slow startup. |

---

## Overall Conclusion

The production issue appears to be a **workflow-template / credential-routing problem**.

The UAT issue appears to be an **agent infrastructure stability problem**, likely tied to **k3s/rootless process buildup and slow offline cron health checks**.