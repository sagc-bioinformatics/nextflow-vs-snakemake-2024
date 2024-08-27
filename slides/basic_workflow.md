% Writing a basic workflow
% John Salamon
% Aug 28, 2024

# Getting started

- Nextflow code is written in `.nf` files
- Let's start with an empty workflow definition. Create `main.nf`:

```
workflow {

}
```

# Running our workflow

- To run a Nextflow pipeline on the cli, you need to use the `nextflow` command
- This is already set up on your VMs
- Run your workflow by typing `nextflow run main.nf`:

```
 N E X T F L O W   ~  version 24.04

Launching `main.nf` [exotic_brenner] DSL2 - revision: 11d0d8e692
```

- What happened?
    - `.nextflow.log` was created
    - `work/` directory was created

# Adding a process

- Now let's add a process. We can add this anywhere in `main.nf`, just not inside the workflow definition:

```
process EXAMPLE {
    "echo hello > out.txt"
}

workflow {
    EXAMPLE()
}
```

- A process can consist of a nothing more than a string containing some code, to be executed by `bash` by default
- By convention, process names tend to be upper case, we've named ours EXAMPLE
- We've *invoked* the process like it's a function in our workflow
    - Critically, although the syntax is similar, this is not the same thing as calling a function 


# Running our process again

- We can re-run our workflow: `nextflow run main.nf`:

```
 N E X T F L O W   ~  version 24.04.2

Launching `main.nf` [romantic_snyder] DSL2 - revision: 5ef5ed8a25

executor >  local (1)
[ee/c693f3] EXAMPLE [100%] 1 of 1 ✔
```

- Nextflow gives us a summary of which processes were run
- Why does EXAMPLE run?
    - There is no input defined, so as per the dataflow paradigm, it can run immediately (all inputs are valid)
- The output of our process is found in an isolated subdirectory of `work/`:

```
work/
└── ee
    └── c693f3241dcd5e3576e575b62ce5e8
        └── out.txt
```

- we know what is stored under `work/` by inspecting the process hash (you can see those as you run the pipeline)


# Checking logs of previous executions

- Use `nextflow log` to see the log of previous executions of your script
- each *execution* of our script is assigned a randomly-generated *run_name*
- Inspect details of a specific execution:

```
$ nextflow log romantic_snyder -f 'process,exit,hash,duration'
EXAMPLE	0	ee/c693f3	6ms
```

- See available fields with `nextflow log -l`

# Defining inputs

- We can define our process's inputs with an [*input block*](https://www.nextflow.io/docs/latest/process.html#inputs):

```
process EXAMPLE {
    
    input:
    val msg

    "echo $msg > out.txt"
}
```

- [the `val` input type](https://www.nextflow.io/docs/latest/process.html#input-type-val) can be any type (string, integer, etc)
- our input is named "msg"
- the value of `msg` is inserted into our script string using the dollar sign: `$msg`


# Defining outputs

- [the `path` input type](https://www.nextflow.io/docs/latest/process.html#input-type-path) represents a file or directory path

```
process EXAMPLE {

    input:
    val msg

    output:
    path "out.txt"

    "echo $msg > out.txt"
}
```

- We declare the name of our output file, "out.txt"
- This file will be emitted by our output channel

# Script block

- We can omit explicitly typing script as long as it is the last line in the process
- We can make the script multi-line, with `"""`
- We can actually use any code, as long as it returns a String at the end

```
process EXAMPLE {

    input:
    val msg

    output:
    path "out.txt"
    
    script:
    """
    echo $msg > out.txt
    """
}
```

# Defining channels in our workflow

- We can pass an input value directly to our process, which implicitly creates a [*value channel*](https://www.nextflow.io/docs/latest/channel.html#value-channel):

```
workflow {
    EXAMPLE( "hello world" )
}
```

- Here's the same thing stated explicitly using [channel factory methods](https://www.nextflow.io/docs/latest/channel.html#channel-factories):

```
workflow {
   
    ch_input = channel.value( "hello world" ) 
    EXAMPLE( ch_input )
}
```

- The other main type of channel is a [queue channel](https://www.nextflow.io/docs/latest/channel.html#queue-channel), which we need to create explicitly:

```
workflow {
   
    ch_input = channel.of( "one", "two", "three" )
    EXAMPLE(ch_input)
}
```

# Order of execution is not guaranteed

Let's try adding in the `view` operator and `println` function in different places...

# In the next section

- Let's start building a proper pipeline!

<meta name="copyright" 
content="Basic workflow"/> 
