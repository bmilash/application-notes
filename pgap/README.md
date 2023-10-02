# pgap

## Running

pgap.py requires either command-line argument -n or -r. The argument 
--no-self-update is probably required as well, since updating the 
pgap.py script will not be possible for most users.

This script checks if the container .sif file needs updating each time
it is run. Even if it does not require updating, it seems to want to
run out of a file in your .singularity/cache. I'm not certain how to
prevent that time-consuming step.

Some of my test jobs on shared nodes failed with errors from taskset,
and I suspect this is because the job did not have access to all the
CPUs. I've never seen this happen in a job on non-shared nodes, so
I suggest you not use shared partitions.

I found the following SLURM script ran successfully. The run time was exactly 
2 hours on a 32-core notchpeak node. You will need to update the SLURM
account and potentially the partition:

```
#!/bin/bash
#SBATCH -A chpc
#SBATCH -p notchpeak

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
The genome I tested is E. coli, downloadable here:
https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2/

## Installation

The pgap package runs from a singularity container based on
docker://ncbi/pgap:2023-05-17.build6771. The main program, pgap.py
(downloaded via wget) attempts to build a copy of this container but the
resulting container is incomplete, lacking the cwltool program. I had to
manually "singularity pull" this container to make a complete and
working .sif file, named pgap_utils.sif.

I tried installing cwltool outside the container, but that doesn't work
as it needs to be within the container.

I also found the PGAP_INPUT_DIR environment variable needs to be set for
pgap.py to find the various files, so this is set by the module file.
