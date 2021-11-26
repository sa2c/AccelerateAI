# Running interactively on AccelerateAI

While we recommend using the batch queues for intensive workloads,
we also have capacity for some interactive use of GPUs to test workflows
and validate models before submitting larger jobs to the batch queues.

The A100 GPU is partitioned using [Multi-Instance GPU][mig]; each interactive
GPU has either 5GB or 10GB of memory, and 1/7 or 2/7 of the compute units of
the full GPU respectively.

There are two routes to running interactively on AccelerateAI:

* JupyterHub: This allows you to write and run Jupyter Notebooks that have direct
  access to the GPU.
* Command-line: this allows you to run scripts and other software directly
  at the command prompt. Unlike a batch job, you do not lose access to the GPU
  after each process completes.

Both are time-limited, as all jobs on AccelerateAI are. If your training is at the
point that it is taking longer than the time limit, then we recommend packaging your
workload for the batch queue, whose more powerful GPUs should let it complete more
quickly, and which will allow you to checkpoint and resume your job more easily.

## JupyterHub

JupyterHub is currently in the process of being installed. Please check back later
for instructions on how to use it.

For the time being, we recommend trying the command-line interface. If you need
urgent access to a Jupyter Notebook interface backed by a GPU, then please get
in touch with [SA2C Support][sa2c-support]


## Command-line

To allocate and start an interactive session on AccelerateAI, run:

```
$ srun --pty --account=scwXXXX --gres=gpu:1 --partition=accel_ai_mig /bin/bash
```

(Replace `scwXXXX` with your Supercomputing Wales project identifier.)

From here, you can load any needed modules and run software as if you were running
directly at a command prompt on the machine.

Once your interactive session is concluded, make sure to end the session with the
`exit` command, so that the GPU is freed to allow others to use it.

[mig]: https://www.nvidia.com/en-us/technologies/multi-instance-gpu/
[sa2c-support]: mailto:sa2c-support@swansea.ac.uk
