# Open Runbook

The OpenRunbook project aims to maintain a public runbook for open source projects.

# Why?
Modern open source software has grown very complex and is fast changing. Without proper observability and runbooks it has become increasingly hard to run these systems reliably.

Open Runbook is a public runbook for operating modern software systems. Modern software systems are complex and take a lot of tribal knowledge to run. Open Runbook aims to be a place to capture document some of that tribal knowledge. If you are running an open source software, open runbook can be a starting point for effectively running the service.

# Goal
The goal of OpenRunbook project is to provide a starting point for your runbooks. However, since runbooks look at specific metrics we would also provide sample dashboards and configs. Every environment is a special snowflake so our runbook will not be a drop in replacement for your environment, but aims to be a better than avrage starting point on which your internal runbook can be based on.

# Repo Organization

Each folder in the project will contain a runbook for a specific service. The runbook will be a single page. In addition, the runbook can contain additional assets like public dashboards and good starter configurations. For example, `kafka` directory contains the Kafka runbook. 

# FAQ

## Why does this project exist?
* We are tired of haphazardly hunting through messy threads of GitHub issues and StackOverflow during an incident.
* We don’t want a one-off fix, we want to deepen our understanding of the problem space and have somewhat decent solutions to try.
* We want to give people a resource where they can benefit from our collective experience of running systems.
* We want to share best practices for writing runbooks.

## This runbook seems incomplete?
A runbook is very customized to a specific environment. Since these runbooks are meant to be as general purpose as possible, they typically would have some holes in them where general commands would go in. So, they may appear incomplete. But, they are definitely better than going through the rabbit hole of github threads and stack overflow questions during an incident.

## Where are these runbooks from?
These runbooks are production runbooks contributed by people from various companies like Slack, Salesforce etc..
