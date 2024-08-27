---
layout: page
title: Snakemake
permalink: /snakemake/
---

<nav>
  <h4>**Contents**</h4>
  * toc goes here
  {:toc}
</nav>


# Part 1

By the end of this tutorial you should be able to generate a Snakemake version of our template bash script.
Scroll to the end of part 2 for my attempt at the Snakemake conversion.

## Inspiration

Snakemake gets its inspiration from the program _Make_.
_Make_ is an old program that was originally designed to compile and install software.
It was appropriated by bioinformaticians because the program is ideal for writing pipelines.

The key concept with Makefiles is that they describe __recipes__ not __steps__.
A recipe tells the computer how to create one type of file from another.
The magic happens by having one recipe to describe one step that can be used for hundreds of different files.

```text
file_to_make: file_to_read
    shell_command
```

An example Makefile to compile some FORTRAN code:

```text
# These recipes can only be used for newExe.f90 and newExe.o
newExe.o: newExe.f90
    gfortran -c newExe.f90

newExe: newExe.o
    gfortran -o newExe newExe.o

# This will run on any .f90 file
%.o: %.f90
    gfortran -o $@ -c $<

# This will run on any .o file
%: %.o
    gfortran -o $@ $<
```



## Minimum requirements for Snakefile

Using the same concept as before, you start with your destination, _then_ you describe the journey.
__By default, Snakemake runs the first rule it encounters in the Snakefile.__
This is important because we use the first rule--`rule all`-- to declare the pipeline targets.
There are no outputs or shell command for rule all,
so the rule just tells Snakemake that we need the file "A.sam".

Create a file called `Snakefile` and add the following:

```python
# Targets - what is the final thing the pipeline will make?
rule all:
    input:
        "A.sam"

# rules - how will the pipeline make the target files?
rule map_reads:
    input:
        "genome.fa",
        "A.fastq"
    output:
        "A.sam"
    shell:
        "minimap2 -a -x sr genome.fa A.fastq > A.sam"
```

This is your pipeline and Snakemake will recognise it when you run Snakemake.
The only thing you need to tell Snakemake at this stage is how many concurrent jobs to run.
For only 1 job at a time, you would run this:

`snakemake -j 1`

If you name your snakefile something else you need to tell Snakemake that it is your Snakefile. 
e.g. 

`snakemake -s mySnakeFile.smk -j 1`



## Chaining rules

SAM files are stupid, lets convert it to a BAM file.
Update the targets (your destination) and add a new rule for file conversion.

```python
# Targets
rule all:
    input:
        #"A.sam"
        "A.bam"

# rules
rule map_reads:
    input:
        "genome.fa",
        "A.fastq"
    output:
        "A.sam"
    shell:
        "minimap2 -a -x sr genome.fa A.fastq > A.sam"

rule convert_sam:
    input:
        "A.sam"
    output:
        "A.bam"
    shell:
        "samtools view -bh A.sam > A.bam"
```

Why is this better than a bash script?
Currently, it's not. 
Here's what the bash script would look like:

```shell
minimap2 -a -x sr genome.fa A.fastq > A.sam
samtools view -bh A.sam > A.bam
```

Snakemake is better in the long run though.



## Wildcards

You hardly ever have one sample, so we need recipes that will work for files with different names.
We also saw this concept at the top with the Makefile.
Snakemake uses jinja variables (curly braces) for wildcards and accessing elements in the shell commands.
Inputs and outputs can be accessed from the shell commands as these jinja variables.
If you're familiar with Python, these are __not__ the same as f-strings.

```python
# jinja variable - variable is interpreted by Snakemake
"{file}.bam"

# f-string - variable is resolved by Python BEFORE being passed to Snakemake
f"{file}.bam"
```

Wildcards must be consistent and present in both inputs and outputs.
They can be accessed in the shell command by prepending _wildcards._ like below.

```python
rule convert_sam:
    input:
        "{file}.sam"
    output:
        "{file}.bam"
    shell:
        "samtools view -bh {input} > {output}"
        # this would also work:
        # "samtools view -bh {wildcards.file}.sam > {wildcards.file}.bam
```

