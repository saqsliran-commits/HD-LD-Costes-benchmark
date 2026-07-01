\# Global Codex instructions for liran



\## User environment



\* My Windows username is `liran`.

\* My Codex home directory is `C:\\Users\\liran\\.codex`.

\* My local Codex skills are:



&#x20; \* `C:\\Users\\liran\\.codex\\skills\\server-login-liran`

&#x20; \* `C:\\Users\\liran\\.codex\\skills\\server-sbatch-guard`



\## Default server connection



When the task involves my server, SSH, HPC, SLURM, sbatch, methylseq outputs, server files, or remote bioinformatics work, assume this server by default:



\* Server user: `liran`

\* Host: `10.1.20.11`

\* SSH port: `2277`

\* Known previously working Windows local bind address: `10.2.15.218`

\* SSH pattern:



```powershell

ssh.exe -o BindAddress=<local\_ip> -p 2277 liran@10.1.20.11

```



\* SCP pattern:



```powershell

scp.exe -o BindAddress=<local\_ip> -P 2277 <src> liran@10.1.20.11:<dst>

```



Before connecting, check the current Windows IPv4 address with `ipconfig`. Prefer the active `10.x.x.x` address. If `10.2.15.218` is stale, replace it with the current active local IPv4.



Use OpenSSH from Windows PowerShell.



\## Password and credential rules



\* Never store my server password in AGENTS.md, skills, scripts, logs, project files, shell history, final answers, or git commits.

\* Never print, echo, reveal, or hard-code my password.

\* If password entry is required, let me type it interactively in the terminal.

\* Prefer SSH key authentication or SSH config aliases when possible.

\* If a temporary password helper is absolutely necessary during a current interactive session, it must be created only under `%TEMP%`, used once, deleted immediately, and related environment variables must be cleared afterward.



\## Always use these skills for server work



For server login, SSH, SCP, connection checks, remote file inspection, login-node problems, or methylseq/HPC output inspection, first consult:



```text

C:\\Users\\liran\\.codex\\skills\\server-login-liran\\SKILL.md

```



For SLURM, sbatch, srun, HPC job submission, login-node safety, large bioinformatics files, Python/R analysis, model training, or long-running server tasks, first consult:



```text

C:\\Users\\liran\\.codex\\skills\\server-sbatch-guard\\SKILL.md

```



\## Login-node safety rule



Never run compute-heavy work directly on the login node.



If uncertain whether a command is heavy, treat it as heavy and use SLURM/sbatch.



\## Allowed directly on login node only when lightweight and bounded



These are generally allowed if the path and scope are small:



```text

pwd

hostname

ls

find with shallow depth or tight paths

head

tail

sed -n

wc

du -sh on targeted paths

grep on specific small text/log files

mkdir

chmod

ln -s

conda env list

which python

python -m py\_compile

bash -n

short python -c import checks

sbatch

squeue

sacct

scontrol show job

sstat

scancel

scp

rsync

wget

curl

git clone/fetch/pull

```



When inspecting methylseq or HPC work, prefer small existing text files such as logs, samplesheets, manifests, reports, `.nextflow.log\*`, and `job.\*.out`.



\## Forbidden directly on login node



Do not directly run heavy or long-running commands over SSH, including:



```text

python script.py

Rscript script.R

julia script.jl

matlab

notebooks

large FASTQ/BAM/CRAM processing

alignment

methylation extraction

samtools/bedtools over large files

large sort

large compression/decompression

large recursive scans

Scanpy/Seurat/h5ad processing

large matrix operations

UMAP/PCA/Harmony

model training

batch plotting over large data

nextflow compute pipelines

nohup/screen/tmux/background jobs used to bypass SLURM

```



If the user asks to run one of these directly on the server, explain that it should go through SLURM and proceed by preparing an sbatch job.



\## Default server workflow



1\. Classify the action as `light check / transfer / download` or `compute`.

2\. For light checks, run only bounded login-node commands.

3\. For compute, write or update a reproducible script and an sbatch script.

4\. Put environment activation inside the sbatch script.

5\. Submit with `sbatch`.

6\. Record job id, log path, working directory, environment, inputs, and expected outputs.

7\. Monitor with `squeue`, `sacct`, `scontrol`, `sstat`, and log tails.

8\. If a job fails, inspect logs and revise the sbatch job. Do not rerun compute directly on the login node.



\## Default sbatch template



```bash

\#!/usr/bin/env bash

\#SBATCH -o results/<task>/logs/slurm-%j.out

\#SBATCH -p cpu

\#SBATCH --qos=low

\#SBATCH -J <job\_name>

\#SBATCH --nodes=1

\#SBATCH --cpus-per-task=<n>

\#SBATCH --mem=<memory>



set -eo pipefail

cd <project\_root>



conda activate <env\_name>

python <script.py>



echo "END $(date '+%F %T')"

```



Adjust partition, memory, CPU, GPU, modules, conda environment, log path, and working directory according to the actual task.



\## Remote command quoting rule



Avoid complex inline SSH commands that contain pipes, heredocs, redirection, `$?`, nested quotes, or complicated shell logic.



For complex remote work:



1\. Create a local script.

2\. Upload it with `scp`.

3\. Run it remotely with `bash` or submit it with `sbatch`.

4\. Delete temporary remote scripts when appropriate.



Use `scp -o BindAddress=<local\_ip>`, not `scp -b`.



Do not use SSH `-B` batch mode when password authentication is needed.



\## Response preference



When helping with server/HPC work, prefer giving:



\* exact commands,

\* safe login-node checks,

\* sbatch scripts for heavy work,

\* log-check commands,

\* and clear explanation of what is safe to run directly versus what must be submitted.



