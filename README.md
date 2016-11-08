# NGI-MethylSeq
Methylation (BS-Seq) Best Practice pipeline for bisulfite sequencing data analysis at
the [National Genomics Infastructure](https://ngisweden.scilifelab.se/)
at [SciLifeLab Stockholm](https://www.scilifelab.se/platforms/ngi/), Sweden.

# UNDER DEVELOPMENT
This pipeline is currently under active development and has very little in the way of testing.
There's a pretty good chance that it won't run. Any help debugging is very welcome! Please
fork, make changes and submit a pull request.

## Pipeline Results
See the [pipeline documentation](docs/README.md) for explanations of
the results files.

## Installation
### NextFlow installation
See https://github.com/SciLifeLab/NGI-NextflowDocs for instructions on how to install and configure
Nextflow.

### Pipeline installation
This pipeline itself needs no installation - NextFlow will automatically fetch it from GitHub when run if
`SciLifeLab/NGI-MethylSeq` is specified as the pipeline name.

If you prefer, you can download the files yourself from GitHub and run them directly:
```
git clone https://github.com/SciLifeLab/NGI-MethylSeq.git
nextflow NGI-MethylSeq/main.nf
```

## Configuration
By default, the pipeline is configured to run on the Swedish UPPMAX cluster (milou / irma).

You will need to specify your UPPMAX project ID when running a pipeline. To do this, use
the command line flag `--project <project_ID>`.

To avoid having to specify this every time you run Nextflow, you can add it to your
personal Nextflow config file instead. Add this line to `~/.nextflow/config`:

```groovy
params.project = 'project_ID'
```

The pipeline will exit with an error message if you try to run it pipeline with the default
UPPMAX config profile and don't set project.


### Running on other clusters
It is entirely possible to run this pipeline on other clusters, though you will need to set up
your own config file so that the script knows where to find your reference files and how your
cluster works.

Copy the contents of [`conf/uppmax.config`](conf/uppmax.config) to your own config file somewhere
and then reference it with `-c` when running the pipeline.

If you think that there are other people using the pipeline who would benefit from your configuration
(eg. other common cluster setups), please let us know. It should be easy to create a new config file
in `conf` and reference this as a named profile in [`nextflow.config`](nextflow.config). Then these
configuration options can be used by specifying `-profile <name>` when running the pipeline.


## Running the pipeline
The typical command for running the pipeline is as follows:
```
nextflow SciLifeLab/NGI-MethylSeq --reads '*_R{1,2}.fastq.gz'
```

Note that the pipeline will create files in your working directory:
```bash
work            # Directory containing the nextflow working files
results         # Finished results (configurable, see below)
.nextflow_log   # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```

### `--reads`
Location of the input FastQ files:
```
 --reads 'path/to/data/sample_*_{1,2}.fastq'
```

**NB: Must be enclosed in quotes!**

Note that the `{1,2}` parentheses are required to specify paired end data. Running `--reads '*.fastq'` will treat
all files as single end. Also, note that the file path should be in quotation marks to prevent shell glob expansion.

If left unspecified, the pipeline will assume that the data is in a directory called `data` in the working directory.

### `--genome`
The reference genome to use of the analysis, needs to be one of the genome specified in the config file.
The human `GRCh37` genome is used by default.

See [`conf/uppmax.config`](conf/uppmax.config) for a list of the supported reference genomes
and their keys. Common genomes that are supported are:

* Human
  * `--genome GRCh37`
* Mouse
  * `--genome GRCm38`
* Drosophila
  * `--genome BDGP6`
* _S. cerevisiae_
  * `--genome 'R64-1-1'`

> There are numerous others - check the config file for more.

### Trimming Parameters
The pipeline accepts a number of parameters to change how the trimming is done, according to your data type.
The following settings are set for these command line flags:

| Parameter       | 5' R1 Trim | 5' R2 Trim | 3' R1 Trim | 3' R2 Trim |
|-----------------|------------|------------|------------|------------|
| `--pbat`        | 6          | 6          | 0          | 0          |
| `--single_cell` | 9          | 9          | 0          | 0          |
| `--epignome`    | 6          | 6          | 6          | 6          |
| `--accel`       | 10         | 15         | 10         | 10         |
| `--cegx`        | 6          | 6          | 2          | 2          |

You can specify custom trimming parameters as follows:

* `--clip_r1 <NUMBER>`
* `--clip_r2 <NUMBER>`
* `--three_prime_clip_r1 <NUMBER>`
* `--three_prime_clip_r2 <NUMBER>`

Finally, specifying `--rrbs` will pass on the `--rrbs` parameter to TrimGalore!

### Bismark Parameters
Using the `--pbat` parameter will affect the trimming (see above) and also set the `--pbat` flag when
aligning with Bismark.

Using the `--single_cell` parameter will set the `--non_directional` flag when aligning with Bismark.
This can also be set with `--non_directional` (doesn't affect trimming).

Use the `--unmapped` flag to set the `--unmapped` flag with Bismark align and save the unmapped reads.

### Deduplication
By default, the pipeline includes a deduplication step after alignment. If you would like to skip this
step (eg. for RRBS data), use the `--nodedup` command line option.

### `--index`
If you prefer, you can specify the full path to your reference genome when you run the pipeline:
```
--index [path to Bismark index]
```

### `--outdir`
The output directory where the results will be saved.

### `-c`
Specify the path to a specific config file (this is a core NextFlow command). Useful if using different UPPMAX
projects or different sets of reference genomes.

## Credits
These scripts were written for use at the [National Genomics Infrastructure](https://portal.scilifelab.se/genomics/)
at [SciLifeLab](http://www.scilifelab.se/) in Stockholm, Sweden.
Written by Phil Ewels (@ewels) and Rickard Hammarén (@Hammarn).

<p align="center"><a href="stand_alone/http://www.scilifelab.se/" target="_blank"><img src="docs/images/SciLifeLab_logo.png" title="SciLifeLab"></a></p>
