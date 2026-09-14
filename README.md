# SOAR-EDR-Project
A hands-on SOC investigation project demonstrating how to detect, analyze and contain credential-dumping activity using LimaCharlie.

## Workflow Overview

This project implements a SOAR workflow using Tines and LimaCharlie EDR to detect, investigate and respond to suspicious activity.

The workflow is designed to automate the initial notification and response process while keeping the final isolation decision under the investigator's control.

## Workflow Structure 

1. **Threat detection**

   LimaCharlie monitors the endpoint and generates a detection when suspicious activity is identified.

2. **Detection retrieval**

   Tines receive the detection through a webhook and retrieves the relevant alert details from LimaCharlie.

3. **Initial notifications**

   Once the detection is received, Tines sends an alert through Slack and email. The notifications include information such as the hostname, username, process, command line, file path and detection details.

4. **Investigator decision**

   Tines displays a user prompt to the investigator with the details of the detection and asks whether the affected endpoint should be isolated from the network.

5. **Isolation decision**

   If the investigators select YES, Tines sends an HTTP request to LimaCharilie to isolate the endpoint and sends a confirmation message to Slack.
   On the other hand, if the investigators selects NO, Tines sends a Slack notification indicating the isolation was not performed.
<img width="1496" height="936" alt="diagrama" src="https://github.com/user-attachments/assets/937f4e09-8205-438b-b223-7895e356eeae" />

## Detection and Investigation Workflow

Once the alert is received, I open the LimaCharlie link included in the notification to begin the investigation.
<img width="737" height="567" alt="prompt" src="https://github.com/user-attachments/assets/2e98b2a4-6a26-4ee8-9f87-bba99a9b613f" />
<img width="1772" height="496" alt="notificacions" src="https://github.com/user-attachments/assets/ac8952ab-91c6-4883-82f0-280c8cdff8cb" />

The following screenshots show the process activity generated after the execution of 'LaZagne.exe'.

The first process event confirms that 'LaZagne.exe' was executed from the user's 'Downloads' directory with the 'all' argument. Then this process spawned several 'cmd.exe' instances to ecevute assitional commands.
<img width="1481" height="199" alt="analisis1" src="https://github.com/user-attachments/assets/12f7d0ff-ab8a-4f10-9b69-baeedf487bc5" />
<img width="1535" height="838" alt="analisis2" src="https://github.com/user-attachments/assets/169dab9f-c5d6-43d7-8def-b8add9afe465" />

The 'reg.exe' processes use the 'save' command to create copies of the 'SECURITY' and 'SYSTEM' registry hives in the user's temporary directory.
<img width="746" height="680" alt="analisis4" src="https://github.com/user-attachments/assets/3dd0a373-1ac4-4c4b-ab22-60040af8367a" />

The parent-child relationship are confirmed by the 'PROCESS_ID' and 'PARENT_PROCESS_ID' fields in the event. 'LaZagne.exe' is the parent of the 'cmd.exe' processes, while each 'reg.exe' process is a chald of the corresponding 'cmd.exe' process.

The observed activity can be mapped to the MITRE ATT&CK credential accesss tactic. Based on this analysis, I treated the activity as suspicious and isolated the affected endpoint to contain the potential crendential-access activity.
<img width="1765" height="78" alt="isolacions" src="https://github.com/user-attachments/assets/2d8b15b9-9c6c-4cc7-b842-1bf5e12556fc" />
<img width="1791" height="767" alt="isolacionl" src="https://github.com/user-attachments/assets/b99b51ba-b9af-4647-bc5f-7e596f786243" />

## Response Actions

Based on the evidence collected during the investigation, I recommend the following response actions:

- Terminate the LaZagne.exe process and it's related child processes.
- Review the endpoint for additional suspicious files, persistence mechanisms and services.
- Identify whether any credentials may have been exposed.
- Reset potentially compromised local and domain credentials according to the organization's incident-response procedures.
- Return the endpoint to normal operation only after confirming that the system is clean.
