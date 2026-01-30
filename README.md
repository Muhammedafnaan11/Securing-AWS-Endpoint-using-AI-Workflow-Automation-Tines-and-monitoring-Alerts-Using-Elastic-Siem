This lab replicates a real SOC workflow from collecting endpoint logs to correlating events in Elastic SIEM and using AI automation in Tines for alert summarization.

🎯 Objective
I built this to understand how a cloud-hosted Windows endpoint can be monitored, detected, and automated using Elastic SIEM and Tines.

The goal was simple:
Make an admin login happen → detect it → push it to automation → receive a clean AI-generated alert in my inbox.

Basically, simulate a real detection-to-notification cycle.

🧠 Skills Demonstrated

Deployed Windows Server 2025 on AWS EC2

Installed Elastic Agent with Elastic Defend (EDR enabled)

Forwarded Windows security logs to Elastic SIEM

Created a custom detection rule for privileged logins (Event ID 4672)

Integrated Elastic SIEM with Tines using a webhook

Built an AI workflow to summarize alerts automatically



Simulated an admin login to validate the entire pipeline

🛠️ Setup Overview

I launched a Windows Server 2025 instance in AWS EC2 and configured secure RDP access.
<img width="1915" height="958" alt="Screenshot 2026-01-28 104216" src="https://github.com/user-attachments/assets/d2948f9a-2f69-4a0f-988d-16a71e74e5ff" />



After deployment, I installed Elastic Agent and enrolled it into Elastic Cloud. Once the agent showed as healthy, I enabled Elastic Defend to get full endpoint visibility.

Logs from the server started flowing directly into Elastic SIEM.
