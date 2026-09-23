# SOAR-EDR Investigation: Credential Dumping with LaZagne

Hands-on SOC investigation of credential-dumping activity using LimaCharlie EDR and a Tines SOAR workflow.  
Focus: detection, analyst reasoning and containment decision, not tool setup.

## Scenario Overview

In a controlled Windows lab, LaZagne was executed to dump credentials from the SAM and SECURITY hives. LimaCharlie EDR detected the suspicious activity and sent the alert to Tines, which orchestrated notifications, analyst approval and endpoint isolation.

As the SOC analyst, I:

- Received an automated alert via Slack and email with key context (hostname, user, process, command line).  
- Investigated the process tree and registry-hive access in LimaCharlie.  
- Mapped the activity to MITRE ATT&CK (Credential Access).  
- Decided to isolate the endpoint to contain potential credential compromise.  
- Defined response actions for credential reset and further hunting.

**Key outcomes**

- End-to-end SOAR workflow: detection → alert → investigation → containment.  
- Demonstrated analyst-in-the-loop automation (human approval before isolation).  
- Practical experience with EDR triage and incident-response decision-making.

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | How it appears in this case |
|---|---|---|---|
| Credential Access | OS Credential Dumping: Security Account Manager | T1003.002 | reg.exe used to save SECURITY and SAM hives. |
| Credential Access | OS Credential Dumping: Cached Domain Credentials | T1003.005 | Access to SYSTEM/SECURITY to extract cached credentials. |
| Defense Evasion | Obfuscated Files or Information | T1027 | LaZagne executed from the user's Downloads folder. |

## SOAR Workflow (Tines + LimaCharlie)

The project implements a SOAR workflow using Tines and LimaCharlie EDR to detect, investigate and respond to suspicious activity. The workflow automates notifications and response, while keeping the final isolation decision under the investigator's control.

1. **Threat detection**  
   LimaCharlie monitors the endpoint and generates a detection when suspicious activity is identified (e.g., LaZagne execution and registry-hive access).

2. **Detection retrieval**  
   Tines receives the detection through a webhook and retrieves the relevant alert details from LimaCharlie.

3. **Initial notifications**  
   Tines sends an alert through Slack and email. The notifications include hostname, username, process name, command line, file path and detection details.

4. **Investigator decision**  
   Tines displays a prompt to the investigator with the detection context and asks whether the affected endpoint should be isolated from the network.

5. **Isolation decision**  
   - If the investigator selects **YES**, Tines sends an HTTP request to LimaCharlie to isolate the endpoint and posts a confirmation message to Slack.  
   - If the investigator selects **NO**, Tines sends a Slack notification indicating that isolation was not performed.

<img width="1496" height="936" alt="diagrama" src="https://github.com/user-attachments/assets/937f4e09-8205-438b-b223-7895e356eeae" />

## Detection and Investigation

Once the alert is received, I open the LimaCharlie link included in the notification to begin the investigation.

<img width="737" height="567" alt="prompt" src="https://github.com/user-attachments/assets/2e98b2a4-6a26-4ee8-9f87-bba99a9b613f" />

<img width="1772" height="496" alt="notificacions" src="https://github.com/user-attachments/assets/ac8952ab-91c6-4883-82f0-280c8cdff8cb" />

The following screenshots show the process activity generated after the execution of `LaZagne.exe`.

The first process event confirms that `LaZagne.exe` was executed from the user's `Downloads` directory with the `all` argument. This process then spawned several `cmd.exe` instances to execute additional commands.

<img width="1481" height="199" alt="analisis1" src="https://github.com/user-attachments/assets/12f7d0ff-ab8a-4f10-9b69-baeedf487bc5" />

<img width="1535" height="838" alt="analisis2" src="https://github.com/user-attachments/assets/169dab9f-c5d6-43d7-8def-b8add9afe465" />

The `reg.exe` processes use the `save` command to create copies of the `SECURITY` and `SYSTEM` registry hives in the user's temporary directory.

<img width="746" height="680" alt="analisis4" src="https://github.com/user-attachments/assets/3dd0a373-1ac4-4c4b-ab22-60040af8367a" />

The parent–child relationships are confirmed by the `PROCESS_ID` and `PARENT_PROCESS_ID` fields in the events:

- `LaZagne.exe` is the parent of the `cmd.exe` processes.  
- Each `reg.exe` process is a child of the corresponding `cmd.exe` process.

This chain (LaZagne → cmd → reg.exe saving SAM/SECURITY/SYSTEM) is consistent with credential-dumping behavior. I mapped the activity to the MITRE ATT&CK **Credential Access** tactic and treated it as a **high-confidence suspicious incident**.

Based on this analysis, I decided to **isolate the affected endpoint** to contain the potential credential-access activity.

<img width="1765" height="78" alt="isolacions" src="https://github.com/user-attachments/assets/2d8b15b9-9c6c-4cc7-b842-1bf5e12556fc" />

<img width="1791" height="767" alt="isolacionl" src="https://github.com/user-attachments/assets/b99b51ba-b9af-4647-bc5f-7e596f786243" />

## Response Actions

Based on the evidence collected during the investigation, I recommend:

- **Terminate `LaZagne.exe` and its related child processes** (`cmd.exe`, `reg.exe`) on the affected endpoint.  
- **Review the endpoint** for additional suspicious files, persistence mechanisms and services.  
- **Identify potentially exposed credentials** (local accounts, domain accounts cached on the host).  
- **Reset compromised or potentially compromised credentials** according to the organization's incident-response procedures.  
- **Return the endpoint to normal operation only after confirming** that the system is clean and no further suspicious activity is observed.

## Learning Outcomes

This project demonstrates practical experience with:

- EDR-based detection and investigation of credential-dumping activity.  
- SOAR workflow design with human-in-the-loop approval.  
- Slack and email alerting for SOC analysts.  
- Endpoint isolation as a containment action.  
- Mapping host-level evidence to MITRE ATT&CK techniques.  
- Incident-response decision-making in a SOC context.