Now, the rule _convert_sam_ will match any file ending in _.sam_ and can make a _.bam_ file from it.



## Named inputs and outputs

For rule _map_reads_ we have two different inputs: the genome fasta file, and the reads to map.
This is a list and you can access list elements like normal with python, 
where `input[0]` is `genome.fa` and `input[1]` is `A.fastq`. 
This can get confusing quickly so named inputs are better.

convert the rule to use wildcards, and convert the inputs to named inputs.

```python
rule map_reads:
    input:
        genome = "genome.fa",
        reads = "{sample}.fastq"
    output:
        "{sample}.sam"
    shell:
        "minimap2 -a -x sr {input.genome} {input.reads} > {output}"
```



## Expand

Now that our rules will work with any sample names, lets declare the other samples as targets.
We can do this one-by-one like so:

```python
rule all:
    input:
        "A.bam",
        "B.bam",
        "C.bam"
```

Which is fine if you only have a couple of samples, but it's a pain if you have lots.
The expand function lets you declare multiple samples with similar prefixes and/or suffixes.
You can use this function to declare targets, or in regular rules for inputs or outputs etc.
You can also use it outside of rules in regular Python code.

```python
rule all:
    input:
        expand("{sample}.bam", sample=['A','B','C'])
```



## Temp files

After we convert the SAM to a BAM file, we don't need the SAM file.
We can tell Snakemake to delete any intermediate files as soon as it doesn't need them anymore.
We do this by marking them as temporary with `temp()` or `temporary()`.
Only outputs can be marked as temporary.

```python
rule map_reads:
    input:
        genome = "genome.fa",
        reads = "{sample}.fastq"
    output:
        temp("{sample}.sam")
    shell:
        "minimap2 -a -x sr {input.genome} {input.reads} > {output}"
```



## Protected files

If a certain job is very computationally expensive, 
you can make the output write-protected to prevent accidental deletion.
Just mark it with `protected()`

```python
rule convert_sam:
    input:
        "{file}.sam"
    output:
        protected("{file}.bam")
    shell:
        "samtools view -bh {input} > {output}"
```



## Configuration

At the moment, we have the reference genome and the sample names hard-coded in the pipeline.
This is bad practice.
If we want to run the pipeline on a different genome or with different samples, 
we would have to manually rename the file names throughout the Snakefile.
It is better to have the file names declared in a separate configuration file.

Create a new file called config.yaml and add the following:

```yaml
genome: genome.fa
samples:
 - A
 - B
 - C
# or 
# samples: ['A','B','C']
```

YAML is a very popular format for config files as it is very simple and human-readable.
Above, `genome` is a string variable with the value `genome.fa` 
and `samples` is a list with the variables `['A','B','C']`.

To use this config file in our pipeline, we can pass it when we run it like so:

`Snakemake -j 1 --configfile config.yaml`

or specify the config file name in the pipeline:

```python
configfile: "config.yaml"
```

You can also do both;
the command line option `--configfile` will override the file specified in the Snakefile.

We can now access the config variables in the pipeline via a dictionary variable called 'config'.
Let's update our pipeline to remove the hard-coded file names and use the config dictionary.

```python
configfile: "config.yaml"

rule all:
    input:
        expand("{sample}.bam", sample=config['samples'])

rule map_reads:
    input:
        genome = config['genome'],
        reads = "{sample}.fastq"
    output:
        temp("{sample}.sam")
    shell:
        "minimap2 -a -x sr {input.genome} {input.reads} > {output}"

rule convert_sam:
    input:
        "{file}.sam"
    output:
        "{file}.bam"
    shell:
        "samtools view -bh {input} > {output}"
```



## Directories

It's generally good practice to keep each rule's files in their own directories.
Let's move the reads to their own directory. 
Create a folder called `reads/` and move the sample `.fastq` files there.
Make sure the new file paths are specified in either the config file, or the Snakefile.
Let's save the SAM files to `temp/` and the BAM files to `output/`.

