# Batch Jobs

[1]: #example-batch-scripts
[2]: #common-slurm-options
[3]: #pipelining-with-dependencies

[slurm-quickstart]: ./slurm-quickstart.md
[interactive]: ./interactive.md
[slurm-doc]: https://slurm.schedmd.com/documentation.html
[slurm-man]: https://slurm.schedmd.com/man_index.html
[slurm-sbatch]: https://slurm.schedmd.com/sbatch.html
[slurm-srun]: https://slurm.schedmd.com/srun.html

This page aims to allow the user to submit a job using the Slurm resource 
manager and scheduler which is responsible for allocating resources. **Resource 
intensive applications should always be run via Slurm**.

It is assumed that you are already familiar with Slurm. If not, you can read the
[Slurm quickstart][slurm-quickstart] which cover the basics. You can also refer
to the Slurm [documentation][slurm-doc] or [manual pages][slurm-man] and in 
particular the page about [sbatch][slurm-sbatch].

## Example Batch Scripts

### Shared memory jobs

TODO

### MPI-based jobs

TODO

### Hybrid MPI+OpenMP jobs

TODO

## Common Slurm options

### Basic job specification

| Option        | Description                                              |
| --------------|----------------------------------------------------------|
| `--time`      | Set a limit on the total run time of the job allocation  |
| `--account`   | Charge resources used by this job to specified project   |
| `--partition` | Request a specific partition for the resource allocation |
| `--job-name`  | Specify a name for the job allocation                    |

### Specify tasks distribution

| Option                | Description                                 |
| ----------------------|---------------------------------------------|
| `--nodes`             | Number of nodes to be allocated to the job  |
| `--ntasks`            | Set the maximum number of tasks (MPI ranks) |
| `--ntasks-per-node`   | Set the number of tasks per node            |
| `--ntasks-per-socket` | Set the number of tasks on each node        |
| `--ntasks-per-core`   | Set the maximum number of task on each core |

### Request CPU cores

| Option            | Description                              |
| ------------------|------------------------------------------|
| `--cpus-per-task` | Set the number of cores per tasks        |
| `--cpus-per-gpu`  | Set the number of CPUs per allocated GPU |

### Request GPUs

| Option            | Description                                              |
| ------------------|----------------------------------------------------------|
| `--gpus`          | Set the total number of GPUs to be allocated for the job |
| `--gpus-per-node` | Set the number of GPUs per node                          |
| `--gpus-per-task` | Set the number of GPUs per task                          |


### Request memory
 
| Option            | Description                            |
| ------------------|----------------------------------------|
| `--mem`           | Set the memory per node                |
| `--mem-per-cpu`   | Set the memory per allocated CPU cores |
| `--mem-per-gpu`   | Set the memory per allocated GPU       | 

### Receive email notifications

Email notifications from Slurm can be requested when certain events occur (job
starts, fails, ...).

| Email Type    | Send email when                                            |
| --------------|------------------------------------------------------------|
| `--mail-user` | Used to specify the email that should receive notification |
| `--mail-type` | When to send an email: `BEGIN`, `END`, `FAIL`, `ALL`       |

## Pipelining with dependencies

Job dependencies allow you to defer the start of a job until the specified
dependencies have been satisfied. Dependencies can be defined in a batch script
with the `--dependency` directive or be passed as a command-line argument to
`sbatch`.

```
sbatch --dependency=<type:job_id[:job_id]>
```

The `type` defines the condition that the job with ID `job_id` must fulfil so
that, the job on which it depends can start. For example

```
$ sbatch job1.sh
Submitted batch job 123456

$ sbatch --dependency=afterany:123456 job2.sh
Submitted batch job 123458
```

Will only start execution of `job2.sh` if, `job1.sh` has finished. The available
types and their description are presented in the table below.

| Dependency type               | Description                                           |
| ------------------------------|-------------------------------------------------------|
| `after:jobid[:jobid...]`      | Begin after the specified jobs have started           |
| `afterany:jobid[:jobid...]`   | Begin after the specified jobs have finished          |
| `afternotok:jobid[:jobid...]` | Begin after the specified jobs have failed            |
| `afterok:jobid[:jobid...]`    | Begin after the specified jobs have run to completion |

### Example

The example below demonstrate the submission of jobs with dependencies with a
bash script. It also shows you an example of a helper function that extracts 
the job ID from the output of the `sbatch` command.  

``` bash
#!/bin/bash

submit_job() {
  sub="$(sbatch "$@")"
  
  if [[ "$sub" =~ Submitted\ batch\ job\ ([0-9]+) ]]; then
    echo "${BASH_REMATCH[1]}"
  else
    exit 1
  fi
}

# first job - no dependencies
id1=$(submit_job job1.sh)

# Two jobs that depend on the first job
id2=$(submit_job --dependency=afterany:$id1 job2.sh)
id3=$(submit_job --dependency=afterany:$id1 job3.sh)

# One job that depends on both the second and the third jobs
id4=$(submit_job  --dependency=afterany:$id2:$id3 job4.sh)
```

!!! warning
    The example above is not a Slurm batch script. It should be used where the
    `sbatch`is available. Typically from a login node as the command is not 
    available on the compute node.
