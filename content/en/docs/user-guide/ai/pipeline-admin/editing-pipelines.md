---
title: "Editing pipelines"
linkTitle: "Editing pipelines"
weight: 20
date: 2026-10-27
tags:
  - ai
  - pipeline
  - xslt
  - processing
description: >
  Looking into, stepping, fixing and changing existing pipelines with the agent, and processing and reprocessing their streams.
---

Beyond [onboarding new sources]({{< relref "onboarding" >}}), the agent can work on the pipelines you already have.
Attach the pipeline (or its Stepping view) to the chat, or name it.


## Understanding a Pipeline

* **The pipeline as Stroom runs it**: the agent reads a pipeline merged with the templates it inherits from, so it sees every element, property and XSLT that applies, and which come from the template.
* **What it processes**: the feeds a pipeline's processor filters select (through dictionaries and stream ids too), and the filters' progress.
* **Its dependencies**: the documents it uses, and those that use it.
* **Its reference data**: the lookups its XSLTs make, which maps they read, and the pipelines and feeds that load each map.
* **Its translation**: what an XSLT does to an event, and, for a translation Stroom generated, the field mapping it was made from.

> _"Explain what the ACME-EVENTS pipeline does to a login event, and which parts come from its template."_


## Stepping and Fixing a Translation

With a pipeline's Stepping view attached, the agent sees the selected element's input, output and code, as you do.

* **Stepping**: it can step one record through a pipeline and show what each element does, step every record of a set of streams for one verdict, or step chosen records of existing streams where they are.
* **Proposing a change**: asked to fix something (_"Make the time stamp UTC"_), the agent proposes a change to the XSLT or text converter being stepped.
  Before it is offered, the change is tried by stepping the record with it, and refused if it gives errors the current code doesn't.
  It is shown as a diff with _Review and apply_, which puts it in the editor, unsaved, for you to check and save.
* **Proving a fix**: for a fix to a production pipeline, the agent steps every record of the affected streams with the fixed code, compares the outputs with the current code's (only the expected parts should change), and gives you the code's diff and the outputs that change, to review before anything is saved.
* **Comparing outputs**: it can step the same records through the current pipeline and a candidate (draft code, or a new version), and compare each record's output field by field.

XSLT the agent writes is compiled with Stroom's own XSLT engine and functions, and checked against the event-logging schema and quality rules (a full UTC time stamp, the System's name and environment, one action under EventDetail, no empty elements) before it is saved.
It may not call functions that reach outside Stroom.

> _"Step stream 4567 through this pipeline: why are records 12 to 15 producing errors?"_


## Changing Production Pipelines

The agent doesn't change production documents in place.
Instead, in a [build]({{< relref "onboarding#how-a-build-works" >}}):

* **New version**: a production pipeline's next version, with its translation written anew from a field mapping, keeping the reference data lookups you choose.
* **Working copy**: a copy of a source's pipeline (or XSLT) with exactly the original's structure and settings, to change and test.
* **New output**: a further pipeline for a source's Events, e.g. CEF (ArcSight Common Event Format) output, or forwarding.

When it is done and checked, promotion writes working copies back over the production documents (after a backup), and puts new versions where their type's destination is.
Promotion shows you the layout as a tree first, and asks before it changes anything.

The agent can also change a pipeline it made: its elements' properties, and the reference data its XSLT looks up (a feed and its loader pipeline).

> _"Make a new version of the ACME-EVENTS translation that maps the new `src_zone` field."_


## Reference Data

* The agent lists the reference data maps the environment loads, with each map's key and value, and the loader pipeline and feeds behind it, so a translation can look values up with `stroom:lookup()`.
* It can write the XSLT of a reference data pipeline from a mapping, and load a new table as a [reference data build]({{< relref "onboarding#reference-data" >}}).
* It can attach reference data to a pipeline it made, so its lookups find their maps.


## Processing and Reprocessing

* **Processor filters**: the agent can create a processor filter for a pipeline (asking you first), and disable one on a pipeline it made.
  Filters that promotion hands over are disabled, for you to enable with a button.
* **Reprocessing**: after changing a pipeline under development, it can process streams again through it (asking you first).
* **Waiting for processing**: it waits until each input stream's processing has ended, then reports the Events (or Reference) and Error streams it wrote.


## Finding Why Processing Fails

* **Across the cluster**: what is failing today and why, backlogs, queued and stuck tasks by node, and task lists, from processor tasks (this needs _Manage Processors_).
* **Errors**: the errors in Error streams, grouped by severity, element and message, each triaged as blocking, for review, or benign, and the pipeline's own elements told from inherited ones.
  Your own [error triage rules]({{< relref "configuration#error-triage-rules" >}}) are applied first.
* **Events**: what a stream's Events hold: counts per event type and action, and how often each part of the event-logging schema is filled in.
* **Tracing**: a stream, or one event in it, traced back to its raw stream, the record within it, the pipeline, and the XSLTs and text converters it went through.
* **Coverage**: the kinds of event a feed holds that a generated translation doesn't match yet, from the records it logged as unmatched.

> _"Which tasks for the ACME pipeline failed today, on which nodes, and why?"_


## Documentation

The agent can write a pipeline's documentation, on its Documentation tab, from what it knows of the source: its field mappings by event type, what is indexed, and the source's notes.
For a pipeline it made in a build, it writes it directly; for any other document, it proposes the change for you to review and apply.