```python
configfile: "config.yaml"

rule all:
    input:
        expand("output/{sample}.bam", sample=config['samples'])

rule map_reads:
    input:
        genome = config['genome'],
        reads = "reads/{sample}.fastq"
    output:
        temp("temp/{sample}.sam")
    shell:
        "minimap2 -a -x sr {input.genome} {input.reads} > {output}"

rule convert_sam:
    input:
        "temp/{file}.sam"
    output:
        "output/{file}.bam"
    shell:
        "samtools view -bh {input} > {output}"
```

Snakemake will automatically create the directories if they don't exist,
and it will remove any empty directories after their temp files are removed.

It can be helpful to use variables for directory names in case you wish to change the 
directory structure of you pipeline, or have it a configurable option.

In the config.yaml file:

```yaml
genome: genome.fa
samples: ['A','B','C']
outputDirectory: output
readDirectory: reads
```

In the Snakefile, use Python's plus symbol for string concatenation to combine string variables with bare strings.
e.g. `out_dir + "/{sample}.bam"` will result in `"output/{sample}.bam"`.

```python
configfile: "config.yaml"

temp_dir = 'temp'
out_dir = config['outputDirectory']
read_dir = config['readDirectory']

rule all:
    input:
        expand(out_dir + "/{sample}.bam", sample=config['samples'])

rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    shell:
        "minimap2 -a -x sr {input.genome} {input.reads} > {output}"

rule convert_sam:
    input:
        temp_dir + "/{file}.sam"
    output:
        out_dir + "/{file}.bam"
    shell:
        "samtools view -bh {input} > {output}"
```



## Threads

Snakemake supports some sophistocated resource management.
The most obvious is multithreading.
Minimap2 can utilise say 8 threads by passing the `-t 8` on the command line.
We can tell Snakemake how many threads a job will use, and it will adjust jobs accordingly.

Let's tell Snakemake to use 8 threads for the mapping step with `Threads:` and tell Minimap to use that many threads.

```python
rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    threads:
        8
    shell:
        "minimap2 -t {threads} -a -x sr {input.genome} {input.reads} > {output}"
```

Now when we run the pipeline, if we increase `-j` to 8, minimap will use 8 threads. e.g.

`Snakemake -j 8`

If we leave it at 1, Snakemake will automatically scale the minimap step down to 1 thread.



## Params

It can be helpful to make run parameters for some steps customisable, rather than hard-coded in the rules.
At the moment, minimap is using short-read mapping parameters `-x sr`.
To map nanopore reads we would instead use `-x map-ont`.
Parameters can added as configuration options, included in rules under `params:`, 
and accessed in the command as jinja variables.
Like inputs and outputs, you can have multiple parameters and they can be named.

in config.yaml:

```yaml
genome: genome.fa
samples: ['A','B','C']
outputDirectory: output
readDirectory: reads
minimapParameters: -x sr
```

in the Snakefile:

```python
rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    threads:
        8
    params:
        config['minimapParameters']
    shell:
        "minimap2 -t {threads} -a {params} {input.genome} {input.reads} > {output}"
```



## Conda environments

Installing things with conda is easy.
Making Snakemake do it is even easier.
You can declare a conda environment for a rule to ensure that the correct version of the program is being used,
and that it is in an isolated environment that will not mess with other programs.

Create a directory to host your environment files called `envs/` and create a file in it called `minimap.yaml`.
This file will tell Snakemake what programs it needs to install from conda to run a particular rule.

```yaml
name: minimap2
channels:
    - bioconda
    - conda-forge
    - defaults
dependencies:
    - minimap2>=2.20
```

Use this environment in a rule with `conda:` like so:

```python
rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        'envs/minimap2.yaml'
    shell:
        "minimap2 -t {threads} -a {params} {input.genome} {input.reads} > {output}"
```

__The path to the environment is always relative to the location of the Snakefile.__
This is a useful feature, because you can run your pipeline from a different directory and not have to worry about
copying the env files to your current location.

To make use of these conda environments you need to run Snakemake with the `--use-conda` flag, e.g.

`Snakemake -j 8 --use-conda`

Snakemake will now create an isolated environment for running this job.



## Run directives

A Snakemake rule does not have to be a shell job, you can run some python code instead.
Lets add a rule to count the reads in the .fastq files, and update the targets.
Run directives are useful because they can directly access the input and output variables as well as global variables.

