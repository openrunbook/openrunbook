# Runbook template
This document captures a template for a runbook. This template will be used as a guide for all the runbooks in this project. 

## Single page
A runbook is a single page document that acts as an operational manual for a a service. Keeping the runbook to be a single page document has several advantages.
* All the information related to a service can be found in one place.
* Since all the information is in a single page, information can be cross referenced easily across sections using a simple search on the page. 
* If it's a single page, it much easier to keep the runbook coherant, consistent, accurate and up-to-date since various sections of the runbook will be referenced muliple times during an incident.

## Sections in a runbook
A runbook should atleast contain the following sections: 

### Table of Contents
Since a runbook is a single page document, every runbook should have a table of contents at the top which lists the contents of the runbook.

### Introducion

The introduction section of the runbook should provide a very high level introduction to the service like what the service does and the impact if it's down.

It should also briefly explain the operational model of a service from a high level, so users have a mental model of how the service works internally. This operational model is extremely helpful to understand the metrics and logs a system would provide. 

### Links

The links section of the runbook should provide all the useful links a user would need to run this service effectively. For example, it can contain links to source code for this service, links to CI/CD environment, links to production and test environment, links to deployment tools, dashboards, alerts etc..

In addition, it can also contain useful links to additional documentation for a service, design docs, links to security creds etc..

### Alerts 
This section contains a sub-section for each alert defined for the service. Each of the alert should contain the following information:

* Why does this alert happen? What is the impact of this alert on the functioning of the system?
* Steps to triage the issue and identify the root cause. If the root cause couldn't be identified with these steps, we should also document an escalation path for this alert.
* Once the root cause is identified, document the steps to mitigate this issue. 

### Operations

This section contains one section for each operation of the system. These steps can be used for routine maintainance of the system or may be needed to triage or fix the root cause of the issue. 

Just like code, this section would have a sub-section for any operation that is needed in more than one alert above. If a single operation is only needed once, it may be better to add that command with the alert section itself.
