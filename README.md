# Snakemake template

This is the Blekhman lab's template for building Snakemake pipelines. It's not an opinionated template, but it should at least provide a shortcut around all the boilerplate stuff.

## Setup

The setup is identical whether you're testing locally on a Mac or running it for real on a cluster.
```sh
# Grab your pipeline code via git or FTP or copy/paste
git clone https://github.com/your_username/your_repo.git
# Go into the project directory (where this README would be)
cd your_repo
# Set up a virtual environment in a directory
# called "venv"
python -m venv venv
# activate the virtual environment
source venv/bin/activate
# install snakemake into the environment
pip install snakemake
```

## Testing your pipeline

To do a "dry run" of your snakemake pipeline, you will need to install Snakemake as described above—it should work as expected on your laptop, but if you're getting inconsistent results or don't want to get things set up locally, you can do the same on an HPC login node. Once Snakemake is installed, you should be able to run a single command to test it:

```
snakemake -np
```

The Snakemake command-line interface has [lots of options](https://snakemake.readthedocs.io/en/stable/executing/cli.html); `-n` flag signifies a dry run that calculates what Snakemake *would* do, but **does not execute any pipeline components**—it just generates a report, which is why you can do this on your laptop if you want. The `-p` flag prints out the commands Snakemake would have run. That part isn't necessary, but may help troubleshooting.

Testing your pipeline anywhere will require a directory structure that looks like the real deal. Because (ideally) most file paths in your Snakefile and configuration will be relative, you can create fake input files to see how Snakemake will interpret your instructions. For example, if you have a rule to run FastQC for every fastq file in `directory1`, you can create a folder called `directory1` in your snakemake directory and create files called `directory1/abc.fastq` and `directory1/def.fastq` to ensure that when you run the pipeline, Snakemake will submit jobs for samples "abc" and "def". If you're doing the testing on the cluster, it may get annoying to make tweaks to your files as you develop them, but you can use the real file paths (and real files) for testing, which may be helpful.

## Running your pipeline

The first thing you'll need to do is make sure all your pipeline's dependencies are installed at the location you'll be executing the pipeline

Once your pipeline is ready, there are several options for executing it. The key concept here is that **Snakemake has a supervisor process that coordinates all the work that's happening**: Each execution of each rule will get its own Slurm job, so "FastQC for sample 1" will be submitted separately from "FastQC for sample 2," and both will run with different resources than those allocated for a third job, "Assemble MAGs from these samples." Those jobs are submitted by a snakemake process that runs continuously until the whole pipeline is complete—this process is the one that determines which jobs need to run, submits them to the scheduler, and periodically checks on their status.

This is the process you need to start, and submitting that job can happen in several ways—the most straightforward is to start an interactive job and, once you're connected to your session, go to your pipeline directory, activate your virtual environment, and run a command like this:

```sh
snakemake --jobs 25 --slurm --default-resources slurm_account=blekhman slurm_partition=blekhman
```

(You should refer to the Snakemake docs for all the flags you may want to attach (e.g. `--use-singularity` if any of your rules are containerized), but regardless of which method you use to start the pipeline, the command will likely look the same.)

Running this command from the terminal can be helpful because you can watch the output accumulate as you go, which can speed up troubleshooting by letting you re-run snakemake after making edits without requiring a whole new job to be submitted. The downside is that if you disconnect from the session, or your interactive job just runs out of time, then snakemake is killed and your pipeline won't progress further. You can restart the pipeline where you left off by re-running the command, but worrying about your connection and randomly restarting things can make things messy.

Another option is to put this snakemake command into a batch script and submit it like any other job: An example is included here as `run_snakemake.slurm`, and it can be used to run the snakemake supervisor process without being personally connected to the node. You can still keep an eye on logs by running `tail` and looking at the output in the Slurm logs.