```python
rule all:
    input:
        expand("output/{sample}.{file}", sample=config['samples'], file=['bam','reads.tsv'])

rule count_reads:
    input:
        read_dir + "/{sample}.fastq"
    output:
        out_dir + "/{sample}.reads.tsv"
    run:
        with open(output[0],'w') as out:
            count = file_len(input[0]) / 4
            out.write(wildcards.sample + '\t' + count + '\n')
```



## Script directives

If you have a lot of code, or an R script to run, it's neater to use the script directive than the run directive.
Lets convert the above to a python script.
With the script directive, you can access the snakemake inputs, outputs etc via a class called `snakemake`.

Create a folder called `scripts/`, and add a file in it called `count_reads.py`:

```python
with open(snakemake.output[0],'w') as out:
    with open(snakemake.input[0],'r') as f:
        for i, l in enumerate(f):
            pass
        count = str((i + 1) / 4)
        out.write(snakemake.wildcards.sample + '\t' + count + '\n')
```

update the rule to use the script directive:

```python
rule all:
    input:
        expand("output/{sample}.{file}", sample=config['samples'], file=['bam','reads.tsv'])

rule count_reads:
    input:
        read_dir + "/{sample}.fastq"
    output:
        out_dir + "/{sample}.reads.tsv"
    script:
        "scripts/count_reads.py"
```

__Like with envs, the script paths are always relative to the Snakefile.__
So much cleaner, isn't it?



# Part 2


## Standardised structure

We've been a bit lazy and have simply been putting files wherever we want. 
It's good practice to put all files into a standardised directory structure and the Snakemake docs shows the following example:

```text
├── .gitignore
├── README.md
├── LICENSE.md
├── workflow
│   ├── rules
|   │   ├── module1.smk
|   │   └── module2.smk
│   ├── envs
|   │   ├── tool1.yaml
|   │   └── tool2.yaml
│   ├── scripts
|   │   ├── script1.py
|   │   └── script2.R
│   ├── notebooks
|   │   ├── notebook1.py.ipynb
|   │   └── notebook2.r.ipynb
│   ├── report
|   │   ├── plot1.rst
|   │   └── plot2.rst
|   └── Snakefile
├── config
│   ├── config.yaml
│   └── some-sheet.tsv
├── results
└── resources

```

This structure has the Snakefile and all it's associated scripts, environments, and rules in a single directory called `workflow/`.
We have a separate directory called `config/` which should house our configuration file for our run.
Nothing in `workflow/` should have to change depending on the individual runs. 
Any and all run-dependent files, parameters, variables etc should be defined in configuration files in the `config/` directory.
This separation of run-dependent and run-independent files is important for keeping everything clean and allowing
you to easily apply your pipeline to different datasets.

Lets rearrange the files to better conform to the best-practice directory structure.
Something like this:
```text
.
├── config
│   └── config.yaml
├── inputs
│   ├── genome.fa
│   └── reads
│       ├── A.fastq
│       ├── B.fastq
│       └── C.fastq
└── workflow
    ├── envs
    │   └── minimap2.yaml
    ├── scripts
    │   └── count_reads.py
    └── Snakefile

6 directories, 8 files
```

The path to the config file specified in the Snakefile is __relative to the working directory.__
We need to update the default path to the config file in `workflow/Snakefile` to:

```python
configfile: "config/config.yaml"
```

However, the paths to the various scripts and envs are __relative to the Snakefile__ and these have not changed.
We should also update the config file for the new `inputs/` directory:

```python
# config.yaml
genome: inputs/genome.fa
samples: ['A','B','C']
outputDirectory: output
readDirectory: inputs/reads
minimapParameters: -x sr
```

Interestingly, Snakemake recognises the structure and will find the Snakefile within workflow, so,
you can still run the entire pipeline with `snakemake -j 1` and not have to specify the Snakefile within `worklfow/`.



## Log files

