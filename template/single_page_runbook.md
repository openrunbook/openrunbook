# Single page runbook

There is no one way to structure the runbook for a service. So, people structure them in several ways.

One way to organize a runbook is as a single long page with an index at the top. Another way could be creating a page for each section in the runbook or sometimes creating a page for each alert etc.. For various reasons like aversion to a long page of text, better organizational clarity etc.. the later method is commonly used. There are several other ways in between to create a runbook.

Regardless of which structure is used, it is often the case that information in runbooks goes stale and may need updates from time to time. Another common problem with a runbook is that information in different sections is often inconsistent.

In practice, we found that a single page runbook suffers from fewer flaws than a runbook spread across multiple pages. Some of the issues we found include:

* In a multi-page runbook the information across sections(command to start or stop a process) can often get inconsistent as systems evolve. In contast, in a single page runbook since everyone is looking at the same page, often common actions are linked to a common section, thus avoiding duplication of data. Thus, it's easier to keep the runbook DRY(Do not repeat yourself). Further, since many eye balls are looking at the same runbook over and over again, information generally gets updated when inconsistencies or errors are found.

* Often the actions in the runbook may not match the issue or incident at hand. In such situations, an engineer has to understand other sections to take a different action. In case of a multi-page runbook, quickly skimming the other section may mean opening 10s of pages/browser tabs. In contrast, in a single page runbook all the context and other similar operations are on the same page.

* Using a simple text based search on the page, a user can quickly search through all the runbook. This ease of use is very important during an incident. Such ease of use is not possible in a multi-page runbooks.

* Multi-page runbook has a clear deliniation of sections. The same ease of navigation can be achieved by adding an index at the top of the page.

* One criticism of a long single page runbook is that it may obscure some useful information among information that may be some day useful. While true, a long list of runbook actions is a sign that users should probably use a script to achieve the same outcome instead of manual steps. Also, in those cases, it is ok to create a new page and link to it. However, such an action should be an exception and not the norm. 


