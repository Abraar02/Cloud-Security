<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Security Monitoring System

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-monitoring)

---

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_reghtjy)

---

## Introducing Today's Project!

In this project, I will demonstrate how to build a comprehensive AWS security monitoring system that tracks and alerts on secret access by creating a secret in AWS Secrets Manager, enabling CloudTrail to log all API activity, setting up CloudWatch metric filters to detect "GetSecretValue" events, configuring CloudWatch Alarms with SNS email notifications, and testing the entire pipeline to ensure I receive real-time alerts whenever my secret is accessed, ultimately implementing a critical security practice for protecting sensitive credentials and detecting potential security incidents.

### Tools and concepts

Services I used were CloudTrail for capturing AWS API calls, AWS Secrets Manager for storing sensitive information, CloudWatch Logs for centralizing and storing audit logs, CloudWatch Alarms for monitoring metrics and triggering automated responses, and Amazon SNS for delivering email notifications. Key concepts I learnt include how CloudTrail provides a complete audit trail of AWS activities, how metric filters automatically detect specific patterns in logs, how alarms use thresholds to determine when to trigger notifications, the importance of least privilege access through IAM roles, and why targeted pattern-based alerting (CloudWatch) is more practical than comprehensive event notifications (direct CloudTrail SNS) for human-actionable security monitoring.

### Project reflection

This project took me approximately around 90 mins to work through all the steps from initial CloudTrail setup through to configuring alarms and SNS notifications. The most challenging part was understanding the architectural differences between direct CloudTrail SNS notifications and CloudWatch Alarms realizing that CloudTrail sends alerts for every log file delivery (creating notification overload) while CloudWatch intelligently filters for specific patterns, which required thinking critically about why each approach exists and when to use them. It was most rewarding to see the complete monitoring system come together and receive that first email notification confirming the alarm worked, demonstrating how multiple AWS services can be orchestrated to create a practical security monitoring solution.

---

## Create a Secret

AWS Secrets Manager helps you protect secrets, which are passwords, API keys, credentials and sensitive information. Instead of storing important credentials in your code (yikes!) or sharing them via email (double yikes!), you can tuck them safely away in Secrets Manager.
In our project, we're just storing a dummy secret, but in real life, this is where you'd keep database passwords, API keys, and other sensitive information that would cause a major headache if they leaked.

To set up for my project, I created a secret called TopSecretInfo that contains a key-value pair where the key is "The Secret is" and the value is a personal hot take or random secret (for example, "I need 3 coffees a day to function" or "rice is the best carb"), which serves as a dummy secret to demonstrate the monitoring system's capabilities in tracking access to sensitive information stored in AWS Secrets Manager.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_o5p6q7r8)

---

## Set Up CloudTrail

CloudTrail is a monitoring service, it records events that happened in your AWS account, like creating resources, updating a name or setting and accessing secrets in Secrets Manager. A trail tells CloudTrail exactly what activity to record and where to save those recordings. When you create a trail, you're essentially saying "Hey CloudTrail, please keep track of all xyz activities and store the data in this specific location."

CloudTrail events include types like Management, Data, Insights and Network Activity events. The different types of API activities are Read, Write, Exclude AWS KMS events and Exclude Amazon RDS Data API events.

### Read vs Write Activity

Read API activity happens when someone views but doesn't change anything. For example, listing your S3 buckets, describing your EC2 instances, or in our project, viewing (but not changing) metadata about a secret.
Write API activity occurs when changes happen - creating, deleting, modifying resources, or even retrieving the value of a secret (which is what we want to monitor).

---

## Verifying CloudTrail

I retrieved the secret in two ways: First through the AWS Secrets Manager console by navigating to the secret details page and clicking "Retrieve secret value." Second using the AWS CLI in CloudShell by running the `aws secretsmanager get-secret-value` command to fetch the secret value from the terminal.

To analyze my CloudTrail events, I visited the CloudTrail console and navigated to Event history, where I filtered by Event source for "secretsmanager.amazonaws.com." I found GetSecretValue events logged for both my console access and my AWS CLI command, which confirms that CloudTrail successfully captured both secret access attempts. This tells me that CloudTrail is working correctly and recording all API calls to Secrets Manager, providing a complete audit trail of who accessed the secret and when, exactly what we need for security monitoring and compliance.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_s8t9u0v1)

---

## CloudWatch Metrics

CloudWatch Logs is a centralized logging service that collects and aggregates logs from multiple AWS services and applications into one place for unified visibility and analysis. It's important for monitoring because it allows you to search through logs, create custom metrics based on specific patterns or events, set up automated alarms that trigger notifications when certain conditions are met, and enable faster troubleshooting and security incident response by having all your logs accessible in one searchable location.

CloudTrail's Event History is useful for quick, real-time investigations of recent AWS API calls within the last 90 days, while CloudWatch Logs are better for long-term log retention, setting up automated alerts and alarms based on specific event patterns, and applying powerful filtering and analysis tools to trigger proactive security responses.