If you scale up your pipeline to lots of threads and lots of samples you will have many steps running concurrently.
When you have jobs that fail it will dump the error messages to the terminal with everything else making debugging hard.
It's better to redirect error messages to a log file for each job.
This is very easy to do, simply declare a log file that uses the wildcard for a rule,
and in the command, redirect STDERR to the log file (with `2>`).

Let's update the minimap2 rule to include logging, 
and we'll split it over multiple lines as it's starting to get a bit long.

```python
# near the top of the file add:
log_dir = 'logs'

# update rule map_reads to log STDERR
rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        'envs/minimap2.yaml'
    log:
        log_dir + '/map_reads.{sample}.log'
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """
```

If one of your minimap2 jobs fails, Snakemake will tell you which log file to look at.



## Include

After a while, you might find that your pipeline is turning into a gigantic file of thousands of lines of code etc.
You can use the `include:` directive to drop in code from another file as if it were there in the main Snakefile.

Let's put all of our rules (except rule all) into their own file.
Make a directory called `rules/`, and create a file called `mapping.smk`.
Cut and past the rules into this new file and add an include statement in `Snakefile`.
__Don't forget to update the relative paths for scripts, conda environments etc.__

```python
# file: rules/mapping.smk
rule count_reads:
    input:
        read_dir + "/{sample}.fastq"
    output:
        out_dir + "/{sample}.reads.tsv"
    script:
        "../scripts/count_reads.py"

rule map_reads:
    input:
        genome = config['genome'],
        reads = read_dir + "/{sample}.fastq"
    output:
        temp(temp_dir + "/{sample}.sam")
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        '../envs/minimap2.yaml'
    log:
        log_dir + '/map_reads.{sample}.log'
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """

rule convert_sam:
    input:
        temp_dir + "/{file}.sam"
    output:
        out_dir + "/{file}.bam"
    shell:
        "samtools view -bh {input} > {output}"
```

Update your Snakefile to include this new rule file.
It doesn't matter if we add the 'include: rules/mapping.smk' above rule all, Snakemake will still recognise
"rule all" as the first rule to declare targets for the pipeline.

```python
# Snakefile
configfile: "config/config.yaml"

temp_dir = 'temp'
log_dir = 'logs'
out_dir = config['outputDirectory']
read_dir = config['readDirectory']

include: 'rules/mapping.smk'

rule all:
    input:
        expand(out_dir + "/{sample}.{file}", sample=config['samples'], file=['bam','reads.tsv'])
```


## Python's os.path

You can make use of Python's os.path library for managing file paths.
Notice in our pipeline we have to be careful about placing in our forward slashes,
especially when using variables as part of the file path?
What if your pipeline is designed to work on both Windows and Linux? 
You'd need to replace all those forward slashes with backslashes. 

Instead of writing your file paths like this:

```
out_dir + "/{file}.bam"
```

You can write like this:

```
os.path.join(out_dir, "{file}.bam")
```

and you don't have to worry about keeping track of where you need to include your slashes,
or what slashes to use.

__Update the pipeline to make use of os.path.join()__



## Input functions and lambda functions

Delving further into Python, you can write your own functions and use them in rules (or anywhere).
At the moment, we are assuming that the reads for sample "A" will be called "A.fastq".
We can let the user specify both the sample name, and its read file.
Update the config file to make the samples a dictionary, specifying sample name and read file like so:

```yaml
genome: inputs/genome.fa
samples:
  A: inputs/reads/A.fastq
  B: inputs/reads/B.fastq
  C: inputs/reads/C.fastq
outputDirectory: output
minimapParameters: -x sr
```

`config['samples']` will now be a dictionary where the keys are the sample names, and the values are the read files.
We need to make a list of the keys to use with declaring targets.
We do that with `list(config['samples'].keys())`, which will collect the keys for the dictionary,
and then convert it to a list.
We then write a function to find the read file based on the wildcard that will match the sample name.
Input functions take a single wildcards object, so you can design the functions with that in mind.

