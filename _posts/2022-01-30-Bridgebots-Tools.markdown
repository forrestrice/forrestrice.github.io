---
layout: post
title:  "Bridgebots Tools: CSV Report"
date:   2022-01-30 11:05:09 -0700
categories: bridge bridgebots
---

I enjoy participating in the [bridge-dev](https://groups.io/g/bridge-dev) mailing list. There are a lot of cool ideas and discussions happening about building software for bridge. Recently, another group member reached out with questions about Bridgebots. They wanted to know if it would be possible to create a Comma Separated Values (CSV) report of LIN files for easier search and analysis of records. This seemed like a great way to build upon the work I've already done to support LIN and PBN parsing, so I started building. The result is the first release of the [bridgebots_tools](https://github.com/forrestrice/bridge-bots/tree/master/tools) package. For now the CSV report is the only tool, but I hope to add more in the future.

## Getting Started
The tools package will contain mostly useful scripts and/or command-line utilities. This means there are a couple ways it can be used. For basic usage, I would recommend installing the package with `pip install bridgebots_tools` and then executing it as a module with `python -m bridgebots_tools.csv_report`. If you want to edit the script (to add or remove output fields, for example) then you may prefer to download the source from github. The dependencies for the script ([bridgebots core](https://github.com/forrestrice/bridge-bots/tree/master/bridgebots) and [click](https://click.palletsprojects.com/en/8.0.x/)) can be installed manually or with `poetry install`.

## Report!
The csv_report tool takes in a bridge record file (PBN or LIN format) or directory of files and creates a report CSV. There are several input and output configuration options, and you can read all about them in the [Readme](https://github.com/forrestrice/bridge-bots/blob/master/tools/README.md). Here are a couple examples of how to run the report. Let's say we're interested in analyzing the 2019 Spingold Final. We can start by analyzing a [single segment](https://www.bridgebase.com/tools/handviewer.html?linurl=https://www.bridgebase.com/tools/vugraph_linfetch.php?id=64787). You can find the download link by searching for "spingold" in the [BBO vugraph archive](https://www.bridgebase.com/vugraph_archives/vugraph_archives.php?v3b=). For simplicity I have also uploaded all the examples to a public [google drive folder](https://drive.google.com/drive/folders/1hOQlNl7Ld9e6Ms8DbZEeUyDWl9il9Ikd?usp=sharing).

```console
python -m bridgebots_tools.csv_report bridgebots_tools_example/64787.lin bridgebots_tools_example/64787.csv
```
The team results CSV can be viewed on [google sheets](https://docs.google.com/spreadsheets/d/1UgL_Oe4R4g_4lEgeDuYkFp8VZoPNy9AJjfq_VJYirIc/edit?usp=sharing).

Now let's analyze all the segments of the finals, and list the boards in `individual` format (one row per board instead of one row per board pair).

```console
python -m bridgebots_tools.csv_report --output_format=individual bridgebots_tools_example/ bridgebots_tools_example/spingold_finals_all.csv
```

All 120 boards can now be analyzed. Check out the results on [google sheets](https://docs.google.com/spreadsheets/d/1_Iplmjc7-ipEED0qGOXiTc9iFpGj7zQE4qtbW5Eem7M/edit?usp=sharing).

## Analysis
Now that the data has been exported to CSV, there a variety of ways we could continue our analysis. We could import the data to a tool like pandas, or we could filter, aggregate, and graph the data inside of excel or sheets. Here are some examples of what you might try:

A histogram of the HCP held by whoever opened the bidding:
![histogram_of_opener_hcp.png](/assets/bridge/bridgebots/histogram_of_opener_hcp.png)

The relationship between opening HCP and opening seat:

![opener_hcp_vs_opener.png.png](/assets/bridge/bridgebots/opener_hcp_vs_opener.png)


The relationship between opening HCP and the final score for opener:
![opener_score_vs_opener_hcp.png](/assets/bridge/bridgebots/opener_score_vs_opener_hcp.png)

## Building a Toolbox
It was fun to build a tool on top of Bridgebots to help meet the analysis goals of other bridge players. In the future, I would like to expand the toolbox to include features like automated BBO history downloads and Double Dummy Analysis. If you have ideas for improvements or new tools to build, please reach out!