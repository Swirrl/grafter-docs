---
layout: page
title: 2. Running Pipelines
---

**NOTE: This guide covers Grafter 0.3.0**

# Running Pipeline Transformations

*This is the second part of the Grafter Getting Started Guide.*

In this section you'll see how a grafter can be used to:

- Clean tabular data
- Convert a CSV file into an Excel file or vice versa
- Convert tabular data into Linked Data (RDF)

You'll also learn how to list and run grafter transformations with our
command line tools, how transformations themselves are specified and
how to run them at the REPL.

## Running Transformations from the Commandline

To understand how Grafter works lets first take a look at the example
CSV file (`./data/example-data.csv`) which was installed by the
template:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ cat ./data/example-data.csv
name,sex,age
Alice,f,34
Bob,m,63</div>
</div>

This dataset follows the common pattern in CSV files of specifying the
header on the first row followed by the rows of source data.  Here we
see there are two records, one for Alice, the other Bob listing both
their sex and age.

Next lets take a look at what transformations are defined in this
project, by running the command:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter list
Pipeline                                                     Type      Arguments            Description
test-project.pipeline/convert-persons-data                   pipe      data-file            ;; An example pipeline that converts tabular person data into a different tabular format.
test-project.pipeline/convert-persons-data-to-graph          graft     data-file            ;; A pipeline that converts the persons data sheet into graph data.</div>
</div>

The command `lein grafter list` is one of the commands provided by the
plugin to list all of the pipelines defined within a project.  It
scans the projects classpath finds all of the pipelines (defined with
`defpipe` or `defgraft`) and displays their name, their type, the
arguments they expect and their documentation string.

The template project defines two pipelines, the first,
`convert-persons-data` is a `pipe`, which means it converts tabular
data back into another tabular format.  The second pipeline is a
`graft` which means it converts tabular data into graph data.

We'll talk more about this distinction later but lets try running both
of the pipelines to get an idea about the differences between pipes
and grafts.

## Executing a Pipeline Transformation

You can run both pipe and graft transformations with the `lein run`
command.  The format of the run command is:

    lein run <pipeline-name> input-args... output-file

So remember pipes convert data from one tabular format to another
tabular format.  So we can use the
`test-project.pipeline/convert-persons-data` pipe to convert our
`example-data.csv` file into another csv file.  We know from running
`lein grafter list` that the pipeline is expecting one input argument
as its source data-file.  Note that pipelines declare their input
arguments, but not their outputs, so its upto us and `lein run` to
supply a final output argument, lets ask for it to put the output into
a new CSV file (`output.csv`):

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter run test-project.pipeline/convert-persons-data ./data/example-data.csv output.csv
./data/example-data.csv --[test-project.pipeline/convert-persons-data]--> output.csv</div>
</div>

The output from the command above indicates that the pipeline function
operated on its sole input file and produced an output file as we
asked.  Looking at the output we can see what happened:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ cat output.csv
name,sex,age,person-uri
Alice,female,34,http://my-domain.com/id/Alice
Bob,male,63,http://my-domain.com/id/Bob</div>
</div>

We can see the transformation has converted the representation of
gender from the strings 'm' and 'f' to 'male' and 'female', whilst
deriving a new column called `person-uri` which has been built out of
a prefix and their name.  So lets ask for the data in an Excel file:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter run test-project.pipeline/convert-persons-data ./data/example-data.csv output.xlsx
./data/example-data.csv --[test-project.pipeline/convert-persons-data]--> output.xlsx</div>
</div>

This time the same table will be output but this time as an Excel
file.  You can see from this that for registered exporters, grafter
will detect desired file format from the file extension.  Grafter
currently ships with tabular data exporters for CSV, and Excel (both
xls and xlsx), we plan to add more, such as open-office in the future.

Exporting to a format like Excel has some benefits over CSV, in that
CSV will lose type information by converting all values back into
string values.

