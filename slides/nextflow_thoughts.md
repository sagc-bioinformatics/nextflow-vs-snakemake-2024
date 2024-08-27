% Nextflow Thoughts (jekyll please don't make this a page)
% John Salamon
% Aug 28, 2024


# Parsing vs. running and order of execution

- There are two main phases
- Our results are not guaranteed to come back in the same order
- I'll demonstrate with some `println`s

# Operators

- Operators are used to control / inspect / modify channels
- I'll demonstrate with some `println`s
- demonstrate view, and demonstrate looking inside text

- Running relative to nextflow directory
- bin, templates, and lib directories

# 

- params, configs, more complex operators, executors
 
# Adding

- Groovy, parsing, workflow lazy execution, DSL2
- Configs and executors
- I'm showing you basic Nextflow. `nf-core` adds a lot extra, but for the basics, you don't need any of it

 (go thru params, then put em in config, and show that)

 ( to replicate csv, splitCsv operator is what you want https://nextflow-io.github.io/patterns/process-per-csv-record/ ) 

(transpose, groupTuple, meta maps, indexing outputs, pipes, collect...)

(dynamic language, cool and not cool)
(those recommendations..)


# script as separte file

should show that maybe

# Stuff to think about

- you work from inputs, and define your workflow explicitly
- Java/Groovy based, can use any java code (lots of quite useful libraries available))



# Things I don't like about Nextflow

- dynamic typing leads to larger possible error space to check
  - making things robust to difficult inputs requires checking types religously
  - this has also led to complex input schema validation tools in nf-core
- error messages often suck
  - hard to read if you're not experienced
  - often not localised, difficult to understand where exactly error arises
- dealing with exploding storage with scattered work/ directories
  - you can auto-delete after success, but you often don't want to
  - this means you come back later and realise you've got mountains of data sitting there
- documentation is comprehensive but can be hard to navigate
  - where to start? show some tutorials

# community

- community. everything here is cool but actually none of it comes close to why I'd
  personally go with nextflow, which is the community
- nf-core is great, lots of open discussion, opportunities to seek help, find collaborators..

- we wrote some processes here. if you separate a process out into its own file, write some tests,
  and provide a container image, then you can submit it as a module on nf-core
- there are thousands of these modules available on nf-core and you can import them really easily
- you can use tooling from nf-core - which I've installed, but we haven't touched
- the nf-core template is a great place to start with nextflow best practices - but is intimidating
  - I considered basing this workflow on the nf-core template but didn't.
    we could have easily spent all this time just learning about the nf-core template
    you don't actually need to interact with nf-core at all to use nextflow, and for a beginner
    it can definitely be hard to figure out where one thing ends and another begins
    lots of "magic" in the template to do common tasks that aren't part of nextflow proper

- but on the flip side, for many things you can just browse nf-core, find a pipeline that's
  close enough to what you want, and just modify it to suit you
  because configuration is separate, any pipeline on nf-core can (in theory) run anywhere,
  and every module will have a container image so you don't need to worry about it

# profilng etc

`nextflow main.nf -with-dag -with-report -with-timeline`

quick testing, nextflow console


# problems, gotchas

good link:

https://midnighter.github.io/nextflow-gotchas/gotchas/variable-scope/


One thing is the whole scope of variables - def means local scope, no def is global

outputs can have variable names but must be global, e.g. path(myPath), you can globally definie myPath in the script and that'll work.. it still has to be a string tho


# structure ideas

- show the cli tool first

- show version pinning?

- show https://training.nextflow.io/
  - this has a good example of the process things - drawing there is better too

 - also shows the pipeline abstraction from runtime


- pipeline parsing vs pipeline running



good points here: https://github.com/nextflow-io/nextflow/discussions/3107


other things:

- errorStraregy
- conditionality
- arity
-


# microbench

for this comparison, you'll notice if we get nf to use conda, it's slower than externally activating
this is because nextflow switches the env on for each run of the process individually
in this teeny tiny workflow, that added time (conda takes a while to activate and deactivate) we 
end up much slower overall because of this

Also maybe show, we can let nextflow manage it, or point to our already existing conda env


hmm, samtools view/flagstat could be same process.. or optimised a bit
and then also, if we have 4 cores, and set fastp -w flag properly, I think we'll actually benefit from parallel

# Comparison

|                  | Nextflow        | Snakemake     |
|------------------|-----------------|---------------|
| Language extends | Groovy          | Python        |
| DAG is defined   | Explicitly      | Implicitly    |
| Root of graph is | Inputs          | Outputs       |
| History          | More commercial | More academic |
| Community on     | Slack           | Distributed?  |


# Modularisation

- You can [include](`https://www.nextflow.io/docs/latest/module.html`) Nextflow scripts from other files.

# Dynamic issues

- `.getClass()` can be handy
