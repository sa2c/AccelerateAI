# Submitting batch jobs on AccelerateAI

Once you have [installed the software that your workload needs](installing)
and perhaps [tested interactively that it works on a small problem](running),
you are ready to move to running your workloads as batch jobs. This means defining
what you want them to do in the form of a Unix shell script, and submitting this
script to the Supercomputing Wales job scheduler, Slurm.

This has a number of advantages:

* It gives you access to a full A100 (or more than one!), rather than the slice of one
  that the interactive queue allows.
* The job can start running at any time of day or night, without you needing to be
  sitting in front of your computer waiting.
* You can run many analyses simultaneously.
* The program won't sit idly waiting for your input between steps, again helping to
  make sure that the machine is maximally used.

If you're not already familiar with Unix shell scripting, you may want to work through
the [Software Carpentry introduction to the Unix shell][swc-shell].


## Slurm

[Slurm][slurm] is the batch scheduler used by Supercomputing Wales on SUNBIRD,
and which manages workloads submitted to AccelerateAI.

Details on how to use Slurm on Supercomputing Wales can be found at the [Supercomputing Wales Portal][portal], specifically at the pages on:

* [Running jobs with Slurm][run-slurm]
* [Slurm Job Directives, Variables & Partitions][slurm-directives]
* [Using GPUs][slurm-gpus]

To get started quickly, you can base your submission scripts on the templates below.
To use one,
copy its contents into a file called,
for example,
`submit.sh`,
modify it to run the commands you need,
and submit it with the command:

```shellsession
$ sbatch submit.sh
```

For further information,
see the documentation linked above.

## Queues and time limits

There are two batch queues on AccelerateAI that give access to a full A100 GPU:

* `accel_ai` is the primary queue, intended for production jobs.
  It can be used to run jobs up to two days in length.
* `accel_ai_dev` is a development queue, intended for short test jobs. 
  It can be used for jobs up to two hours in length.

If you need to run workloads that take longer than two days, then you can use
checkpointing to save the state of your computation, allowing it to be
resumed in a subsequent Slurm job. To see an example of how this can be done
in Tensorflow, see [the Tensorflow documentation][tf-checkpointing].

There is one additional partition, `accel_ai_mig`, which holds the partitioned
GPUs used for interactive test workflows. We would recommend using
[the interactive workflow](running) to use this queue, but if you have a 
use case for submitting batch jobs to these nodes, then please
[contact SA2C support][sa2c-support] and we will work with you
to get running.


## Example job scripts

### Single-GPU job

```
#!/bin/bash

#SBATCH --account=scw0000
#SBATCH --partition=accel_ai
#SBATCH --job-name=training
#SBATCH --output=training.out.%j
#SBATCH --error=training.err.%j
#SBATCH --gres=gpu:1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=0-6:00:00

module load anaconda/2021.05
source activate tf_training

echo "Job ${SLURM_JOB_ID} is running on ${HOSTNAME}."
echo "It has access to GPU ${CUDA_VISIBLE_DEVICES}."

python3 training.py
```

Explaining the options in turn:

* `--account` sets the name of the Supercomputing Wales project that this workload
  is associated with. You will need to adjust this!
* `--partition` tells Slurm that we want to use the AccelerateAI nodes.
* `--job-name` sets the name of the job. This is primarily cosmetic, allowing you
  to distinguish your jobs in the queue, but can also be used for managing
  dependencies between different jobs. It's a good idea to set this to something
  specific.
* `--output` and `--error` tell Slurm where to direct the standard and error outputs
  from the job. `%j` is replaced by the job ID, so multiple jobs won't overwrite
  each other's outputs.
* `--gres=gpu:1` tells Slurm that we will be using a single GPU.
* `--ntasks=1` tells Slurm that we will be running a single program. This should
  not be changed unless you know what you're doing!
* `--cpus-per-task=4` tells Slurm to reserve 4 CPU cores. If your code behaves badly
  when multithreading, this can be reduced to 1. Do not set this number higher
  than 4 for a single GPU, as otherwise other users will not be able to use the
  second GPU that shares resources with the one you are allocated. (Currently Slurm
  cannot allocate the "spare" cores that are not directly attached to GPUs; this is
  a problem that we are aware of and looking to solve.)
* `--time` sets the time limit for the job. `0-6:00:00` indicates a six-hour limit;
  the number before the hyphen is a number of days, so `1-12:00:00` would be a day
  and a half. This should not be set to less time than your job takes, as Slurm
  will terminate your job once this time is reached. However, setting a smaller value
  (while still big enough for your job to finish) will help Slurm to find a slot to
  start your job in.
* `module load anaconda/2021.05` loads the Anaconda module, so that we can load the
  Python environment that this job will use. If you are using something other than
  Conda to manage your software environment, then this can be removed, and/or
  replaced with setup code for the tooling you need.
* `source activate` tells Anaconda to activate itself, and then load the specific
  Conda environment specified. You should change this to the name of the environment
  that you have created for your work.
* The two lines starting with `echo` output some information about the job that
  may be useful when debugging.
* `python3 training.py` can be replaced with the command you wish to run.

### Two-GPU job

```
#!/bin/bash

#SBATCH --account=scw0000
#SBATCH --partition=accel_ai
#SBATCH --job-name=training_2gpu
#SBATCH --output=training.out.%j
#SBATCH --error=training.err.%j
#SBATCH --gres=gpu:2
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=0-6:00:00

module load anaconda/2021.05
source activate tf_training

echo "Job ${SLURM_JOB_ID} is running on ${HOSTNAME}."
echo "It has access to GPUs ${CUDA_VISIBLE_DEVICES}."

python3 training.py
```

Most of the options are the same as explained above. The differences:

* `--gres=gpu:2` now requests 2 GPUs instead of 1. Where possible Slurm will
  schedule these to be connected via NVLink.
* `--cpus-per-task=8` now uses all 8 CPU cores that are associated with the
  GPU pair allocated to the job. (It is not currently possible to allocate
  more than 8 CPU cores per GPU pair.)


### Four-GPU job

```
#!/bin/bash

#SBATCH --account=scw0000
#SBATCH --partition=accel_ai
#SBATCH --job-name=training_4gpu
#SBATCH --output=training.out.%j
#SBATCH --error=training.err.%j
#SBATCH --gres=gpu:4
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --time=0-6:00:00

module load anaconda/2021.05
source activate tf_training

echo "Job ${SLURM_JOB_ID} is running on ${HOSTNAME}."
echo "It has access to GPUs ${CUDA_VISIBLE_DEVICES}."

python3 training.py
```

The options are as described above, but now using 4 GPUs and 16 CPU cores.


### Many-GPU job

To use more than four GPUs, please get in touch with the [SA2C RSE team][sa2c-support]
so that we can help you check that your workload scales up on AccelerateAI.

[portal]: https://portal.supercomputing.wales
[run-slurm]: https://portal.supercomputing.wales/index.php/index/submitting-jobs/
[sa2c-support]: mailto:sa2c-support@swansea.ac.uk
[slurm-directives]: https://portal.supercomputing.wales/index.php/index/slurm/migrating-jobs/
[slurm-gpus]: https://portal.supercomputing.wales/index.php/using-gpus/
[slurm]: https://slurm.schedmd.com
[swc-shell]: https://swcarpentry.github.io/shell-novice
[tf-checkpointing]: https://www.tensorflow.org/guide/checkpoint
