## Cluster Resources (Expanse)

One might also want to submit bulk jobs while using NMMA. Here, we have
included an example script for job submission (called as jobscript.sh) in SLURM. This job was submitted on SDSC's
Expanse (ACCESS) cluster:

	#!/bin/bash
	#SBATCH --job-name=gw170817_gp_test.job
	#SBATCH --output=logs/gw170817_gp_test.out
	#SBATCH --error=logs/gw170817_gp_test.err
	#SBATCH -p compute
	#SBATCH --nodes=1
	#SBATCH --ntasks-per-node=10
	#SBATCH --mem=249325M
	#SBATCH --time=00:30:00
	#SBATCH --mail-type=ALL
	#SBATCH --mail-user= your_full_email
	#SBATCH -A <<project*>>
	#SBATCH --export=ALL
	module purge
	module load sdsc
	module load cpu/0.15.4 gcc/10.2.0 intel-mpi/2019.8.254
	source /home/username/anaconda3/bin/activate nmma_env
	export PATH=$PATH:/home/username/anaconda3/lib/
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/username/anaconda3/lib/
	mpiexec -n 10 lightcurve-analysis --model Me2017 --outdir outdir --label injection --prior /home/username/nmma/priors/Me2017.prior --tmin 0.1 --tmax 20 --dt 0.5 --error-budget 1 --nlive 512 --Ebv-max 0 --injection /home/username/nmma/injection.json --injection-num 0 --injection-outfile outdir/lc.csv --generation-seed 42 --filters u,g,r,i,z,y,J,H,K --plot --remove-nondetections

To submit the job, run:

	sbatch jobscript.sh

To check the job allotment, you can run:

	squeue -u username


Test runs on other clusters are currently in progress. Further examples on other cluster resources will be subsequently added.

### Generating a slurm script

The `analysis_slurm.py` code in the `tools` directory can be used to generate a slurm script for Expanse. This code takes all arguments accepted by `em/anaysis.py`. Some of these arguments can be defined as wildcards in the generated slurm script so that different values can be provided for unique script runs.

Specifically, the `--model`, `--label`, `--trigger_time`, `--data`, `--tmin`, `--tmax`, and `--dt` arguments can be set to `None` or `nan` (depending on input data type) to instead reference the environment variables `$MODEL`, `$LABEL`, `$TT`, `$DATA`, `$TMIN`, `$TMAX`, and `$DT` in the slurm script.

Additionally, if `--prior` is not specified, it is filled as `priors/$MODEL.prior`. `--outdir` is always filled as `args.outdir/$LABEL` to create a separate directory for each unique label.

Finally, setting the boolean flag `--skip-sampling` in `analysis_slurm.py` will connect it with the environment variable `$SKIP_SAMPLING` in the slurm script. When submitting the script, this variable should contain a string (either `"--skip-sampling"` or `""`) in order for the script to successfully recognize the presence or absence of this flag.

Slurm scripts are submitted asynchronously via the `sbatch` command, and the `--export` argument allows environment variables to be passed to the script. To make use of the variables above to customize a run, use a command like the following:
```
sbatch --export=MODEL=Bu2019lm,LABEL=ZTF21abdpqpq_1696550950.272051,TT=59361.0,DATA=example_files/candidate_data/ZTF21abdpqpq.dat,TMIN=0.0,TMAX=14.0,DT=0.1,SKIP_SAMPLING="--skip-sampling" slurm.sub
```

Above, `slurm.sub` is the name of the file generated by `analysis_slurm.py`. The slurm-specific arguments of this code are described below:

1. `--Ncore` : number of cores to use when running
2. `--job-name` : name of slurm job
3. `--logs-dir-name` : name of directory within nmma to save slurm logs
4. `--cluster-name` : name of cluster (currently only `"Expanse"` is supported)
5. `--partition-type` : name of HPC partition to request for jobs
6. `--nodes` : number of nodes to request
7. `--gpus` : number of GPUs to request
8. `--memory-GB` : memory allocation to request
9. `--time` : maximum walltime for job to run
10. `--mail-type` : whether/how to receive emails about job status (e.g. `"NONE", "FAIL", "ALL"`)
11. `--mail-user` : contact email address
12. `--account-name` : name of HPC account
13. `--python-env-name` : name of python environment name with NMMA installed
14. `--script-name` : name of slurm script
