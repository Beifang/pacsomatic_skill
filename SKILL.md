---
name: pacsomatic
description: Use this skill when an agent needs to run, validate, troubleshoot, or explain the nf-core/pacsomatic matched tumor-normal pipeline on any compute platform. This skill handles input checks, samplesheet generation, Nextflow command assembly, optional scheduler submission, and post-run output triage.
argument-hint: "Provide tumor/normal BAMs, patient/sample IDs, outdir, and one reference mode (--fasta or --genome). Optionally set profile/resources, choose --executor (local/lsf/slurm/pbs/sge/none), and use --dry-run or --run."
user-invocable: true
version: 2.0.0
author: Beifang Niu
ai_assistant: OpenAI Codex
authors:
  - Beifang Niu
  - Haidong
  - Wenchao
license: MIT
compatibility:
  python: ">=3.9"
---

# Pacsomatic Pipeline

## Overview

This skill helps an agent operate nf-core/pacsomatic as a reproducible workflow.
Use it to validate tumor-normal inputs, enforce nf-core samplesheet constraints,
prepare run artifacts, manage runtime setup, execute on local or scheduler backends, and summarize how to inspect
results and failures.

The primary helper is:

- `scripts/run_pacsomatic.py`

All operations MUST go through `scripts/run_pacsomatic.py`.
Do NOT manually construct or execute `nextflow run nf-core/pacsomatic` commands outside this helper.

The helper also supports workflow setup tasks inspired by full pipeline runners:

- optional pipeline repository clone/use (`--repo-path` or `--checkout-dir`)
- conda runtime management (`--conda-env`, `--create-conda-env`, `--conda-env-file`)
- generated params YAML config (`--config-name`, optional `--use-generated-params-file`)

## When To Use

Use this skill when the user wants to:

- run nf-core/pacsomatic from matched tumor and normal BAM files
- prepare or fix a pacsomatic samplesheet and Nextflow launch command
- launch on local workstation, HPC scheduler, or cloud-backed Nextflow profiles
- do a preflight validation before scheduling (`--dry-run`)
- troubleshoot launch failures and locate first logs to inspect

Do not use this skill for:

- downstream biological interpretation beyond basic run sanity checks
- deep pipeline code modifications in nf-core/pacsomatic itself unless explicitly requested

## What The User Needs To Provide

The user does not need to understand nf-core/pacsomatic internals. They only
need to provide:

- a tumor BAM path
- a normal BAM path
- a reference input: either `--fasta` (file path) or `--genome` (genome key)
- an output directory

Optional:

- sample metadata (`patient-id`, `tumor-sample-id`, `normal-sample-id`) if not using defaults from prior context
- executor and resource preferences (`--executor`, queue/account, cpu/memory/walltime)
- optional `pbi` paths
- optional repository source settings (`--repo-path` or `--checkout-dir` + `--repo-url`)
- optional conda runtime settings (`--conda-env`, `--create-conda-env`)

Note:

- no pipeline repository checkout directory is required for this skill workflow

## Recommended Agent Prompt

For local dry-run validation:

```text
Use $pacsomatic.
Run nf-core/pacsomatic in dry-run mode.
My tumor BAM is at /path/to/tumor.bam
My normal BAM is at /path/to/normal.bam
Patient ID is P001
Tumor sample ID is P001_T
Normal sample ID is P001_N
My reference FASTA is /path/to/reference.fa
Write output to /path/to/output
Use executor local and profile test.
```

For HPC scheduler submission:

```text
Use $pacsomatic.
Run nf-core/pacsomatic and submit to scheduler.
My tumor BAM is at /path/to/tumor.bam
My normal BAM is at /path/to/normal.bam
Patient ID is P001
Tumor sample ID is P001_T
Normal sample ID is P001_N
Use genome GRCh38
Write output to /path/to/output
Use executor slurm, queue compute, account my_account, cpus 16, memory 64GB, walltime 48:00.
```

For LSF scheduler submission (cluster-style example):

```text
Use $pacsomatic.
Run nf-core/pacsomatic and submit to LSF.
My tumor BAM is at /path/to/tumor.bam
My normal BAM is at /path/to/normal.bam
Patient ID is P001
Tumor sample ID is P001_T
Normal sample ID is P001_N
Use genome GRCh38
Write output to /path/to/output
Use executor lsf, queue heavy_io, account Somatic_singularity, cpus 16, memory 64GB, walltime 48:00.
```

## Execution Protocol

1. Collect required inputs before any execution.
  Require tumor BAM, normal BAM, patient ID, tumor sample ID, normal sample ID, outdir, and exactly one reference mode (`--fasta` or `--genome`).
