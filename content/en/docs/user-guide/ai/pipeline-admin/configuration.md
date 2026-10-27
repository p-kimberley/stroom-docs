---
title: "Configuring for your environment"
linkTitle: "Configuring for your environment"
weight: 30
date: 2026-10-27
tags:
  - ai
  - pipeline
  - configuration
description: >
  Pointing the agent at your pipeline templates, and setting its build folder, error triage rules, field conventions and sample uploads.
---

The agent's onboarding and pipeline tools follow Stroom's own conventions by default.
These settings, all under `stroom.ai`, fit them to your environment.


## The Build Folder

`builds.workspaceFolder` (default `MCP Workspace`) is the folder below `System` in which each build gets a folder of its own.
Users who onboard sources need permission to create folders in it.


## Pipeline Stages and Templates

The agent recognises each pipeline's _stage_, and looks for templates for a new pipeline of that stage, by `pipelineStages`.
The stages are:

Stage | What it does
-- | --
`translation` | Raw Events to Events.
`indexing` | Events to an index.
`discovery` | Raw structured data straight to an index, with no Events.
`reference` | A Raw Reference feed to the reference data maps that `stroom:lookup()` reads.
`records` | A source to a Records stream.
`forwarding` | Events sent on, e.g. as CEF.

Each stage has:

Property | Description
-- | --
`templateFolders` | The Explorer folders (paths from System, e.g. `System/Template Pipelines`) where the agent looks for templates for a new pipeline of this stage.
`templateNames` | Glob patterns (e.g. `Event Data (*)`) that the names of the templates in those folders must match. Empty for any name.
`schemaGroups` | The SchemaFilter `schemaGroup` values that mark a pipeline as this stage, when its elements don't say.
`streamTypes` | The StreamAppender `streamType` values that mark a pipeline as this stage, when its elements don't say.

Templates are looked for in the configured folders first, then among pipelines that working pipelines already inherit from, then in Stroom's standard templates.
If no template has the parser the samples need, the agent offers a template with its parser replaced; only if there is no translation template at all does it build an events pipeline of its own.
Likewise, only if there is no discovery template does it build a discovery pipeline of its own.
So that new pipelines inherit from your templates, make sure each stage you use (including `discovery` and `reference`) can find them.

Set the folders and names to your own templates, so that new pipelines inherit from them, and the schema groups and stream types if your templates validate against other schema groups or write other stream types.


## Error Triage Rules

The agent triages pipeline errors as **blocking** (must be fixed), **review** (may be caused by the pipeline's own content) or **benign** (report only), using Stroom's own rules.
`errorTriage.rules` adds your environment's rules, checked in order before Stroom's; the first to match decides.

Each rule may give:

Field | Matches
-- | --
`severity` | A list of `FATAL`, `ERROR`, `WARNING` and `INFO`.
`element` | A glob on the element's id.
`own` | `true` for an element the pipeline sets itself, `false` for one it inherits.
`message` | A regular expression on the error message.

and must give its `class` (`blocking`, `review` or `benign`) and `reason`.
For example, so that lookup misses for accounts not in reference data are reported but not fixed:

```yaml
appConfig:
  ai:
    errorTriage:
      rules:
        - element: decorationFilter
          severity: [WARNING, INFO]
          message: 'lookup'
          class: benign
          reason: 'User decoration miss for accounts not in reference data'
```

Errors a user tells the agent are benign during onboarding are also recorded in the source's documentation.


## Field Conventions

When indexing, the agent names fields following a convention that the user chooses: Elastic Common Schema (`ecs`) or Stroom's flat naming (`stroom-flat`).
It can also follow the fields of an index you point it at.
The conventions are served to MCP clients as resources (`stroom://conventions/<name>`).


## Sample Uploads

Samples on a user's machine are uploaded through Stroom's MCP endpoint, which needs to be turned on, with the commands the agent gives (see [Sample Data]({{< relref "onboarding#sample-data" >}})).
`sampleUploads.maxUploadMb` (100), `sampleUploads.commandsLast` (15 minutes) and `sampleUploads.maxPendingPerUser` (20) limit them (see [Deploying the MCP endpoint]({{< relref "../agent-tools/deployment" >}})).


## Standing Instructions (AGENTS Documents)

Give the agent your environment's own rules in {{< stroom-icon "document/Documentation.svg" >}} Documentation documents named `AGENTS`, the equivalent of an `AGENTS.md` file.
An `AGENTS` document in a folder applies to everything in that folder and below it, e.g. `/Feeds/AGENTS` for every feed.
The agent reads those that apply to the folders, feeds and documents it works on, most general first, as the user (so only those they can view), e.g.:

* how XSLT is written, and how fields are mapped;
* how feeds, pipelines and other documents are named;
* folders that searches should leave out.

Each read is recorded in the audit log.


## Guides for the Agent

The agent also works from built-in guides on XSLT, the event-logging schema, Data Splitter, JSON input, reference data, indexing, CEF and documentation, which MCP clients can read too (`stroom://guide/<name>`).
