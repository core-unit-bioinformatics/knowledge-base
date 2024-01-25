# Setup nf-core workflows

Table of contents

1. [What is Nextflow and Nextflow core?](#Whatis)
2. [Running nf-core workflows](#Runningnf)
   0. [Install nextflow and nf-core](#installation)
   1. [Download the workflow / offline usage](#Downloadeverything)
   2. [Configuration of workflow](#Configurationof)
      1. [Creating a parameter file](#Creatinga)
      2. [Predefined configuration profiles](#Predefconfig)
      3. [Resource Caps](#Rescaps)
      4. [Optional: Modify resources for specific processes](#Modres)
      5. [Optional: Extra arguments for processes](#Exttraarg)
   3. [Running the workflow](#Runningthe)

## What is Nextflow and Nextflow core? <a id="Whatis"></a>

[Nextflow](https://www.nextflow.io/) is a workflow scripting language commonly used in bioinformatic applications.
Around nextflow, a large community of researchers develops standards for pipelines and provides a big catalogue of curated workflows called [nextflow core (nf-core)](https://nf-co.re/).
The nf-core community also provides a toolbox package `nf-core` for creating and running workflows, available through [pip](https://pypi.org/project/nf-core/) and [bioconda](https://anaconda.org/bioconda/nf-core).

## Running nf-core workflows <a id="Runningnf"></a>

You were browsing through the [nf-core workflow catalogue](https://nf-co.re/pipelines) and found the desired workflow?
Great! The next steps guide you through the process of running it on CUBI infrastructure.

### 0. Install nextflow und nf-core packages in conda environment  <a id="installation"></a>

To avoid annoying dependency issues the `nextflow` and `nf-core` packages should be installed and run via a conda environment.
A working `yml` file for which this guide was tested looks like this:

```
name: nextflow-env
dependencies:
  - Python=3.10.*
  - mamba=1.5.6
  - nextflow=23.10.1
  - nf-core=2.11.1
  - singularity=3.7.1
```

### 1. Download the workflow / offline usage <a id="Downloadeverything"></a>

It is recommended to download the workflow, also for offline usage.
For offline usage of Nextflow, also add `export NXF_OFFLINE='TRUE'` to your `.bashrc` or scripts to avoid nextflow looking online for updates.
More information at [https://nf-co.re/docs/usage/offline](https://nf-co.re/docs/usage/offline).

The download can be automated using the `nf-core download` command, which is part of the nf-core tools package and prompts you through the necessary options for downloading.
You can also specify them through the CL (Command Line).

NOTE: Nextflow can store and pull previously downloaded singularity images from a local cache folder, which is recommended to set using the `NXF_SINGULARITY_CACHEDIR` environemnt variable (also see [here](https://www.nextflow.io/docs/latest/singularity.html#singularity-docker-hub))

Here is an example for the nf-core bamtofastq workflow:

```bash
nf-core download bamtofastq --revision 2.1.0 --compress none --container-system singularity
```

This will create the following three folders within `nf-core-bamtofastq-2.1.0/`:

1. `workflow/`: Workflow code from github repository.
2. `singularity`: Containers for workflow processes, e.g., singularity containers.
3. `configs/`: basic configs and preconfigured [institutional configs](https://github.com/nf-core/configs).

Moreover, some workflows need additional databases.
Most nf-core workflows use the human reference genome that is provided for nf-core workflows through the AWS iGenomes collection.
More information for usage with nf-core at [nf-core reference genomes](https://nf-co.re/docs/usage/reference_genomes).
You can get the download command for specific datasets from [AWS-iGenomes](https://ewels.github.io/AWS-iGenomes/).
For example, downloading the human reference genome GRCh38 assembled for usage with GATK:

```bash
aws s3 --no-sign-request --region eu-west-1 sync s3://ngi-igenomes/igenomes/Homo_sapiens/GATK/GRCh38/ db/references/Homo_sapiens/GATK/GRCh38/
```

Within your nextflow parameters, you need to specify the igenomes base directory and the reference genome using the parameters `igenomes_base` (pointing to the iGenomes directory, e.g. `db/references/`) and `genome` (Name of reference downloaded, e.g. `GATK.GRCh38`).
If `igenomes_base` is not defined, it will automatically catch it from the internet source defined in `conf/igenomes.config`.  
Further, some workflows require workflow-specific references that need to be separately downloaded, e.g., the VEP cache in sarek.

### 2. Configuration of workflow <a id="Configurationof"></a>

For generally configuring nextflow workflows for your infrastructure, see the [nextflow config tips](tech/nextflow-config.md).

### 2.1 Creating a parameter file <a id="Creatinga"></a>

The params file is required for configuring your nf-core workflow.
For this, you can use the `nf-core launch` command.
For example, after you downloaded the bamtofastq workflow:

```
nf-core launch -x -a nf-core-bamtofastq-2.1.0/workflow/
```

The `-a` and `-x` flags ensure, all parameters will be displayed and configurable.  
You will be guided through a web or CL-based interface to configure all parameters.
In the end, it will create the `nf-params.json`, which can be provided via the -params-file flag.

For choosing the right parameters and writing the `samplesheet.csv`, you can read the documentation of the respective workflow, e.g., for bamtofastq at [https://nf-co.re/bamtofastq](https://nf-co.re/bamtofastq).

#### 2.2 Predefined configuration profiles <a id="Predefconfig"></a>

Several configuration profiles are [predefined for nf-core workflows](https://nf-co.re/usage/configuration#basic-configuration-profiles) that specialize on usage with container systems.
Important are profiles as `singularity`, `docker`, or `conda` for usage of containers in the workflow.
For example, if you downloaded the workflow using singularity images, you need to enable the singularity profile with `-profile singularity`.  
Also, some pipelines include the `test` profile for running automatic test samples (internet connection required) once you set up the workflow.
These test profiles usually require internet connection.

#### 2.3 Resource Caps <a id="Rescaps"></a>

Nf-core workflows allow the definition of resource _caps_ that prevent nextflow to submit jobs with higher resources as required.
These should exceptionally be configured in the `run.config` file as shown here, as these are specific for the infrastructure.
The recommended use is within the infrastructure-specific profiles, e.g., in the `hilbert` profile:

```
params {
    // maximum HPC job resources for jobs
    max_memory = 200.GB
    max_cpus = 32
    max_time = 72.h
}
```

More information at [max-resources](https://nf-co.re/docs/usage/configuration#max-resources).

#### 2.4 Optional: Modify resources for specific processes <a id="Modres"></a>

The resource requirements of nextflow processes are specified in process labels which are defined in the `base.config` file within the workflow.
If a process exits because of lacking resources, Nextflow automatically retries the process with doubled resources until it reaches the specified `max_memory`, `max_cpus` or `max_time` values. Hence, you can increase these parameters and restart the process.

To avoid long runtimes, e.g., due to several retries by nextflow or too low numbers of CPU cores for big datasets, you can also increase the resource requirements for specific processes in the _process_ scope within a separate `run.config` file.

Each process contains a label that specifies its resource requirements (see `nextflow.config` file) that can be overwritten, e.g., like this for the medium label:

```
process {
  withLabel:process_medium {
    cpus = 32
  }
}
```

More information at [tuning-workflow-resources](https://nf-co.re/docs/usage/configuration#tuning-workflow-resources)

#### 2.5 Optional: Extra arguments for processes <a id="Exttraarg"></a>

For most processes in nf-core workflow and modules you can also define extra arguments or flags for the running command using the process name and the `ext.args` argument. For example, we can define another flag for the VEP process in sarek:

```
process {
	withName: 'ENSEMBLVEP' {
    	ext.args         = '--everything'
    }
}
```

More information about nf-core configurations can be found at [https://nf-co.re/usage/configuration](https://nf-co.re/usage/configuration).

### 3. Running the workflow <a id="Runningthe"></a>

You downloaded all files, specified your parameters in `nf-params.json` and added the infrastructure configuration as separate profile in `run.config`?
Great! Now, you can run your workflow using the `nextflow run` command, e.g., for the bamtofastq workflow on HILBERT:

```
nextflow run nf-core-bamtofastq-2.1.0/workflow \
-profile singularity,hilbert \
-c run.config \
-params-file nf-params.json
```

> NOTE: If the workflow exits unsuccessfully, e.g. due to a wrongly specified parameter, you can relaunch this command by adding the `-resume` flag.

For debugging, you can also have a look at the log file `.nextflow.log` or into the `pipeline_info/` folder within your results directory.