2. Validate input identity and schema constraints.
  Reject execution if patient/sample IDs contain spaces or if samplesheet semantics would violate `patient,sample,status,bam,pbi` with status mapping tumor=`1`, normal=`0`.
3. Validate required file paths.
  Reject execution if local BAM files are missing; if `pbi` is provided, reject execution when its path is missing.
4. Resolve runtime and source configuration through helper options.
  If requested, clone/use pipeline repository and resolve or create conda runtime environment.
5. Invoke `scripts/run_pacsomatic.py` to generate artifacts.
  Use the helper to generate the samplesheet, params YAML, and launch script; do not bypass it.
6. Stop after validation when in dry-run mode.
  If `--dry-run` is set and `--run` is not set, end after artifact generation and validation.
7. Execute or submit only in run mode.
  If `--run` is set, execute or submit through the selected `--executor` backend.
8. Capture runtime output and route triage.
  Capture launcher output, report job ID when available, and direct users to output/log locations for next checks.

## Quick Start

Preferred helper command:

```bash
python .github/skills/pacsomatic/scripts/run_pacsomatic.py \
  --tumor-bam /path/to/tumor.bam \
  --normal-bam /path/to/normal.bam \
  --patient-id P001 \
  --tumor-sample-id P001_T \
  --normal-sample-id P001_N \
  --outdir /path/to/output \
  --genome GRCh38 \
  --profile singularity,sanger \
  --dry-run
```

Run locally (or with cluster profile) using generated script:

```bash
python .github/skills/pacsomatic/scripts/run_pacsomatic.py \
  --tumor-bam /path/to/tumor.bam \
  --normal-bam /path/to/normal.bam \
  --patient-id P001 \
  --tumor-sample-id P001_T \
  --normal-sample-id P001_N \
  --outdir /path/to/output \
  --fasta /path/to/reference.fa \
  --profile singularity,sanger \
  --executor local \
  --run
```

Run on scheduler (example: LSF):

```bash
python .github/skills/pacsomatic/scripts/run_pacsomatic.py \
  --tumor-bam /path/to/tumor.bam \
  --normal-bam /path/to/normal.bam \
  --patient-id P001 \
  --tumor-sample-id P001_T \
  --normal-sample-id P001_N \
  --outdir /path/to/output \
  --genome GRCh38 \
  --profile singularity,sanger \
  --executor lsf \
  --queue heavy_io \
  --project Somatic_singularity \
  --cpus 16 \
  --memory-gb 64 \
  --run
```

## Direct Command-Line Use

This skill can also be used directly from the terminal without invoking an
agent workflow wrapper.

Local/direct execution:

```bash
python .github/skills/pacsomatic/scripts/run_pacsomatic.py \
  --tumor-bam /path/to/tumor.bam \
  --normal-bam /path/to/normal.bam \
  --patient-id P001 \
  --tumor-sample-id P001_T \
  --normal-sample-id P001_N \
  --outdir /path/to/output \
  --genome GRCh38 \
  --profile singularity \
  --executor local \
  --run
```

HPC execution backends supported by `--executor`:

- `lsf`
- `slurm`
- `pbs`
- `sge`

For HPC launch, keep the same core command and switch executor/resources, for
example:

```bash
python .github/skills/pacsomatic/scripts/run_pacsomatic.py \
  --tumor-bam /path/to/tumor.bam \
  --normal-bam /path/to/normal.bam \
  --patient-id P001 \
  --tumor-sample-id P001_T \
  --normal-sample-id P001_N \
  --outdir /path/to/output \
  --genome GRCh38 \
  --profile singularity,sanger \
  --executor slurm \
  --queue compute \
  --project my_account \
  --cpus 16 \
  --memory-gb 64 \
  --walltime 48:00 \
  --run
```

This script:

- validates required run inputs and local path assumptions
- enforces nf-core samplesheet semantics for `patient`, `sample`, and `status`
- writes a samplesheet CSV into the output directory
- writes a generated params YAML config into the output directory
- writes a launch script that executes `nextflow run nf-core/pacsomatic`
- can clone/use local pipeline repository when requested
- can auto-detect/create conda runtime and resolve Nextflow from that environment
- supports multiple backends via `--executor` (`local`, `lsf`, `slurm`, `pbs`, `sge`, `none`)
- supports dry-run validation without execution/submission
- executes locally or submits through scheduler commands when `--run` is provided
- reports launcher stdout and extracts a scheduler job ID when present
- fails clearly when required runtime tools or required files are missing

## Parameter Safety Rules

- Do not invent nf-core parameters.
- Do not assume all nf-core pipelines share the same arguments.
- Only use parameters supported by nf-core/pacsomatic and this helper script.
- If parameter support is uncertain, ask the user before constructing the command.

## Operational Guidance

