# Nextflow Workflows

## What is Nextflow and Nextflow core?

[Nextflow](https://www.nextflow.io/) is a workflow scripting language commonly used in bioinformatics. 
Around Nextflow, a large community of researchers developed standards for pipelines and provides a big cataloque of (often) continuously maintained workflowsm, called [nextflow core (nf-core)](https://nf-co.re/).

## Configuration of Nextflow pipelines

The configuration for Nextflow workflows can be defined on different levels and are applied with the following priority:  
(Source: [https://www.nextflow.io/docs/latest/config.html](https://www.nextflow.io/docs/latest/config.html))

```
1. Parameters specified on the command line (--something value)
2. Parameters provided using the -params-file option
3. Config file specified using the -c my_config option
4. The config file named nextflow.config in the current directory
5. The config file named nextflow.config in the workflow project directory
6. The config file $HOME/.nextflow/config
7. Values defined within the pipeline script itself (e.g. main.nf)
```

1.-3. are commonly used to configure a workflow run.
4., 5. & 7. are commonly used in building workflows and defining standards.

Within nf-core workflows, it is not necessary to change the nextflow.config files within the pipeline.
More information about nf-core configuration can be found at [https://nf-co.re/usage/configuration](https://nf-co.re/usage/configuration).

### Configuration files

The config file consists of _scopes_ that organizes and groups configuration settings (more Information at [https://www.nextflow.io/docs/latest/config.html#config-scopes](https://www.nextflow.io/docs/latest/config.html#config-scopes)).  
Two of the most important are _params_ and _process_.

#### _process_ - Executors for HPC systems

In the _process_ scope, you can define specific parameters for the workflow processes.  
These include the configuration of the executor system, that supports a large variety of systems and is is set to _local_ in default.
(More information at [https://www.nextflow.io/docs/latest/executor.html#executor-page](https://www.nextflow.io/docs/latest/executor.html#executor-page))  

At CUBI, we use a PBSPro system configured like the following:

```
process {
	// Adapted for HPC using qsub argument for project code.
	executor = 'pbspro'
	module = 'Singularity/3.7.1'
	queuesize = 100
	clusterOptions = '-A "Project"'
}
```
**Parameters:**
- **module:** The submitted jobs can load environmental modules on the HPC system with the `module load` command.
For example, if the singularity profile is enabled, singularity needs to be loaded with every single job.
- **queuesize:** Maximum number of jobs submitted in parallel. Avoids nextflow going rogue.
- **clusterOptions:** Strings that will be attached to the `qsub` command. At HILBERT, we need to specify the project for each submitted job using the `-A "Project"` flag.

For some nf-core workflow you can also define extra arguments or flags for single processes using its name and the `ext.args` argument, e.g. defining another flag for the vep command line in Sarek:
```
process {
	withName: 'ENSEMBLVEP' {
    	ext.args         = '--everything'
    }
}
```

#### _params_ - Parameters for the workflow

In the _params_ scope, you can define parameters utilized by the workflows, for example the maximum resources. Nf-core workflows allow the definition of resource _caps_ that prevent nextflow to submit jobs with higher resources as required.  
More information at [https://www.nextflow.io/docs/latest/config.html#scope-params](https://www.nextflow.io/docs/latest/config.html#scope-params) and [https://nf-co.re/docs/usage/configuration#max-resources](https://nf-co.re/docs/usage/configuration#max-resources)

```
params {
  // Config profile name, displayed in output
  config_profile_description = 'TSO500 run on PBSPro HPC'
  
  // maximum job resources
  max_memory = 200.GB
  max_cpus = 32
  max_time = 72.h
}
```

#### Configuration profiles

Within configuration files, you can define profiles that are enabled using the `-profile` flag in the nextflow command.  
(More information at [https://www.nextflow.io/docs/latest/config.html#config-profiles](https://www.nextflow.io/docs/latest/config.html#config-profiles))  

Several configuration profiles are [predefined for nf-core workflows](https://nf-co.re/usage/configuration#basic-configuration-profiles) that specialize on usage with container systems, but for some pipelines also include _test_ profiles for running automatic test samples.  
Important are profiles as `singularity`, `docker` or `conda`  for usage of implemented tools in the workflow.

## Setup of Nextflow pipelines for offline usage

More information at [https://nf-co.re/docs/usage/offline](https://nf-co.re/docs/usage/offline)

For offline deployment of nf-core workflows, the following is needed:

1. Workflow code
2. Installed tools, e.g. as singularity containers
3. Downloaded references

For 1. & 2., a toolkit is available to download these automatically for nf.core workflows using the `nf-core download` command.  
Additionally, another folder nf-core-sarek-3.1.2/config with public institution-specific config files for nextflow is downloaded.
For example, downloading sarek workflow using singularity containers:
```bash
nf-core download sarek --revision 3.1.2 --compress none --container singularity
```

For 3., generally used references as the human reference genome are provided thorugh the AWS iGenomes collection.
More information for usage with nf-core at [https://nf-co.re/docs/usage/reference_genomes](https://nf-co.re/docs/usage/reference_genomes).
You can get the download command for specific datasets from [https://ewels.github.io/AWS-iGenomes/](https://ewels.github.io/AWS-iGenomes/).
 
Within your mnextflow parameters, you need to specify the igenomes base directory and the reference genome using the parameters `igenomes_base` (pointing to the iGenomes directory, e.g. "db/references") and `genome` (Name of reference downloaded, e.g. "GATK.GRCh38"). If `igenomes_base` is not defined, it will automatically catch it from the internet source defined in `conf/igenomes.config`.

Further, some workflows require workflow-specific references that need to be separately downloaded, for example the VEP cache in Sarek.