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
<img width="940" height="470" alt="image" src="https://github.com/user-attachments/assets/daefaf8a-e0e7-4690-aac0-b7f474ba99ea" />

The instance was named Server Test, and selected the Microsoft Windows Server 2025 Base AMI from the Quick Start options.
<img width="940" height="408" alt="image" src="https://github.com/user-attachments/assets/f0cecac7-cff0-42b4-8416-ef8dfcea9323" />

This AMI is free-tier eligible and provided directly by Amazon.
For compute resources, we selected the m7i-flex.large instance type (2 vCPU, 8 GB RAM).

<img width="940" height="264" alt="image" src="https://github.com/user-attachments/assets/74a70e4e-730c-43c9-b469-a9c40be1c1b1" />

I then generated a new key pair called Test Purpose for secure RDP access and downloaded the .pem file locally

<img width="940" height="210" alt="image" src="https://github.com/user-attachments/assets/039360cf-98cb-4847-bf0f-253c978e0918" />

<img width="940" height="945" alt="image" src="https://github.com/user-attachments/assets/6f063db3-0068-44ae-9421-040517102ab5" />


In the network configuration, I allowed inbound RDP access restricted only to any address.

<img width="940" height="567" alt="image" src="https://github.com/user-attachments/assets/c139cba5-1983-4ab8-8ad9-d3fad1f5abb9" />


<img width="940" height="1100" alt="image" src="https://github.com/user-attachments/assets/78867701-ee24-4d9a-8c16-d89b39f1ca3a" />


<img width="940" height="127" alt="image" src="https://github.com/user-attachments/assets/bca7278f-026c-42fb-ba99-4aa91fb1b5f8" />


Storage was configured with a 30 GB gp3 SSD. Encryption was left disabled.
After reviewing the configuration, I launched the instance.
Once it was running, I navigated to:
Actions → Security → Get Windows Password


<img width="940" height="268" alt="image" src="https://github.com/user-attachments/assets/db6c1a6d-446d-4e80-b374-a3d48df24236" />


I uploaded the previously downloaded .pem key file and decrypted the Administrator credentials to access the server via RDP.

<img width="940" height="505" alt="image" src="https://github.com/user-
attachments/assets/86b1f906-803b-4e3e-b15a-1f4ab5bcf662" />


<img width="940" height="381" alt="image" src="https://github.com/user-attachments/assets/9e6ba97e-7790-488d-9605-5ccb2f1ad51c" />

Once I get the credentials, I plan to test the machine by logging in 
To do this, I downloaded the RDP file and opened it using the Remote Desktop Connection application 


<img width="940" height="162" alt="image" src="https://github.com/user-attachments/assets/49a5dc46-f824-4c57-bb30-d1a4fd6f8f7f" />

<img width="940" height="210" alt="image" src="https://github.com/user-attachments/assets/523b3422-ca17-4be0-adaa-8d2e69dfd9d0" />


<img width="679" height="734" alt="image" src="https://github.com/user-attachments/assets/1e275d4d-790c-4777-a004-5e3d283329a0" />

I  entered the password, which I got while decrypting the private key

<img width="869" height="931" alt="image" src="https://github.com/user-attachments/assets/a7b80d54-2195-4dfc-b1e2-993a453934be" />

Click Yes


<img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/9d1db075-a887-4108-ba4a-a0dfa354190e" />

I  have now successfully connected to the cloud desktop hosted by Amazon, aka EC2
The next phase is the Elastic Siem Setup




2️⃣ Install Elastic Agent & Integrate Elastic Defender



<img width="940" height="549" alt="image" src="https://github.com/user-attachments/assets/56621814-7223-4b66-97c6-424dd1253d56" />

<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/364253d7-fc26-454e-bf30-97891effd065" />

With the Windows instance running, I proceeded to install the Elastic Agent to begin log collection.
Inside Elastic Cloud, we created a new deployment and ensured Elastic SIEM features were included.
Then I navigated to:
Assets → Fleet → Agents → Add Agent


<img width="694" height="1043" alt="image" src="https://github.com/user-attachments/assets/5df39bc3-e342-463d-9e38-591ad0989dee" />

<img width="940" height="497" alt="image" src="https://github.com/user-attachments/assets/e553dde1-410d-41db-b156-97bf269616ed" />


<img width="940" height="946" alt="image" src="https://github.com/user-attachments/assets/240685be-0fdc-4918-87ef-c45f19ed9793" />


I created a new agent policy named   Policy7 and confirmed that the System integration was included.
After selecting Windows as the platform, I copied the provided PowerShell installation command.



<img width="940" height="961" alt="image" src="https://github.com/user-attachments/assets/0555c458-2345-45f9-a1fa-b73aa8d035dc" />


On the Windows Server, I opened PowerShell as Administrator and executed the installation command.

