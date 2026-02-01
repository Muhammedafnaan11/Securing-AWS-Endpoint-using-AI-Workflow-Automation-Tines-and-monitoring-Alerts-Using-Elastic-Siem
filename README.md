AWS • Elastic • Tines • Windows Server • AI
This lab recreates a real SOC-style workflow — from collecting endpoint logs to correlating events in Elastic SIEM and automating alert handling using AI in Tines
🎯 Objective
I built this lab to see how a cloud-hosted Windows endpoint can actually be monitored and protected using Elastic SIEM combined with AI-driven automation in Tines.
The goal was to simulate a full detection and response flow — starting from endpoint log collection all the way to automated alert handling — just like it would run inside a real SOC environment.
🧠 Skills Demonstrated
•	Deployed and configured Windows Server 2025 on AWS EC2
•	Installed and configured Elastic Agent with Elastic Defender (EDR) for endpoint monitoring
•	Forwarded Windows security logs into Elastic SIEM
•	Created a custom detection rule for privileged admin sign-ins
•	Integrated Elastic SIEM with Tines using a webhook for automated alert handling
•	Designed a Tines workflow using AI to summarize alerts and send email notifications
•	Simulated an admin login to trigger the detection and validate the automation pipeline.


🛠️ Setup Walkthrough
I began by launching a Windows Server 2025 instance in AWS EC2 to act as the monitored endpoint.
After installing the Elastic Agent, I configured it to forward Windows security event logs into Elastic SIEM. Inside Elastic, I created detection logic to identify privileged login activity.
💡 If you don’t already have an AWS account, set that up first before starting the lab.
Once alerts were being generated in Elastic SIEM, I connected Tines to process them automatically. Tines receives the alert data, generates a summarized version using AI, and sends it directly to my email — allowing me to test the full detection-to-notification workflow end-to-end.


1️⃣ Launch AWS EC2 Instance
Initially, I created a Windows Server 2025 instance in AWS EC2 to serve as the monitored endpoint USING Launch instance option, as you see below.
 <img width="940" height="470" alt="image" src="https://github.com/user-attachments/assets/40dfd01e-b6fb-4f10-a031-a379c08fe615" />

The instance was named Server Test, and selected the Microsoft Windows Server 2025 Base AMI from the Quick Start options.
 
This AMI is free-tier eligible and provided directly by Amazon.
For compute resources, we selected the m7i-flex.large instance type (2 vCPU, 8 GB RAM).
 
I then generated a new key pair called Test Purpose for secure RDP access and downloaded the .pem file locally.
 
 
In the network configuration, I allowed inbound RDP access restricted only to any address.
 

 
 
Storage was configured with a 30 GB gp3 SSD. Encryption was left disabled.
After reviewing the configuration, I launched the instance.
Once it was running, I navigated to:
Actions → Security → Get Windows Password
 

I uploaded the previously downloaded .pem key file and decrypted the Administrator credentials to access the server via RDP.
 
 

Once I get the credentials, I plan to test the machine by logging in 
To do this, I downloaded the RDP file and opened it using the Remote Desktop Connection application 
 
 
 
I  entered the password, which I got while decrypting the private key
 
Click Yes
 
I  have now successfully connected to the cloud desktop hosted by Amazon, aka EC2
The next phase is the Elastic Siem Setup
2️⃣ Install Elastic Agent & Integrate Elastic Defender
 
 
With the Windows instance running, I proceeded to install the Elastic Agent to begin log collection.
Inside Elastic Cloud, we created a new deployment and ensured Elastic SIEM features were included.
Then I navigated to:
Assets → Fleet → Agents → Add Agent
 
 
 
I created a new agent policy named   Policy7 and confirmed that the System integration was included.
After selecting Windows as the platform, I copied the provided PowerShell installation command.
 
On the Windows Server, I opened PowerShell as Administrator and executed the installation command.
 
 
This downloaded and installed the Elastic Agent, enrolling it into Fleet using the enrollment token.
 
Once installation was complete, I verified in the Fleet dashboard that the agent status showed as Healthy, confirming successful communication.
 

Next, I added the Elastic Defend integration to the Windows Test
 
 
 
I configured it in Complete EDR mode, enabling full endpoint telemetry and real-time monitoring.
At this point, the AWS Windows endpoint was actively monitored by Elastic.
 

3. Create Elastic SIEM Rule & Connect to Tines
To send alerts into Tines, I created a webhook connector in Elastic.
In Elastic:
Management → Stack Management → Connectors → Create Connector
 
I selected Webhook as the connector type and named it:
 
Tines_Webhook_Admin_Sign-in
The Tines webhook URL was pasted into the configuration.
Authentication was set to None, and the HTTP method was set to POST.
Next, I navigated to:
Security → Rules → Create New Rule
 

I selected a Custom Query Rule and used the following query:
event.code: "4672"
This event ID corresponds to special privileges assigned during a logon (admin-level activity).
The rule was named:
Admin_Log-in_Detection
 

Severity was set to High.
Under Rule Actions, I selected the Tines webhook connector and configured it to run per rule execution.
The following JSON payload was sent to Tines:
{
  "rule_name" : "{{context.rule.name}}",
  "description" : "{{context.rule.description}}"
}
 

4 Create Workflow Automation in Tines
I logged in to Tines with the link https://www.tines.com/
Next, I built an automated workflow in Tines to process incoming alerts.
Inside Tines, I created a new Story named Admin_Sign-in_Detection.
 
From the action panel, I added a Webhook action to serve as the entry point for alerts sent from Elastic SIEM.
 
Then, I added an AI Action connected to the Webhook and renamed it Summarise webhook data.
 
The prompt used was:
•	Summarise the following data in one sentence.
•	Provide recommended investigation steps for the analyst.
•	Format the output into bullet points for readability in an email.
The webhook payload was referenced dynamically using:

{{ webhook_action }}
Finally, I added a Send Email action connected to the AI output.
 

The workflow logic became:
1️⃣ Webhook – receives alert from Elastic
2️⃣ AI Action – summarises and formats the alert
3️⃣ Send Email – delivers the summarised alert to my inbox
This completed the automation pipeline.

5️⃣ Simulate & Validate the Workflow
With everything configured, I tested the full pipeline.
I logged into the Windows Server via RDP using administrator credentials.
The Elastic Agent captured the event and forwarded it to Elastic SIEM.
The detection rule triggered immediately and generated an alert in the Security → Alerts panel.
 

Because the rule was connected to the Tines webhook, the alert payload was automatically sent to the workflow.
Within moments, I received an email notification titled:
"New Alert!"
 
The email included:
•	A summarised explanation of the detection
•	Suggested investigation steps (review source IP, validate login time, correlate events)
•	Recommendations for monitoring or mitigation
This confirmed that the full workflow operated as designed:
Log ingestion → Detection → Webhook → AI processing → Email notification

Closing Notes
This project demonstrates how a detection event can move from endpoint telemetry to an AI-generated alert summary within minutes.
Next, I plan to expand the lab by adding more detection rules, integrating threat intelligence, and enhancing automation for deeper response capabilities.


















































































