# Threat Detection with GuardDuty

## Introducing Today's Project!

### Tools and concepts

The services I used were AWS GuardDuty, CloudFormation, EC2, S3, CloudFront, CloudShell, and Malware Protection for S3. Key concepts I learnt include security vulnerabilities like SQL injection (bypassing authentication with malicious queries like ' or 1=1;--), command injection (executing unauthorized code through unsanitized inputs), and credential exfiltration (stealing EC2 IAM credentials from instance metadata); threat detection through GuardDuty's machine learning-based anomaly detection that identifies suspicious credential usage and malware in S3 buckets; AWS security best practices including input sanitization, securing instance metadata, using temporary IAM credentials, and monitoring unusual access patterns; and infrastructure management with CloudFormation for automated deployment and proper resource cleanup to avoid costs.

### Project reflection

This project took me approximately 90 minutes to complete. The most challenging part was understanding the command injection exploit and how the JavaScript code manipulated the EC2 instance metadata service to extract IAM credentials and expose them in a publicly accessible JSON file - wrapping my head around how unsanitized input could lead to such a severe security breach required careful analysis of each command component. It was most rewarding to see GuardDuty successfully detect the unauthorized credential usage in real-time, confirming that the UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS finding appeared after using the stolen credentials from CloudShell, which demonstrated how AWS's machine learning-powered threat detection can identify anomalous behavior and protect against real-world attack scenarios like credential theft and data exfiltration.RetryClaude can make mistakes. Please double-check responses.

I completed this project to gain hands-on experience with AWS security services, particularly GuardDuty's threat detection capabilities, and to understand common web application vulnerabilities like SQL injection and command injection from both an attacker's and defender's perspective. This project absolutely met my goals - it provided practical exposure to real-world security scenarios, teaching me how attackers exploit vulnerabilities to steal credentials and exfiltrate data, while demonstrating how GuardDuty's machine learning algorithms detect and alert on suspicious activity. The step-by-step approach of deploying infrastructure with CloudFormation, executing attacks, and then analyzing GuardDuty findings gave me a comprehensive understanding of the entire security lifecycle. Most importantly, I learned valuable AWS security best practices like input sanitization, securing instance metadata, and the critical importance of monitoring credential usage patterns. 

---

## Project Setup

To set up for this project, I deployed a CloudFormation template that launches multiple resources that make up a web application. The three main components are EC2 instance, Networking, CloudFront Distribution. 

The web app deployed is called OWASP Juice Shop which is used widely for security training

GuardDuty is a Threat Detection service provided by AWS that uses machine learning to look for unusal activity in yout network traffic and CloudTrail activity logs.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_n1o2p3q4)

---

## SQL Injection

The first attack I performed on the web app is SQL injection, which means injecting malicious query or SQL code into an application database to bypass. SQL injection is a security risk because it helps us gain break into web application and gain admin access or try and steal sensitive data from the database



My SQL injection attack involved using a simple query that manipulates to always evaluate the query to true. 

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_h1i2j3k4)

---

## Command Injection

Next, I used command injection which is security vulnerability where the web server mistakenly executes your commands when it should merely be storing it. The Juice Shop web app is vulnerable to this because there is no proper input sanitization

To run command injection, I will input vulnerable instruction or code. The script make sures that server fails to properly validate this input and treat it as legitimate code to run.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_t3u4v5w6)

---

## Attack Verification

To verify the attack's success, we try to access the json file location. The credentials page showed me all the essential information about my AWS access credentials

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_x7y8z9a0)

---

## Using CloudShell for Advanced Attacks

The attack continues in CloudShell, because we're trying to get access to private data using stolen credentials.

In CloudShell, I used wget to download the json file from the web application to our local cloudshell environment. Next, I ran a command using cat and jq to display and process the info for easier reading

We're creating a new profile in CloudShell, called stolen, to use the stolen credentials we've extracted from our attack on the web application.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_j9k0l1m2)

---

## GuardDuty's Findings

It took around 15 minutes

GuardDuty's finding was called UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.InsideAWS which means someone who shouldn't have access to our AWS environment got in. Anomaly detection was used because the algorithm picked up on an unsual use of your EC2 instance's credentials

GuardDuty's detailed finding reported that Credentials for the EC2 instance role GuardDuty-Project-MohammadAbraar-TheRole-65dKlJ3wgF88 were used from a remote AWS account.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_v1w2x3y4)

---

## Extra: Malware Protection

For my project extension, I enabled Malware Protection for S3 in GuardDuty

To test Malware Protection, I uploaded EICAR test file which is a harmless file that is programmed to make antivirus software think it's malware. The uploaded file won't actually cause damage because it's harmless and used for testing purposes

Once I uploaded the file, GuardDuty instantly triggered that the uploaded file has a malware. This started a malware scan on my S3 object arn:aws:s3:::guardduty-project-mohammadabraar-thesecurebucket-1e2x5sgscq2l/EICAR-test-file.txt which detected a security risk EICAR-Test-File (not a virus).

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-guardduty_sm42x3y4)

---
**Author:** MOHAMMAD ABRAAR  
**Email:** mohammadabraar2@gmail.com
