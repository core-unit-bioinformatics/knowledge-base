# Running nf-core workflows

## What is Nextflow and Nextflow core?

[Nextflow](https://www.nextflow.io/) is a workflow scripting language commonly used in bioinformatic applications. 
Around nextflow, a large community of researchers develops standards for pipelines and provides a big catalogue of curated workflows called [nextflow core (nf-core)](https://nf-co.re/).
The nf-core community also provides a toolbox package `nf-core` for creating and running workflows, available through [pip](https://pypi.org/project/nf-core/) and [bioconda](https://anaconda.org/bioconda/nf-core).

## Running nf-core workflows

You were browsing through the [nf-core workflow catalogue](https://nf-co.re/pipelines) and found the desired workflow? 
Great! The next steps guide you through the process of running it on CUBI infrastructure.

### 1. Download everything necessary for the workflow

It is recommended to download the workflow, also for offline usage. 
More information at [https://nf-co.re/docs/usage/offline](https://nf-co.re/docs/usage/offline).
This is quickly done using `nf-core download`, which prompts you through the necessary options for downloading.
Tou can also specify them through the CL, here is an example for the nf-core sarek workflow:

```bash
nf-core download sarek --revision 3.1.2 --compress none --container singularity
```

This will create the following three folders within `nf-core-sarek-3.1.2/`:

1. `workflow/`: Workflow code from github repository.
2. `singularity`: Containers for workflow processes, e.g., singularity containers.
3. `configs/`: preconfigured [institutional configs](https://github.com/nf-core/configs).

Moreover, some workflows need additional databases.
Most workflows use the human reference genome that is provided for nf-core workflows through the AWS iGenomes collection.
More information for usage with nf-core at [https://nf-co.re/docs/usage/reference_genomes](https://nf-co.re/docs/usage/reference_genomes).
You can get the download command for specific datasets from [https://ewels.github.io/AWS-iGenomes/](https://ewels.github.io/AWS-iGenomes/).
For example, downloading the human reference genome GRCh38 assembled for usage with GATK:

```bash
aws s3 --no-sign-request --region eu-west-1 sync s3://ngi-igenomes/igenomes/Homo_sapiens/GATK/GRCh38/db/references/Homo_sapiens/GATK/GRCh38/
```
 
Within your nextflow parameters, you need to specify the igenomes base directory and the reference genome using the parameters `igenomes_base` (pointing to the iGenomes directory, e.g. `db/references/`) and `genome` (Name of reference downloaded, e.g. `GATK.GRCh38`).
If `igenomes_base` is not defined, it will automatically catch it from the internet source defined in `conf/igenomes.config`.  
Further, some workflows require workflow-specific references that need to be separately downloaded, e.g., the VEP cache in sarek.

### 2. Configuration of workflow

Nextflow comes with a great number of possibilities to provide configuration of your workflow, which can be very powerful (and sometimes confusing).
The configuration for Nextflow workflows can be defined on different levels and are applied with the following priority:  
(Source: [https://www.nextflow.io/docs/latest/config.html](https://www.nextflow.io/docs/latest/config.html))

> 1. Parameters specified on the command line (--something value)
> 2. Parameters provided using the -params-file option
> 3. Config file specified using the -c my_config option
> 4. The config file named nextflow.config in the current directory
> 5. The config file named nextflow.config in the workflow project directory
> 6. The config file $HOME/.nextflow/config
> 7. Values defined within the pipeline script itself (e.g. main.nf)

Generally, the following structure for storing configuration is recommended:

- Store configurations in separate `run.config` file, e.g., for adapting to Infrastructure or modifying processes. (3.)
- Store workflow parameters in `nf-params.json` file, e.g., to provide samplesheets. (2.)
- For testing parameters or options, you can provide them using the command line flags. (1.)

Options 4.-7. are mostly relevant for workflow development, e.g., to set default parameters or preconfigure processes.
Within nf-core workflows, it is not necessary to change the nextflow.config file within the pipeline.

#### 2.1 Creating a parameter file

Use the `nf-core launch` command!
For example, after you downloaded the sarek workflow:
```
nf-core launch -x -a nf-core-sarek-3.1.2/workflow/
```
The -a and -x flags ensure, all parameters will be displayed and configurable.  
You will guide you through a web or CL-based interface to configure all parameters.
In the end, it will create the `nf-params.json`, which can be provided via the -params-file flag.

For choosing the right parameters and writing the `samplesheet.csv`, you can read the documentation of the respective workflow, e.g., for sarek at [https://nf-co.re/sarek](https://nf-co.re/sarek).

#### 2.2 Configuration for your Infrastructure

The previously mentioned institutional configs contain infrastructure-specific configurations.
For now, no HHU or UKD config file exists, but may be added in the future.  
> **Important:** You should not store parameters in teh `run.config` file, except those specifying resource requirements!

The config file consists of _scopes_ that organizes and groups configuration settings (more information at [https://www.nextflow.io/docs/latest/config.html#config-scopes](https://www.nextflow.io/docs/latest/config.html#config-scopes)).  
Two of the most important scopes are _params_ and _process_:

**_process_ - Executors for HPC systems and process configs**

In the _process_ scope, you can define specific parameters for the executors and workflow processes.  
A large variety of executor systems is supported and this config is set to _local_ per default.
(More information at [https://www.nextflow.io/docs/latest/executor.html#executor-page](https://www.nextflow.io/docs/latest/executor.html#executor-page))  

**_params_ - Parameters of the workflow**

In the _params_ scope, you can define parameters utilized by the workflows, for example the output directory or maximum resources.  
**In general, you should NOT define your parameters in the `run.config` file, but within the `nf-params.json` file.**  
More information at [https://www.nextflow.io/docs/latest/config.html#scope-params](https://www.nextflow.io/docs/latest/config.html#scope-params)

**Configuration profiles**

Within configuration files, you can define profiles that are enabled using the `-profile` flag in the nextflow command.  
(More information at [https://www.nextflow.io/docs/latest/config.html#config-profiles](https://www.nextflow.io/docs/latest/config.html#config-profiles))  

Several configuration profiles are [predefined for nf-core workflows](https://nf-co.re/usage/configuration#basic-configuration-profiles) that specialize on usage with container systems, but for some pipelines also include the `test` profile for running automatic test samples (internet connection required).  
Important are profiles as `singularity`, `docker` or `conda`  for usage of containers in the workflow.

**Configuration file for HILBERT and local runs**

Here is a working configuration file defining two profiles, one for execution on [HILBERT](https://www.zim.hhu.de/forschung/high-performance-computing), the other for local execution:


```
profiles {
    hilbert {
        params {
            config_profile_description = 'HILBERT'
            // maximum HPC job resources for jobs
            max_memory = 200.GB
            max_cpus = 32
            max_time = 72.h
        }
        process {
            executor = 'pbspro'
            module = 'Singularity/3.7.1'
            queuesize = 100 
            clusterOptions = '-A "PROJECT"'
        }
    }
    local {
        params {
            config_profile_description = 'local computer'
            // maximum local resources for processes
            max_memory = 40.GB
            max_cpus = 6
            max_time = 72.h
        }
        process {
            executor = 'local' #default
        }
    }
}
```
**process configurations:**
- **executor**: specifies the HPC system for job submissions.  
- **module**: The submitted jobs can load environmental modules on the HPC system with the `module load` command. For example, if the singularity profile is enabled, singularity needs to be loaded with every single job.  
- **queuesize**: Maximum number of jobs submitted in parallel. Avoids nextflow going rogue.  
- **clusterOptions**: Strings that will be attached to the `qsub` command. At HILBERT, we need to specify the project for each submitted job using the `-A "PROJECT"` flag.  

Nf-core workflows allow the definition of resource _caps_ that prevent nextflow to submit jobs with higher resources as required.
These can exceptionally be configured in the `run.config` file as shown here, as these are specific for the infrastructure.
More information at [https://nf-co.re/docs/usage/configuration#max-resources](https://nf-co.re/docs/usage/configuration#max-resources).

> NOTE: You can modify the specific process resource requirements in these config files within the _process_ scope.
> Each process contains a label that specifies its resource requirements (see `nextflow.config` file) that can be overwritten:

```
process {
  withLabel:process_medium {
    cpus = 32
  }
}
```

> NOTE: For some nf-core workflow you can also define extra arguments or flags for single processes using its name and the `ext.args` argument. For example, we can define another flag for the VEP process in sarek:

```
process {
	withName: 'ENSEMBLVEP' {
    	ext.args         = '--everything'
    }
}
```

> NOTE: by adding the line `cleanup = true` outside of any scope within `run.config`, the work directory with temporary files will be automatically deleted after a successful run.

More information about nf-core configurations can be found at [https://nf-co.re/usage/configuration](https://nf-co.re/usage/configuration).

### 3. Running the workflow

You downloaded all files, specified your parameters in `nf-params.json` and added the infrastructure configuration?
Great! Now, you can run your workflow using the `nextflow run` command, e.g., for the sarek workflow on HILBERT:

```
nextflow run nf-core-sarek-3.1.2/workflow \
-profile singularity,hilbert \
-c run.config \
-params-file nf-params.json
```

> NOTE: If the workflow exits unsuccessfully, e.g. due to a wrongly specified parameter, you can relaunch this command by adding the `-resume` flag.

For debugging, you can allso have a look at the log file `.nextflow.log` or into the `pipeline_info/` folder within your results directory.