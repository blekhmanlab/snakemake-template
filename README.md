# Snakemake template

This is the Blekhman lab's template for building Snakemake pipelines. It's not an opinionated template, but it should at least provide a shortcut around all the boilerplate stuff.

## Developing your pipeline


## Running your pipeline

```sh
python3 -m venv venv   # or "python" instead of "python3" depending on your system
source venv/bin/activate
pip install --upgrade pip
pip install snakemake

snakemake --use-singularity --jobs 25 --slurm --default-resources slurm_account=blekhman slurm_partition=blekhman
```
