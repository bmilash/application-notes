# pgap

## Installation

The pgap package runs from a singularity container based on
docker://ncbi/pgap:2023-05-17.build6771. The main program, pgap.py
(downloaded via wget) attempts to build a copy of this container but the
resulting container is incomplete, lacking the cwltool program. I had to
manually "singularity pull" this container to make a complete and
working .sif file, named pgap_utils.sif.

I tried installing cwltool outside the container, but that doesn't work
- it needs to be within the container.

I also found the PGAP_INPUT_DIR environment variable needs to be set for
pgap.py to find the various files.

## Running

pgap.py requires either command-line argument -n or -r. 

Some of my test jobs in shared nodes failed with errors from taskset,
and I suspect this is because the job did not have access to all the
CPUs. I've never seen this happen in a job on non-shared partitions.

I found this script ran successfully. The run time was exactly 2 hours
on a 32-core notchpeak node.

```
#!/bin/bash
#SBATCH -A chpc
#SBATCH -p notchpeak
#SBATCH --mail-type=ALL
#SBATCH --mail-user=brett.milash@utah.edu

date '+Starting at %c'
module load pgap

pgap.py --no-self-update \
	-g e_coli_genome.fasta \
	-s "Escherichia coli" \
	-D $(which singularity) \
	-c $SLURM_JOB_CPUS_PER_NODE \
	-r \
	-o ./output

date '+Finished at %c'
```