A CloudWatch metric is a quantifiable measurement that tracks specific events or activities over time, allowing you to monitor patterns and trends in your AWS environment. When setting up a metric, the metric value represents the numerical increment that gets recorded each time the filter detects a matching pattern in your logs, in this case, 1 is added to the counter every time someone accesses the secret. Default value is used when the filter doesn't find any matches during a given time period, so setting it to 0 ensures that periods of no secret access are still recorded as zero rather than appearing blank, giving you a complete visibility into both when access occurred and when it didn't.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_a9b0c1d2)

---

## CloudWatch Alarm

A CloudWatch alarm is a monitoring tool that automatically checks a metric against a defined condition and triggers an action (like sending a notification) when that condition is met. I set my CloudWatch alarm threshold to Greater/Equal to 1 in a 5-minute period, so the alarm will trigger whenever the SecretIsAccessed metric reaches or exceeds 1, meaning any access to the secret will immediately generate an alert.

I created an SNS topic along the way. An SNS topic is a communication channel that acts as a central hub where AWS services can send notifications, and subscribers (like your email address) receive those messages automatically. My SNS topic is set up to deliver email notifications to my inbox whenever the CloudWatch alarm is triggered, so I'll be immediately notified if anyone accesses my secret.

AWS requires email confirmation because it verifies that you actually own and control the email address being added to the SNS topic, ensuring that notifications are only sent to legitimate subscribers who have explicitly requested them. This helps prevent spam, unauthorized access to notifications, and accidental misconfiguration where someone might accidentally subscribe an email address that doesn't belong to them.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_fsdghstt)

---

## Troubleshooting Notification Errors

To test my monitoring system, I retrieved the secret value again from the Secrets Manager console to trigger another GetSecretValue API call that should have been captured by CloudTrail, detected by the metric filter, and flagged by the CloudWatch Alarm to send an email notification. The results were that I did not receive an email notification after waiting several minutes, which indicates that while the individual components (CloudTrail, CloudWatch Logs, metric filter, and alarm) were successfully set up, something in the pipeline is preventing the notification from being delivered likely either the alarm isn't triggering due to timing or metric aggregation issues, the SNS topic isn't properly connected to the alarm, or there's a configuration gap that needs to be troubleshot to complete the end-to-end monitoring system.

Throughout this project, I used five systematic troubleshooting approaches when verifying each component of the monitoring system. First, I checked CloudTrail to ensure it was properly logging API calls and capturing the GetSecretValue events when I accessed the secret through both the console and CLI. Second, I verified Log Delivery by confirming that CloudTrail was successfully sending logs to the CloudWatch log group I created, checking that logs were actually appearing in the CloudWatch console. Third, I tested the Metric Filter by ensuring it correctly detected the "GetSecretValue" pattern in the logs and that the filter was counting secret access events accurately. Fourth, I validated the Alarm Configuration by confirming the threshold was set appropriately (≥1) and that the alarm transitioned to an "In alarm" state when the metric was triggered. Finally, I confirmed SNS Subscription was working by checking my email inbox for the subscription confirmation email from AWS.

I initially didn't receive an email before because the CloudWatch alarm statistic was set to "Average" instead of "Sum," which meant the metric calculation was averaging the number of secret accesses rather than adding them up, so the threshold condition (≥1) was never being met even when the secret was accessed. The key solution was changing the Statistic from Average to Sum in the alarm configuration, which ensures that all occurrences of secret access within the 5-minute period are added together, allowing the alarm to properly trigger when the accumulated count reaches or exceeds the threshold of 1.

---

## Success!

To validate that my monitoring system works, I checked the CloudWatch alarm state and confirmed it transitioned to "ALARM" after manually triggering it with the CLI command. I received an email notification from AWS SNS with the subject line "ALARM: 'Secret is accessed'" in my inbox, which confirmed that the entire monitoring pipeline from CloudTrail capturing the event, to CloudWatch filtering for the GetSecretValue pattern, to the alarm evaluating the metric threshold, to SNS delivering the email notification was functioning correctly end-to-end.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_ageraergearge)

---

## Comparing CloudWatch with CloudTrail Notifications

In a project extension, I enabled direct SNS notifications from CloudTrail and configured it to use the existing SecurityAlarms topic, because this allows me to compare two different notification methods, one that alerts on every log file delivery (CloudTrail SNS) versus one that only alerts when a specific pattern is detected (CloudWatch Alarm) which demonstrates the architectural trade-offs between real-time comprehensive logging notifications and pattern-based intelligent alerting.

After enabling CloudTrail SNS notifications, my inbox was flooded with multiple emails from AWS Notifications as CloudTrail sent a notification for each log file batch it delivered to S3, capturing all management events happening in my AWS account, not just secret access. In terms of the usefulness of these emails, I thought this demonstrates why the CloudWatch Alarm approach is significantly more practical and targeted; CloudWatch filters for only the specific events I care about (GetSecretValue), while direct CloudTrail notifications generate alerts for every single AWS activity, making them overwhelming for human monitoring but better suited for automated security tools that can process and correlate all account activity programmatically.

![Image](http://learn.nextwork.org/joyful_navy_gentle_albatross/uploads/aws-security-monitoring_d7e8f9g0)

---
**Author:** MOHAMMAD ABRAAR  
**Email:** mohammadabraar2@gmail.com
---