Now lets try running the `graft` pipeline
`convert-persons-data-to-graph` that the project defines.  This time
with graft runs we're expected to give it a linked data serialisation
format.  Grafter supports all main RDF serialisations including turtle
(`.ttl`), n-triples (`.nt`), trig (`.trig`), trix (`.trix`), n-quads
(`.nq`) and RDF XML (`.rdf`).  Again the desired format is infered by
grafter from the file extension, so lets ask for some linked data in
turtle format:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter run test-project.pipeline/convert-persons-data-to-graph ./data/example-data.csv output.ttl
./data/example-data.csv --[test-project.pipeline/convert-persons-data-to-graph]--> output.ttl</div>
</div>

Again we can see that like `pipes`, `grafts` are functions, however
this time they output graph data, as linked data in the chosen format.
Lets take a look at the linked data we generated:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ cat output.ttl
&lt;http://my-domain.com/id/Alice&gt; a &lt;http://xmlns.com/foaf/0.1/Person&gt; ;
	&lt;http://xmlns.com/foaf/0.1/gender&gt; "female" ;
	&lt;http://xmlns.com/foaf/0.1/age&gt; "34"^^&lt;http://www.w3.org/2001/XMLSchema#int&gt; ;
	&lt;http://xmlns.com/foaf/0.1/name&gt; "Alice" .

&lt;http://my-domain.com/id/Bob&gt; a &lt;http://xmlns.com/foaf/0.1/Person&gt; ;
	&lt;http://xmlns.com/foaf/0.1/gender&gt; "male" ;
	&lt;http://xmlns.com/foaf/0.1/age&gt; "63"^^&lt;http://www.w3.org/2001/XMLSchema#int&gt; ;
	&lt;http://xmlns.com/foaf/0.1/name&gt; "Bob" .</div>
</div>

Now this is more interesting, we've converted our tabular data into an
RDF graph of triples!  Lets see what it looks like in n-triples:

<div class="terminal-wrapper">
  <div class="terminal-inner">$ lein grafter run test-project.pipeline/convert-persons-data-to-graph ./data/example-data.csv output.nt
./data/example-data.csv --[test-project.pipeline/convert-persons-data-to-graph]--> output.nt

$ cat output.nt
&lt;http://my-domain.com/id/Alice&gt; &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#type&gt; &lt;http://xmlns.com/foaf/0.1/Person&gt; .
&lt;http://my-domain.com/id/Alice&gt; &lt;http://xmlns.com/foaf/0.1/gender&gt; "female" .
&lt;http://my-domain.com/id/Alice&gt; &lt;http://xmlns.com/foaf/0.1/age&gt; "34"^^&lt;http://www.w3.org/2001/XMLSchema#int&gt; .
&lt;http://my-domain.com/id/Alice&gt; &lt;http://xmlns.com/foaf/0.1/name&gt; "Alice" .
&lt;http://my-domain.com/id/Bob&gt; &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#type&gt; &lt;http://xmlns.com/foaf/0.1/Person&gt; .
&lt;http://my-domain.com/id/Bob&gt; &lt;http://xmlns.com/foaf/0.1/gender&gt; "male" .
&lt;http://my-domain.com/id/Bob&gt; &lt;http://xmlns.com/foaf/0.1/age&gt; "63"^^&lt;http://www.w3.org/2001/XMLSchema#int&gt; .
&lt;http://my-domain.com/id/Bob&gt; &lt;http://xmlns.com/foaf/0.1/name&gt; "Bob" .</div>
</div>

Its also worth mentioning that the Grafter plugin can also load inputs
from URL's e.g.

<div class="terminal-wrapper">
  <div class="terminal-inner">lein grafter run test-project.pipeline/convert-persons-data-to-graph "https://raw.githubusercontent.com/Swirrl/grafter-template/master/resources/leiningen/new/grafter/example-data.csv" output.nt
https://raw.githubusercontent.com/Swirrl/grafter-template/master/resources/leiningen/new/grafter/example-data.csv --[test-project.pipeline/convert-persons-data-to-graph]--> output.nt</div>
</div>


Now you know how to list and run pipelines defined in a Grafter
project, lets take an in depth look at how these transformations have
been expressed with Grafter.
