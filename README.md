# Microbiome draft analysis

For this analysis we are going thru recent results of the sequencing tech Aviti, 
here is a [paper that describes the principle for the sequencing technology](https://www.nature.com/articles/s41587-023-01750-7).

When working commands on the development nodes, it is ok to test commands on the prompt
without asking for resources.  However, it is better to start an interactive job session:

```
srun --nodes=1 --ntasks-per-node=1  --cpus-per-task=8 --time=6:00:00 --pty /bin/bash

```

Once the interactive job is running, we can now focus on the task of analyzing the data.  However,
One more thing to get the system ready for use.  It is having conda or mamba active to easily install
packages that are not available. First, we need to load `Miniforge`, which is a free minimal installer 
for conda and Mamba.

```
# Activating Miniforge
module load Miniforge3/24.3.0-0

# Initiate mamba
mamba init

# If you have not created an evironment, then you could create one
mamba create -n QC_Qiime -c conda-forge

# The proceed to activate it
mamba activate QC_Qiime
```
 
Other things that we have to take care are:

## SLURM
Job scheduler used for most HPCC's systems currently
If you know how to use qsub, it will be easy to transition to SLURM.
Today's focus writing SLURM job scripts and we will also practice how to check job status.

````
| SLURM               | TORQUE                   | Function                           |
| ------------------- | ------------------------ | ---------------------------------- |
| sbatch              | qsub                     | submit <job file>                  |
| srun                | qsub -I                  | submit interactive job             |
| squeue              | qstat                    | list all queued jobs               |
| squeue -u -rfeynman | qstat -u rfeynman        | list queued jobs for user rfeynman |
| scancel             | qdel                     | cancel <job#>                      |
| sinfo               | shownodes -l -n;qstat -q | node status;list of queues         |
````

### slurm use:

`sbatch` submit

`srun submit interactive` job

`squeue` list all queued jobs

`squeue -u rfeynman` list queued jobs for user rfeynman

`scancel` cancel <job#>

`sinfo` node status;list of queues

### Runing an interactive session
Running an interactive session will avoid getting warnings or being flag for using resources out of the queue
Now we can start an interactive session:

```                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                n --nodes=1 --ntasks-per-node=1  --cpus-per-task=8 --partition comp06 --time=6:00:00 --pty /bin/bash
##Basic SLURM scripts
## A basic script in slurm looks like:

#!/bin/bash
#SBATCH --job-name=espresso
#SBATCH --output=zzz.slurm
#SBATCH --nodes=4
#SBATCH --tasks-per-node=32
$SBATCH --time=00:00:10 
#SBATCH --partition comp06
module purge

module load intel/14.0.3 mkl/14.0.3 fftw/3.3.6 impi/5.1.2
cd $SLURM_SUBMIT_DIR
cp *.in *UPF /scratch/$SLURM_JOB_ID
cd /scratch/$SLURM_JOB_ID
mpirun -ppn 16 -hostfile /scratch/${SLURM_JOB_ID}/machinefile_${SLURM_JOB_ID} -genv OMP_NUM_THREADS 4 -genv MKL_NUM_THREADS 4 /share/apps/espresso/qe-6.1-intel-mkl-impi/bin/pw.x -npools 1 <ausurf.in
mv ausurf.log *mix* *wfc* *igk* $SLURM_SUBMIT_DIR/
```
Link cheat sheet: https://www.chpc.utah.edu/presentations/SlurmCheatsheet.pdf
```


```
module load FastQC/0.12.1-Java-11
```