# Compiling and installing software

Software provided by Supercomputing Wales is managed using the `module` command.
This includes a variety of compilers and common libraries, as well as some
commercial packages.

To see a list of software available on the system, use the `module avail` command.

If there is software you want to use on AccelerateAI and you think other research
groups or projects would also benefit from,
we can install it as a module.
Please contact [Supercomputing Wales support][scw-support] to discuss this.

If it is likely to only be useful to you or your project, then we recommend
installing software into your own home directory.
To install your own software (in the broadest sense&mdash;from a small Python
script to a large simulation suite), then please see the more detailed sections below.

If you encounter difficulty installing software, please contact
[Supercomputing Wales Support][scw-support],
who will be able to assist and advise.

## Python

We recommend using Conda to manage a Python environment, so that you
get access to the exact version of Python and each of your code's
dependencies.

To load Conda and prepare it for use, use the two commands:

```
$ module load anaconda/2024.06
$ source activate
```

Then, for example, to create a new environment called `ai_2024`,
with Python 3.12 and Tensorflow 2.17.0,
run the command:

```
$ CONDA_OVERRIDE_CUDA="12.4" conda create -n ai_2024 -c conda-forge python=3.12 tensorflow-gpu=2.17.0
```

Notice that we request `tensorflow-gpu` rather than `tensorflow`;
this meta-package automatically installs the dependencies
to ensure that the version of Tensorflow
installed will run on the GPU.
The same is true for other libraries;
for example,
you would want `pytorch-gpu`,
not `pytorch`,
to run on the GPU on AccelerateAI.

To active this environment, then run:

```
$ conda activate ai_2024
```

You can install other dependencies into your Conda environment using either `conda`
or `pip` (or indeed any other Python package installer). For example, to install
packages recommended by your code's `requirements.txt` file, use:

```
$ pip install -r requirements.txt
```

### Troubleshooting

Conda environments use large numbers of files, and by default are created on the
`/home` partition, where there is a relatively small quota on this number.
If you see errors about insufficient disk space, or quota exceeded, while creating
a Conda environment or installing packages, then check the `myquota` command to see
if you have exceeded your allocation. If so, then options are:

* remove some files to make space (for example, delete old Conda environments
  if you aren't using them);
* create the environment in your `/scratch` directory, if you will not use it
  long-term and it can be deleted within a few weeks:
  ```
  $ conda create -p /scratch/s.your.user.name/ai_2024_env
  ```
  or,
* contact [Supercomputing Wales Support][scw-support]
  to request an increase in your file
  count quota.


## C, C++, Fortran

A variety of compilers are installed on the system. Useful ones on AccelerateAI are:

* CUDA, including `nvcc`. Various versions are available; see `module avail CUDA`.
  Note that you must use CUDA 11.0 or later.
* The NVIDIA HPC SDK (formerly the PGI compilers). To use these, run
  ```
  $ module use /apps/local/languages/nvidia_hpc_sdk/21.5/modulefiles
  $ module load nvhpc/21.5
  ```
  If your code makes use of OpenACC, then these compilers are recommended.
* The GNU Compiler Collection is also available; see `module avail compiler/gnu`.
  These do not directly support the A100 GPU but can be used together with
  CUDA or NVHPC.

If you need more help getting started with compiling, please get in touch with
[Supercomputing Wales support][scw-support].

## MPI

If you are looking to run workloads across multiple nodes, then it is likely
you will using MPI, and will need CUDA-aware MPI.

Currently the best option in this space is the version of Open MPI included in the
NVIDIA HPC SDK.

More CUDA-aware versions of MPI are currently being prepared for installation.


[scw-support]: mailto:support@supercomputingwales.ac.uk
