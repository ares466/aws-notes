# Amazon Inspector

Amazon Inspector scans EC2 instances, instance OS, and containers for known vulnerabilities or deviations from best practices.

Inspector `assessments` are run periodically (e.g., 15 minutes, 1 hour, 1 day).

Inspector provides a `report of findings` ordered by priority.

Inspector supports a few types of assessments:
- `Network assessment` (agentless or with agent)
- `Host assessment` (agent)

An Inspector `rules package` determines which vulnerabilities and best practices should be analyzed.
- The `network reachability` package (runs with an agent or agentless) analyzes the end-to-end reachability of a system. The network reachability results in findings such as *RecognizedPortWithListener*, *RecognizedPortNoListener*, or *RecognizedPortNoAgent* (used when no agent is available to check listeners), *UnrecognizedPortWithListener*.
- The `CVE` package analyzes against known security vulnerabilities (agent required).
- The `CIS` (Center for Internet Security) Benchmarks package requires an agent.
- The `Security Best Practices` for Amazon Inspector requires an agent.