```python
# Snakefile
configfile: os.path.join('config', 'config.yaml')

temp_dir = 'temp'
log_dir = 'logs'
out_dir = config['outputDirectory']

# MAKE A NEW SAMPLE LIST
sample_dictionary = config['samples']
sample_list = list(sample_dictionary.keys())

# WRITE FUNCTION TO RETURN READS FILES OF SAMPLES
def reads_from_wildcards_sample(wildcards):
    return sample_dictionary[wildcards.sample]

include: os.path.join('rules', 'mapping.smk')

rule all:
    input:
        expand(os.path.join(out_dir, "{sample}.{file}"), sample=sample_list, file=['bam','reads.tsv'])
```

Then, to use this function update rules "map_reads" and "count_reads" in your "rules/mapping.smk" file.
__Don't end the function name with parenthesis.__
You want to pass the function itself, not the result of the function.

```python
rule count_reads:
    input:
        reads_from_wildcards_sample
    output:
        os.path.join(out_dir, "{sample}.reads.tsv")
    script:
        os.path.join("..", "scripts", "count_reads.py")

rule map_reads:
    input:
        genome = config['genome'],
        reads = reads_from_wildcards_sample
    output:
        temp(os.path.join(temp_dir, "{sample}.sam"))
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        os.path.join('..', 'envs', 'minimap2.yaml')
    log:
        os.path.join(log_dir, 'map_reads.{sample}.log')
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """
```

An alternative would be to embed the whole function into the rule itself using a Python lambda function.
Lambda functions are small anonymous functions using the syntax `lambda arguments : expression`.
In this case you don't need to write the `reads_from_wildcards_sample` function at all in your Snakefile;
instead you simply modify the "map_reads" and "count_reads" rules like so:

```python
rule count_reads:
    input:
        lambda wildcards: sample_dictionary[wildcards.sample]
    output:
        os.path.join(out_dir, "{sample}.reads.tsv")
    script:
        os.path.join("..", "scripts", "count_reads.py")

rule map_reads:
    input:
        genome = config['genome'],
        reads = lambda wildcards: sample_dictionary[wildcards.sample]
    output:
        temp(os.path.join(temp_dir, "{sample}.sam"))
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        os.path.join('..', 'envs', 'minimap2.yaml')
    log:
        os.path.join(log_dir, 'map_reads.{sample}.log')
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """
```

Lambda functions are preferable anywhere you need a function that won't be used in multiple places.



## Resources

