% Creating our pipeline
% John Salamon
% Aug 28, 2024

# Creating our project dir

- It's important to distinguish between launch vs project directories
- We can access the path to our project directory via [workflow introspection](https://www.nextflow.io/docs/latest/metadata.html)
- Special directories (`lib`, `bin`, and `templates`)
  - any Java or Groovy code you place in `lib` will be loaded in your pipeline
  - any executables you place in `bin` can be executed in your scripts
  - any scripts you place in `templates` can be executed using `script: template <script>`, instead of an in-line script.

# A brief word on configuration

- Configs use `key = value` syntax, and can be simplified with config scopes
- There is a [hierarchy of locations](https://www.nextflow.io/docs/latest/config.html) that Nextflow will look in for config files.
- We'll just make a basic `nextflow.config` in our project dir to set up the resources we want to allocate:

```
executor {
    name = 'local'
    cpus = 2
    memory = '4 GB'
}
```
We can also specify our `conda` environment, if we wish:

```
conda.enabled = true

// This is actually a per-process option, as we may have many different envs
process.conda = "assets/environment.yml"
```


# Getting files as inputs, and using params

- we can create pipeline parameters using `params`, and set a default

```
params.samplesheet = 'assets/samplesheet.csv'
```

- We can then modify this on the command line

```
nextflow main.nf --samplesheet 'assets/samplesheet.csv'
```

- channel factories like `fromPath` can create channels from files.
- `view` lets us see the content of a channel
- `map` to manipulate our channel with closures
- We could do basic reading of a samplesheet with `splitCsv`:

```
channel.of( file('samplesheet.csv') )
        | splitCsv( header: true )
```

# Adding metadata to channels

- A super common pattern in Nextflow is use tuples to add sample metadata
    - Tuples ensure that our metadata and data stay linked
    - They also let us keep only a single parameter 
- The only difficulty is remembering the order of the tuples in outputs
- If you have many outputs in *different* channels, you can name them

```
input:
tuple val(id), path(reads)

output:
tuple val(id), path('*.json'), emit: json
```

# Specifying resources in config

- Annotating processing with labels can allow you to apply rules to many processes at once. In process:

```
label 'process_low'
```

- In config:

```
process {
    withLabel: 'process_low' {
        cpus = 1
    }
}
```

Config options can be accessed via `task`:
```
"""
# map reads
echo "mapping reads"
minimap2 -t ${task.cpus} -a -x sr ${genome} ${reads} > ${id}.sam 2> ${id}.minimap2.log
"""
```


# Getting results out

- How do we make sure all our output is in the right place?
- A common method is to use [`publishDir`](https://www.nextflow.io/docs/latest/process.html#publishdir):

```
publishDir "${params.outdir}", pattern: "*.html"
```


