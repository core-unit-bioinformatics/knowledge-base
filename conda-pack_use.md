# How to use conda-pack 

Deploying environments on a cluster without access to internet is problematic for many reasons. Due to the fact that direct data download on Hilbert is not possible, and because installing 'bioconductor-' packages from the Hilbert conda mirrors is at best problematic, an option to deploy environments on our HPC is the use of [conda-pack](https://github.com/conda/conda-pack), a tool for archiving conda environments and porting them on other systems. 

The last update for the [conda-pack documentation](https://conda.github.io/conda-pack/) was in 2017, making some commands outdated.

To use conda-pack for Hilbert, use the following steps:

(this code assumes you want to install _conda-pack_ in your "base" environment. If you create a separate environment for it, then activate it before proceeding with the packing step (2) )

1. install conda-pack locally, either with _conda_ or from _PiPI_
```
conda install conda-pack
conda install -c conda-forge conda-pack
```

2. pack the desired local conda environment (in this case _'my_env'_)
- to obtain an archive with the same name as the name of the environment:
``` 
conda pack -n my_env 
```
   - to pack it in an archive with a different name:
```
conda pack -n my_env -o out_name.tar.gz
```
   - to pack an environment located at location:
```
conda pack -p /explicit/path/to/my_env
e.g. conda pack -p /home/${USER}/miniconda3/bin/conda/my_env
```

3. upload the packed environment to Hilbert, using for example *rsync*
```
rsync -Pavz --info=progress2 user@domain:/full_path_to_directory/ my_env.tar.gz
```

4. Unpack environment into directory *my_env*
```
mkdir -p my_env
tar -xzf my_env.tar.gz -C my_env
```

5. (optional) Activate the environment. This adds **my_env/bin** to your path

(pay attention to the "/" at the end. Otherwise if you have pre-existing conda environment with the same name in "**/home/${USER}/miniconda3/envs/**", it will activate the one in the "envs" location)

(this step becomes mandatory if there is no python installed on the cluster)

```
conda activate my_env/
```

6. Cleanup prefixes from in the active environment.
```
(my_env) $ conda-unpack
```

   - At this point the conda environment looks as if it was directly installed on Hilbert.

7. Deactivate the environment
```
(my_env) $ conda deactivate
```


For the full documentation on CLI use of conda-pack, see [https://conda.github.io/conda-pack/cli.html](https://conda.github.io/conda-pack/cli.html)
