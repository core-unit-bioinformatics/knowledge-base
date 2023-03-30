# Configuration of nextflow workflow

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

## Configuration for your Infrastructure

The config file consists of _scopes_ that organizes and groups configuration settings (more information at [config-scopes](https://www.nextflow.io/docs/latest/config.html#config-scopes)).  
Two of the most important scopes are _params_ and _process_:

**_process_ - Executors for HPC systems and process configs**

In the _process_ scope, you can define specific parameters for the executors and workflow processes.  
A large variety of executor systems is supported and this config is set to _local_ per default.
(More information at [executor-page](https://www.nextflow.io/docs/latest/executor.html#executor-page))

**_params_ - Parameters of the workflow**

In the _params_ scope, you can define parameters utilized by the workflows, for example the output directory or maximum resources.  
**In general, you should NOT define your parameters in the `run.config` file, but within the `nf-params.json` file!**  
(More information at [scope-params](https://www.nextflow.io/docs/latest/config.html#scope-params))

**Configuration profiles**

Within configuration files, you can define profiles that are enabled using the `-profile` flag in the nextflow command.
You can enable multiple profiles which are prioritzed by the order of specification, e.g. `-profile test,docker`.
More information at [config-profiles](https://www.nextflow.io/docs/latest/config.html#config-profiles).

**Configuration file for HILBERT and local runs**

[Institutional configs are provided by nf-core](https://github.com/nf-core/configs) and can contain additional infrastructure-specific configurations.
Currently, no HHU or UKD config file exists, but may be added in the future.  
Here is a working configuration file defining one profile for execution on [HILBERT](https://www.zim.hhu.de/forschung/high-performance-computing):

```
profiles {
    hilbert {
        params {
            config_profile_description = 'HILBERT'
        }
        process {
            executor = 'pbspro'
            module = 'Singularity/3.7.1'
            queuesize = 100
            clusterOptions = '-A "PROJECT"'
        }
    }
}
```

**process configurations:**

- **executor**: specifies the HPC system for job submissions.
- **module**: The submitted jobs can load environmental modules on the HPC system with the `module load` command. For example, if the singularity profile is enabled, singularity needs to be loaded with every single job.
- **queuesize**: Maximum number of jobs submitted in parallel. Avoids nextflow going rogue.
- **clusterOptions**: Strings that will be attached to the `qsub` command. At HILBERT, we need to specify the project for each submitted job using the `-A "PROJECT"` flag.

> NOTE: by adding the line `cleanup = true` outside of any scope within `run.config`, the work directory with temporary files will be automatically deleted after a successful run.