If you're running your Snakemake pipeline on an HPC cluster 
you're going to want to use a Snakemake profile to manage job resources.
Resources can be anything, but typically you would define threads, memory, and runtime.
Resources for each rule are defined under 'Resources:' and 
Snakemake will manage your jobs according to any defaults that you pass on the command line or 
in your profile.
[Resources and profiles are discussed in depth in a previous blog post](https://fame.flinders.edu.au/blog/2021/08/02/snakemake-profiles-updated)
so I wont go into too much detail here.

Let's define some resources for our mapping rule, which often can consume lots of memory and
take a long time to complete.

```python
rule map_reads:
    input:
        genome = config['genome'],
        reads = lambda wildcards: sample_dictionary[wildcards.sample]
    output:
        temp(os.path.join(temp_dir, "{sample}.sam"))
    params:
        config['minimapParameters']
    conda:
        os.path.join('..', 'envs', 'minimap2.yaml')
    log:
        os.path.join(log_dir, 'map_reads.{sample}.log')
    benchmark:
        os.path.join(bench_dir, 'map_reads.{sample}.txt')
    threads:
        8
    resources:
        time = 480,
        mem_mb = 32000
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """
```

Now we're saying that the mapping rule can take up to 480 minutes to complete,
and it can consume up to 32000 mb of memory.
The threads and resources are used in your Snakemake profile to define the cpus,
runtime, and memory that are requested for that job when submitting to your scheduler.
If you don't pass default values and/or you run it locally (i.e. not using a profile), 
then the resources will be ignored when scheduling jobs.
Resources can be accessed in the shell commands etc. in the same way that inputs, output, 
threads, params etc can be. 
For instance, if we wanted to sort our BAM file and limit the memory used:

```python
rule convert_sam:
    input:
        os.path.join(temp_dir, "{file}.sam")
    output:
        os.path.join(out_dir, "{file}.bam")
    resources:
        mem_mb = 8000
    shell:
        """
        samtools view -bh {input} \
            | samtools sort -m {resources.mem_mb}M > {output}
        """
```




## Misc tweaks

__Snakemake version__

Snakemake has and is undergoing lots of changes and updates.
You can make sure that someone running your pipeline is using the correct version of Snakemake with "min_version"
like so:

```python
# Snakefile
from snakemake.utils import min_version

min_version("6.10.0")
```

__Benchmarking__

You can collect resource usage statistics for rules using the "benchmark:" declaration in a rule like so:

```python
bench_dir = 'benchmarks'
rule map_reads:
    input:
        genome = config['genome'],
        reads = lambda wildcards: sample_dictionary[wildcards.sample]
    output:
        temp(os.path.join(temp_dir, "{sample}.sam"))
    threads:
        8
    params:
        config['minimapParameters']
    conda:
        os.path.join('..', 'envs', 'minimap2.yaml')
    log:
        os.path.join(log_dir, 'map_reads.{sample}.log')
    benchmark:
        os.path.join(bench_dir, 'map_reads.{sample}.txt')
    shell:
        """
        minimap2 -t {threads} -a {params} \
            {input.genome} {input.reads} \
            2> {log} > {output}
        """
```

The columns are (from the Snakemake documentation):
CPU time (in seconds),
wall clock time,
memory usage (RSS, VMS, USS, PSS in megabytes),
CPU load (CPU time divided by wall clock time),
I/O (in bytes)

__Wildcard constraints__

Snakemake uses regular expression for wildcard matching.
Occasionally you might have multiple rules able to create the same file according to the wildcard matches.
In these situations you can use wildcard constraints to make sure each rule is matching the correct 
inputs and outputs. e.g. for sample A, group 1 bam file could be written as 'A.1.bam'.
Some rules might be able to match 'A.1' as a sample name, which would be incorrect.
You can constrain the sample wildcard (and or group wildcard) like so:

```python
rule blagh:
    output:
    '{sample,[a-zA-Z0-9]+}.{group}.bam'
# or 
rule blagh:
    output:
        '{sample}.{group}.bam'
    wildcard_constraints:
        sample="[a-zA-Z0-9]+"
```

__More sample options: Use a .TSV file__

Instead of reading in samples from the configuration file, it's probably easier to define them as a 
tab-separated file.

```text
# inputs/samples.tsv
samples	reads
A	inputs/reads/A.fastq
B	inputs/reads/B.fastq
C	inputs/reads/C.fastq
```

Point to the tsv file in your config file (instead of the samples in a yaml dictionary).

```yaml
# config.yaml
genome: inputs/genome.fa
samples: inputs/samples.tsv
outputDirectory: output
minimapParameters: -x sr
```

Read it in and access it as a pandas dataframe, rather than a python dictionary
(or you could write a function to parse samples.tsv into a python dictionary).

```python
# Snakefile
import pandas as pd

configfile: os.path.join('config', 'config.yaml')

samples = pd.read_table(config['samples']).set_index("samples", drop=False)
sample_list = samples['samples'].tolist()
```

Update the lambda functions for the pandas dataframe:

```python
rule count_reads:
    input:
        lambda wildcards: samples.loc[wildcards.sample].reads
    output:
        os.path.join(out_dir, "{sample}.reads.tsv")
    script:
        os.path.join("..", "scripts", "count_reads.py")
```

__More sample options: Glob them from a directory__

You can specify a directory and glob all the fastq files etc to infer all the samples for your analysis.

```yaml
# config.yaml
genome: inputs/genome.fa
sample_dir: inputs/reads/
outputDirectory: output
minimapParameters: -x sr
```

```python
# Snakefile

configfile: os.path.join('config', 'config.yaml')

sample_list, = glob_wildcards(os.path.join(config['sample_dir'],'{sample}.fastq'))
sample_dictionary = {}
for sample in sample_list:
    sample_dictionary[sample] = os.path.join(config['sample_dir'],f'{sample}.fastq')
```

## Example pipeline

[Here is the Snakemake version of the template bash script](https://github.com/beardymcjohnface/snakemake-example-workflow-2024)
