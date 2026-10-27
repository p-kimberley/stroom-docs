---
title: "Analysing tables and charts"
linkTitle: "Analysing tables and charts"
weight: 23
date: 2026-10-27
tags:
  - ai
  - dashboard
  - search
description: >
  Asking questions of the rows of a dashboard or query table, the events behind them, and the charts drawn from them.
---

The agent can answer questions about the results you are looking at, e.g. _"Which users had the most failed logins this week?"_, _"What kinds of error message are there?"_ or _"What's this spike?"_.
It works out figures in code, with its tools, rather than by reading rows, so its counts and totals are exact.


## Attaching a Table

A table gets to the chat in one of these ways:

* a table's or a chart's {{< stroom-icon "ai.svg" "Ask Stroom AI" >}} _Ask Stroom AI_ button, which attaches the table's rows (for a chart, the rows it draws);
* attaching a dashboard or query tab, after which the agent can take a snapshot of the results it shows;
* the agent's own StroomQL queries, whose results it saves as tables.

An attached table is downloaded into the chat's workspace, up to 50,000 rows by default.
The model isn't given the rows themselves, but the table's profile: its row and column counts, and for each column its type, how many values are empty, roughly how many distinct values it has, its smallest and largest values and its most common values, with a few sample rows.
From that it decides what to ask of the data.


## How the Agent Works Out Figures

The agent's table tool queries a table exactly, in Stroom's expression language (as dashboards use): it can add columns worked out for each row, filter rows, group them and aggregate, count distinct values, sort and limit.
The agent is told to use it for every figure, never counting rows or working figures out by reading them.

For free text, such as log messages, the agent can:

* count the text as patterns, with numbers, IP addresses, dates and IDs masked, so that _"User 1234 logged in from 10.1.2.3"_ and _"User 5678 logged in from 10.4.5.6"_ count as one pattern;
* summarise a column, which reduces it to its distinct patterns with their counts and, only if there are still too many, has the model summarise them in batches.

Results the agent works out can be saved as CSV files in the workspace, to work on further or to chart.


## The Events Behind the Rows

When a table isn't grouped, each row carries the stream and event it came from (from the hidden `StreamId` and `EventId` columns), so the agent can look at the events themselves, e.g. for a value the table doesn't show:

* it fetches the events behind the rows (up to 500 at a time by default, which needs the _View Data_ permission);
* it searches them, with XPath or for text, and adds what it finds as a column of the table, for the table tool to count or group.

A value you need regularly can be added to the table itself with a [search extraction]({{< relref "search-extraction" >}}).


## Models without Tools

A model that can't call tools is given the first 10,000 rows of each table (by default), and told when there are more.
If they don't fit its context window, the tables are analysed in batches, and the partial answers are merged into one answer.
These limits, and the prompts used for batches and for merging, are set by an administrator on the **Table Analysis** tab of the chat's **Settings** (from the chat's {{< stroom-icon "ellipses-vertical.svg" "Menu" >}} menu).

Answers worked out this way are less exact than those of a model with tools.


## Analysing a Chart

Each chart Stroom AI generated sends Stroom a render report, which the agent reads: what it drew, any rows left out and why, and any error.
It also says what you are looking at:

* the part zoomed in on, e.g. a period of time;
* what is selected, as the chart labels it;
* the series hidden from the legend;
* the interval an _Auto Interval_ chart groups by.

So you can ask _"What's this spike?"_ or _"Explain what I've selected"_, and the agent reads the rows behind it from the chart's table.

Ask what you can do with a chart, and the agent offers a few next steps that suit its kind and its data.
The chart works out the figures below itself, and the agent quotes them from the report rather than reading them off the picture.


### Trends

Line, bar (of dates) and scatter charts can draw a trend line: a straight-line fit, or a moving average.
For each series (or the total, for a stacked chart), the report gives:

* the **direction**: rising, falling or flat (flat meaning it changes by less than 5% of the series' average over the time shown);
* the **slope**, per a unit that suits the time shown, e.g. _+12/day_;
* **R²**, how well a straight line fits, from 0 to 1.
  Below about 0.3 the trend is weak, and drawn fainter; the agent then says the values vary too much to call it rising or falling.

With _Auto Interval_ on, the trend is of each interval's values, so it changes as zooming changes the interval, and the agent says which interval it was read at.


### Clusters

A network graph can colour its clusters: groups of nodes more linked to each other than to the rest, found with the Leiden method.
The report gives:

* how many clusters there are, and their **modularity**, how clear-cut they are (below about 0.3 they are weak, above about 0.5 clear);
* the largest clusters, each named after its most connected member (e.g. `mchen +11`), with its size;
* the **bridges**: the nodes linked to the most other clusters, often the people or hosts worth a closer look.

Mapping a known grouping, such as each person's department, to the graph's groups also gives the group most of each cluster is in, so you can see where the clusters and the groups differ.


### When a Chart Shows Nothing

Ask why, and the agent checks the render report, the rows of the chart's table, the table's filters and grouping, and the selection filters that feed it.
It says what is filtering the rows out, and offers the fix: often clearing a selection on another chart.

See [Charts]({{< relref "charts" >}}) for making and changing charts.
