GRACC MCP Server
================

The GRACC MCP server lets an AI assistant answer questions about OSG usage in plain language.
It is a [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) server over
[GRACC](https://gracc.opensciencegrid.org/), the OSG accounting system.
It also looks up OSG Topology project and VO metadata and OSG's institution database,
so an answer can say what science a number represents and which institutions it came from.

Ask a question such as "Which VOs used the most core hours last month?"
and the assistant looks up the data and summarizes the results.
This is useful for broader-impacts sections of funding-agency reports,
and for anyone curious about how OSG is used.

!!! tip "Server address"
    Point any MCP client at:

        :::text
        https://gracc.opensciencegrid.org/mcp

    **Public**, with no login or API key.
    **Read-only**: it can query the data but never change it.
    Transport: **Streamable HTTP**.

| Property       | Value                                                                                  |
|----------------|----------------------------------------------------------------------------------------|
| Endpoint       | `https://gracc.opensciencegrid.org/mcp`                                                |
| Transport      | Streamable HTTP                                                                        |
| Authentication | None                                                                                   |
| Data           | OSG job accounting (GRACC) and OSDF data transfers                                     |
| Metadata       | OSG Topology projects and VOs, and OSG's institution database                          |
| Source code    | [opensciencegrid/gracc-mcp](https://github.com/opensciencegrid/gracc-mcp) (Apache-2.0) |

Connect an AI Assistant
-----------------------

Any MCP client that supports remote MCP servers can use it.
In your client, add a remote (sometimes called "custom") MCP server with the address above.
There is nothing else to configure.

### Claude Code

Register the server once, then ask questions in any Claude Code session:

```console
user@host $ claude mcp add --transport http gracc https://gracc.opensciencegrid.org/mcp
```

!!! note "Check the connection"
    Ask the assistant "What data does the GRACC server offer?"
    It should describe OSG job accounting and OSDF data transfers.

Example Questions
-----------------

Each of these questions works against the live server.
Use them as they are, or change the VO, site, state or dates to suit your needs.

### Usage

- "Which VOs used the most core hours last month?"
- "How many jobs did LIGO run last week, and how many core hours did they use?"
- "Show weekly GPU hours for the last three months."

### Science and institutions

- "What ran at the site Nebraska last month, what fields of science were they, and which institutions did they come
  from?"
  ([See how the assistant answers this one.](#how-the-assistant-answers-a-question))
- "Over the last 12 months of OSPool usage, which fields of science used the most core hours, and how many distinct
  institutions were involved?"
- "Which institutions in Nebraska have OSG projects, and how much did each use this year?"
- "Which institutions in EPSCoR jurisdictions ran on the OSPool this year?"

!!! info "EPSCoR jurisdictions"
    The server has no EPSCoR flag.
    It gives each institution's US state, and the AI assistant decides on its own which states are
    EPSCoR jurisdictions.
    Ask the assistant to list the jurisdictions it used, so you can check them against NSF's current list.

### Data transfers

- "Which OSDF namespaces read the most data last month?"

!!! tip "Tips for numbers that go into a report"
    - **Name exact dates**, such as "October 2026" or "2026-01-01 to 2026-11-01", rather than "last month",
      so you can run the same question again later.
    - **Ask how it got the number.**
      For example, "Explain how you calculated that."
      GRACC has no single "OSPool" label, so it is worth knowing how the assistant defined OSPool usage.
    - **Ask for units.**
      Core hours, job counts and GPU hours are different measures; OSDF numbers are in bytes.

How the Assistant Answers a Question
------------------------------------

Here is how an assistant answers
"What ran at the site Nebraska last month, what fields of science were they, and which institutions did they come
from?"
The question was asked in November 2026, so "last month" means October 2026.

1.  **Finds the exact site names.**
    "Nebraska" matches two OSG sites: Nebraska-CMS and Nebraska-Omaha.

1.  **Ranks the projects that ran there.**
    It adds up each project's core hours at those two sites in October: 90 projects in all.

1.  **Looks up the science and the institutions.**
    OSG Topology gives each project's field of science and institution.
    Two of the names, CMS and OSG, are whole VOs rather than projects.

1.  **Describes the VOs.**
    It looks up the CMS (Compact Muon Solenoid) and OSG VOs.

1.  **Groups and adds up.**
    It sums the core hours by field of science and counts the distinct institutions.

!!! success "The answer (October 2026)"
    Jobs at Nebraska-CMS and Nebraska-Omaha that reported a project used about 1.03 million core hours
    across 90 projects.
    The CMS VO accounted for about 0.77 million of them.
    The other 88 projects span 36 fields of science, led by Physics, Computer and Information Services,
    and Bioinformatics, and come from 61 institutions.

Available Data
--------------

### Jobs

OSG job accounting from GRACC, dated by when each job ended.

| Measures                    | Broken down by                       |
|-----------------------------|--------------------------------------|
| Core hours                  | VO                                   |
| Job counts                  | Project                              |
| Wall time and CPU time      | Site and resource group              |
| GPU hours                   | Resource type                        |
| Processor and GPU counts    | Probe and submit host                |

### OSDF data transfers

Daily summaries of [Open Science Data Federation](https://osg-htc.org/services/osdf) (OSDF) transfers.

| Measures                    | Broken down by                                       |
|-----------------------------|------------------------------------------------------|
| Bytes read                  | Namespace, the top-level path such as `/ospool`      |
| Bytes written               | Cache or origin site                                 |
| Vector-read bytes           | Server hostname                                      |
| Transfer counts             | Client domain, such as `wisc.edu`                    |

### Projects, VOs and institutions

| Record       | What it includes                                                                     |
|--------------|--------------------------------------------------------------------------------------|
| Projects     | Field of science, organization, department, sponsor, and institution                 |
| VOs          | Long name, community, fields of science, and URL                                     |
| Institutions | Name, US state, ROR ID, IPEDS and Carnegie details, and their OSG projects           |

This information comes from [OSG Topology](https://topology.opensciencegrid.org/)
and OSG's institution database, and is refreshed about once an hour.

### Limits

| Limit        | Value                                       |
|--------------|---------------------------------------------|
| Time range   | Up to 366 days per query                    |
| Results      | Up to 500 per query                         |
| Request rate | About 120 requests per minute per client    |

### Good to know

- Field of science and institution come from the project that each job reports.
  Some usage, such as CMS's, is recorded under a whole VO rather than a project;
  the assistant describes that usage at the VO level.
- When an institution detail is unknown, it is left blank rather than counted as "no".
- For the full list of the server's tools and options, see the
  [gracc-mcp repository](https://github.com/opensciencegrid/gracc-mcp).

Example Records
---------------

These records come from the live server and show the kinds of details you can ask about.

### Institution: University of Nebraska–Lincoln

| Detail                          | Value                                                     | Question to try                                                     |
|---------------------------------|-----------------------------------------------------------|---------------------------------------------------------------------|
| State                           | NE                                                        | "Which institutions in Kansas have OSG projects?"                   |
| Control                         | Public                                                    | "Which public institutions in Kansas have OSG projects?"            |
| Institution size                | 20,000 and above                                          | "Which projects come from institutions with under 1,000 students?"  |
| Program length                  | Four or more years                                        | "Do any two-year colleges have OSG projects?"                       |
| Carnegie classification (2021)  | Doctoral Universities: Very High Research Activity        |                                                                     |
| Carnegie classification (2025)  | Research 1: Very High Spending and Doctorate Production   | "How many institutions classified as Research 2 have OSG projects?" |
| ROR ID                          | https://ror.org/043mer456                                 | "List the ROR IDs of the institutions behind last month's top 20 projects." |
| Website                         | www.unl.edu                                               |                                                                     |
| OSG projects                    | 29                                                        |                                                                     |

### Project: IceCube

| Detail           | Value                                   |
|------------------|-----------------------------------------|
| Field of science | Astrophysics                            |
| Organization     | University of Wisconsin–Madison         |
| Department       | Physics                                 |
| Sponsor          | OSG Connect                             |
| Institution      | University of Wisconsin–Madison (WI)    |

### VO: LIGO

| Detail            | Value                                                                                          |
|-------------------|------------------------------------------------------------------------------------------------|
| Long name         | The Laser Interferometer Gravitational-Wave Observatory                                        |
| Community         | The LIGO Scientific Collaboration (LSC)                                                        |
| Fields of science | Gravitational Physics (primary); Astrophysics, Physics, and Physics and astronomy (secondary)  |
| URL               | http://www.ligo.org                                                                            |
| Active            | Yes                                                                                            |
