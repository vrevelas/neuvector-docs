---
title: Response Rules
taxonomy:
    category: docs
---

### Policy: Response Rules
Response Rules provide a flexible, customizable rule engine to automate responses to important security events. Triggers ca include Security Events, Vulnerability Scan results, CIS Benchmarks, Admission Control events and general Events. Actions include container quarantine, webhooks, and suppression of alerts. 

![ResponseRules](response1.png)

Creating a new Response Rule using the following:
+ Group. A rule will apply to a Group. Please see the section Run-Time Security Policy -> Groups for more details on Groups and how to create a new one if needed.
+ Category. This is the type of event, such as Security Event, or CVE vulnerability scan result.
+ Criteria. Specify one or more criteria. Each Category will have different criteria which can be applied. For example, by the event name, severity, or minimum number of high CVEs.
+ Action. Select one or more actions. Quarantine will block all network traffic in/out of a container. Webhook requires that a webhook endpoint be defined in Settings -> Configuration. Suppress log will prevent this event from being logged in Notifications.

![NewResponseRules](newrule1.png)

<Strong>IMPORTANT</Strong>  All Response Rules are evaluated to determine if they match the condition/criteria. If there are multiple rule matches, each action(s) will be performed. This is different than the behavior of Network Rules, which are evaluated from top to bottom and only the first rule which matches will be executed.

Additional events and actions will continue to be added by NeuVector in future releases.

### Detailed Configuration for Response Rules



#### Using Multiple Criteria in a Single Rule
The matching logic for multiple criteria in one response rule is:
+ For different criteria types  (e.g. name:Network.Violation, name:Process.Profile.Violation) within a single rule, apply 'and'

 Quarantine – container is quarantined
 Webhook - a webhook log generated
 suppress-log – log is suppressed - both syslog and webhook log

Note2: Action and Event parameters are mandatory; other parameters can be empty to match broader conditions.
 Click "insert to top" to insert the rule at the top
 Choose a service group name if the rule needs to be applied to a particular service group 
 Choose category as security event
 Add criteria for the event log to be included as matching criteria
 Select actions to be applied Quarantine, Webhook or suppress log 
 Enable status
 The log levels or process names can be used as other matching criteria
 Click "insert to top" to insert the rule at the top
 Choose a service group name if the rule needs to be applied to a particular service group 
 Choose Event the category
 Add name of the event log to be included as the matching criteria
 Select actions to be applied - Quarantine, Webhook or suppress log 
 Enable status
 The log Level can be used as other matching criteria

![events](admission.png)
 Click "insert to top" to insert the rule at the top
 Choose a service group name if the rule needs to be applied to a particular service group 
 Choose category CVE-Report 
 Add log level as matching criteria or cve-report type 
 Select actions to be applied Quarantine, Webhook or suppress log (quarantine is not applicable for registry scan)
 Enable status

#### Creating response rule for CIS benchmarks (log level and benchmark number as matching criteria) 

+ Click "insert to top" to insert the rule at the top 
+ Choose service group name if rule need to be applied  for a particular service group
+ Choose category Benchmark
+ Add log level as matching criteria or benchmark number, e.g. “5.12” Ensure the container's root filesystem is mounted as read only
+ Select actions to be applied Quarantine, Webhook and suppress log (quarantine is not applicable Host Docker and Kubenetes benchmark)
+ Enable status

![cis](resp8-b.png)
 You may want to unquarantine a container if it is quarantined by a response rule
 Delete the response rule which caused the container to be quarantined, which can be found in the event log
 Select the unquarantine option to unquarantine the container after deleting the rule
Check the box to unquarantine any containers which were quarantined by this rule

