---
layout: page
title: Filter and import
---
# Filter and import

It's now time to import those quads into a real graph on a Turtle file. BUT if we try now it won't work because of some blanks in our dataset (some missing url basically). When Grafter writes the triples, it validates them and Grafter does not accept missing data. Thus we have to filter those triples so that, if there is a missing data, the whole triple is not written.

## Filter

In a src/cmd-line/filter.clj file.

