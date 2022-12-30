# Open Runbook

The OpenRunbook project aims to maintain a public runbook for open source projects.

# Why?
Modern open source software has grown very complex and is fast changing. Without proper observability and runbooks it has become increasingly hard to run these systems reliably.

Open Runbook is a public runbook for operating modern software systems. Modern software systems are complex and take a lot of tribal knowledge to run. Open Runbook aims to be a place to capture document some of that tribal knowledge. If you are running an open source software, open runbook can be a starting point for effectively running the service.

# Goal
The goal of OpenRunbook project is to provide a starting point for your runbooks. However, since runbooks look at specific metrics we would also provide sample dashboards and configs. Every environment is a special snowflake so our runbook will not be a drop in replacement for your environment, but aims to be a better than average starting point on which your internal runbook can be based on.

# Repo organization

Each folder in the project will contain a runbook for a specific service. The runbook will be a single page. In addition, the runbook can contain additional assets like public dashboards and good starter configurations. For example, `kafka` directory contains the Kafka runbook. 

## Single page runbooks
There is no one way to structure the runbook for a service. So, people structure them in several ways.

One way to organize a runbook is as a single long page with an index at the top. Another way could be creating a page for each section in the runbook or sometimes creating a page for each alert etc.. For various reasons like aversion to a long page of text, better organizational clarity etc.. the later method is commonly used. There are several other ways in between to create a runbook.

Regardless of which structure is used, it is often the case that information in runbooks goes stale and may need updates from time to time. Another common problem with a runbook is that information in different sections is often inconsistent.

In practice, we found that a single page runbook suffers from fewer flaws than a runbook spread across multiple pages. Some of the issues we found include:

* In a multi-page runbook the information across sections(command to start or stop a process) can often get inconsistent as systems evolve. In contast, in a single page runbook since everyone is looking at the same page, often common actions are linked to a common section, thus avoiding duplication of data. Thus, it's easier to keep the runbook DRY(Do not repeat yourself). Further, since many eye balls are looking at the same runbook over and over again, information generally gets updated when inconsistencies or errors are found.

* Often the actions in the runbook may not match the issue or incident at hand. In such situations, an engineer has to understand other sections to take a different action. In case of a multi-page runbook, quickly skimming the other section may mean opening 10s of pages/browser tabs. In contrast, in a single page runbook all the context and other similar operations are on the same page.

* Using a simple text based search on the page, a user can quickly search through all the runbook. This ease of use is very important during an incident. Such ease of use is not possible in a multi-page runbooks.

* Multi-page runbook has a clear deliniation of sections. The same ease of navigation can be achieved by adding an index at the top of the page.

* One criticism of a long single page runbook is that it may obscure some useful information among information that may be some day useful. While true, a long list of runbook actions is a sign that users should probably use a script to achieve the same outcome instead of manual steps. Also, in those cases, it is ok to create a new page and link to it. However, such an action should be an exception and not the norm. 

# FAQ

## Why does this project exist?
* We are tired of haphazardly hunting through messy threads of GitHub issues and StackOverflow during an incident.
* We don’t want a one-off fix, we want to deepen our understanding of the problem space and have somewhat decent solutions to try.
* We want to give people a resource where they can benefit from our collective experience of running systems.
* We want to share best practices for writing runbooks.

## This runbook seems incomplete?
Every production environment is a special snowflake to which a runbook is customized. Since these runbooks are meant to be as general purpose as possible, they typically would have some holes in them where general commands would go in. So, they may appear incomplete and the goal of this project is to act as a starting point for your runbooks.

While not perfect, we hope these runbooks provide a better starting point than going through the rabbit hole of github threads and stack overflow questions during an incident.

## Where are these runbooks from?
These runbooks are production runbooks contributed by people from various companies like Slack, Salesforce etc..

## Contribution guidelines
Please submit a pull request to update the runbooks. 

**Creating a runbook for a new project**

To create a runbook for a new project, please create a folder for it. All configs should be placed in the configs folder.

Please use the single page runbook [template](/template/runbook_template.md) for the runbook.