<img width="940" height="517" alt="image" src="https://github.com/user-attachments/assets/d6fd8dd0-5a72-451f-9602-314d1d719441" />

<img width="940" height="538" alt="image" src="https://github.com/user-attachments/assets/a7000563-952a-4f03-9a83-3e940f57398d" />


This downloaded and installed the Elastic Agent, enrolling it into Fleet using the enrollment token.

<img width="940" height="541" alt="image" src="https://github.com/user-attachments/assets/d578c5ae-c507-4e5f-bcc5-283bf81bd8fc" />

Once installation was complete, I verified in the Fleet dashboard that the agent status showed as Healthy, confirming successful communication.


<img width="940" height="479" alt="image" src="https://github.com/user-attachments/assets/18806ddd-fb4b-4bb6-88ad-7da94574c235" />

Next, I added the Elastic Defend integration to the Windows Test

<img width="448" height="248" alt="image" src="https://github.com/user-attachments/assets/10dd2ebb-8e9f-4d31-a4ca-f0b6c5bc874c" />


<img width="940" height="602" alt="image" src="https://github.com/user-attachments/assets/7584b419-0009-4e67-be35-43dc84cb2eed" />


<img width="940" height="365" alt="image" src="https://github.com/user-attachments/assets/8ba900f4-5790-49b5-bd4e-2fbb9aeae84a" />

I configured it in Complete EDR mode, enabling full endpoint telemetry and real-time monitoring.
At this point, the AWS Windows endpoint was actively monitored by Elastic.

<img width="940" height="470" alt="image" src="https://github.com/user-attachments/assets/435fb4a1-ba8a-48a7-94e5-bd967a9b0cb1" />

3. Create Elastic SIEM Rule & Connect to Tines
To send alerts into Tines, I created a webhook connector in Elastic.
In Elastic:
Management → Stack Management → Connectors → Create Connector


<img width="940" height="963" alt="image" src="https://github.com/user-attachments/assets/bfa8fd3a-471e-4d08-99bc-090f9ba4f048" />


I selected Webhook as the connector type and named it:

<img width="940" height="447" alt="image" src="https://github.com/user-attachments/assets/72695b95-9800-4894-a217-4baa6aa0662b" />


Tines_Webhook_Admin_Sign-in
The Tines webhook URL was pasted into the configuration.
Authentication was set to None, and the HTTP method was set to POST.
Next, I navigated to:
Security → Rules → Create New Rule

<img width="940" height="288" alt="image" src="https://github.com/user-attachments/assets/d221867c-b2dd-4628-aae4-2d564d28c3d9" />


I selected a Custom Query Rule and used the following query:
event.code: "4672"
This event ID corresponds to special privileges assigned during a logon (admin-level activity).
The rule was named:
Admin_Log-in_Detection

<img width="940" height="676" alt="image" src="https://github.com/user-attachments/assets/aa334714-c4fe-49e4-8df2-2ac0ea673377" />



Severity was set to High.
Under Rule Actions, I selected the Tines webhook connector and configured it to run per rule execution.
The following JSON payload was sent to Tines:
{
  "rule_name" : "{{context.rule.name}}",
  "description" : "{{context.rule.description}}"
}



<img width="940" height="644" alt="image" src="https://github.com/user-attachments/assets/f4b44fc2-93b0-4157-bd46-46ac27183e70" />



4 Create Workflow Automation in Tines
I logged in to Tines with the link https://www.tines.com/
Next, I built an automated workflow in Tines to process incoming alerts.
Inside Tines, I created a new Story named Admin_Sign-in_Detection.

<img width="940" height="391" alt="image" src="https://github.com/user-attachments/assets/f5899151-0298-4d70-916a-83b8ba680721" />


From the action panel, I added a Webhook action to serve as the entry point for alerts sent from Elastic SIEM.

<img width="940" height="391" alt="image" src="https://github.com/user-attachments/assets/cf545b1a-8599-4c91-80be-7a30069e8f46" />

Then, I added an AI Action connected to the Webhook and renamed it Summarise webhook data.

<img width="940" height="372" alt="image" src="https://github.com/user-attachments/assets/80218b4d-c708-4412-859e-d568d95d02d8" />



The prompt used was:
•	Summarise the following data in one sentence.
•	Provide recommended investigation steps for the analyst.
•	Format the output into bullet points for readability in an email.
The webhook payload was referenced dynamically using:

{{ webhook_action }}
Finally, I added a Send Email action connected to the AI output.


<img width="940" height="619" alt="image" src="https://github.com/user-attachments/assets/f8e0d753-784e-475f-b2d7-f2ecda5365eb" />

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


<img width="940" height="413" alt="image" src="https://github.com/user-attachments/assets/c3db3685-75de-4f19-839f-421c3238a112" />


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