- Prefer pinned pipeline release (`-r`) for reproducibility.
- Prefer explicit profile list, for example `singularity,sanger`.
- For broad parameter sets, use `-params-file` rather than overloading CLI flags.
- Use `--use-generated-params-file` when you want helper-generated YAML to be passed explicitly via `-params-file`.
- Do not use `-c` for pipeline parameters; reserve `-c` for infrastructure configuration.
- If environment setup is new, run `-profile test` first to verify Nextflow/nf-core readiness.
- Dry-run and generate-only modes can run without Nextflow installed; `--run` requires a valid Nextflow executable.
- The helper performs dependency tool checks based on profile/runtime (for example `java`, `docker`, `singularity|apptainer`, `conda|mamba`).
- Nextflow runtime prerequisites checked by helper:
  - `java` (Java 17+)
  - `nextflow` (required for `--run`, optional warning for dry-run/generate-only)
  - container/backend tools by profile (`docker`, `singularity|apptainer`, `conda|mamba`)
- Scheduler launcher checks in `--run` mode:
  - LSF: `bsub`
  - Slurm: `sbatch`
  - PBS/SGE: `qsub`
- For deeper environment checks, use `--check-host-bio-tools` to verify pacsomatic host tools on PATH (`pbmm2`, `samtools`, `mosdepth`, `clair3`, `hiphase`, `deepsomatic`, `severus`, `cnvkit.py`, `vep`, `svpack`, `AnnotSV`, `multiqc`).
- Use `--strict-host-bio-tools` with `--run` to fail fast when host bio-tools are missing.
- In `-profile local` runs, host bioinformatics tool checks are automatically enforced in `--run` mode.
- Match `--executor` to your platform: local workstation (`local`) or scheduler (`lsf`, `slurm`, `pbs`, `sge`).
- If both `--fasta` and `--genome` are provided, this helper prefers `--fasta`.
- Use `--dry-run` first when path or runtime assumptions are uncertain.

## Troubleshooting

- Optional `pbi` path error:
  if `pbi` is set, the file must exist; otherwise remove the `pbi` value or fix the path.
- Missing Nextflow or scheduler tools:
  ensure `nextflow` is on PATH; for scheduler execution ensure launcher command exists (`bsub`, `sbatch`, or `qsub`).
- Missing dependency tools for selected profile/runtime:
  install or load required tools (`java` and profile-dependent tools such as `docker`, `singularity|apptainer`, `conda|mamba`) before `--run`.
- Submission failure:
  inspect returned `stderr`, keep generated script path, and resubmit manually with the printed run command after fixing environment/queue/project settings.
- Pipeline-level failure after submission:
  inspect launcher stdout/stderr logs first, then inspect `pipeline_info` under the run output directory.
- Workflow completed but expected deliverables are unclear:
  inspect `multiqc` and `pipeline_info` first, then review grouped result folders such as `somatic_snv`, `somatic_sv`, `somatic_cnv`, and `methylation`.
- Report or graph output missing:
  confirm whether `-with-report` and `-with-dag` were enabled in the helper command and verify the output paths passed to those flags.

## Failure Triage Priority

1. Verify input file existence first.
  Confirm tumor/normal BAM paths exist and confirm optional `pbi` paths if provided.
2. Verify samplesheet format and schema.
  Confirm columns and status mapping are correct (`patient,sample,status,bam,pbi`; tumor=`1`, normal=`0`).
3. Verify reference mode consistency.
  Confirm `--fasta` or `--genome` is set correctly and not mismatched with run intent.
4. Verify Nextflow and profile configuration.
  Confirm `nextflow` availability, profile validity, and that pipeline parameters are passed via CLI or `-params-file`.
5. Verify scheduler submission layer.
  Confirm launcher command availability (`bsub`, `sbatch`, `qsub`) and queue/account/resource settings.
6. Verify task-level failures under execution workspace.
  Inspect failing tasks under `work/` and correlate with `pipeline_info` and launcher logs.

## Outputs

Expected artifacts include:

- `results/` directory
- `pipeline_info/`
- MultiQC report
- `.nextflow.log`
- `work/` directory

### What to check first

- MultiQC report
- `pipeline_info`
- failing tasks under `work/` when errors occur

## Response Guidelines

The agent response must include:

- the exact command or generated script used
- explicit confirmation that input validation was performed
- whether execution was dry-run or actual run
- job ID when submitted and available
- a clear next step (for example: check logs, `pipeline_info`, or MultiQC)

## References

- Agent workflow: [references/agent-playbook.md](references/agent-playbook.md)
- Config and outputs: [references/config-and-output.md](references/config-and-output.md)
- Extended usage notes: [references/pacsomatic_guide.md](references/pacsomatic_guide.md)
- Helper script: [scripts/run_pacsomatic.py](scripts/run_pacsomatic.py)
