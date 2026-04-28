---
name: pacsomatic
description: Use this skill when an agent needs to run, validate, troubleshoot, or explain the nf-core/pacsomatic matched tumor-normal pipeline. This skill handles input checks, samplesheet generation, helper-driven Nextflow launch, scheduler submission, and output triage.
argument-hint: "Provide tumor/normal BAMs, patient/sample IDs, outdir, and exactly one reference mode (--fasta or --genome). Optionally set profile/resources, choose --executor (local/lsf/slurm/pbs/sge/none), and use --dry-run or --run."
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

This skill defines how an agent executes nf-core/pacsomatic as a reproducible
workflow on local systems and scheduler-backed environments.

The helper entrypoint is:

- `scripts/run_pacsomatic.py`

The agent MUST use this helper for validation, artifact generation, and launch
logic. Do not bypass it with manually assembled
`nextflow run nf-core/pacsomatic` commands.

## When To Use

Use this skill when the user wants to:

- run matched tumor-normal analysis from BAM files
- generate or fix pacsomatic samplesheet and launch artifacts
- run locally or submit to LSF/Slurm/PBS/SGE
- perform dry-run validation before execution
- troubleshoot launch failures and summarize outputs

Do not use this skill for:

- scientific interpretation beyond basic run sanity checks
- modifying nf-core/pacsomatic pipeline internals unless explicitly requested

## Workflow

1. Validate required user inputs.
  Require tumor BAM, normal BAM, patient ID, tumor sample ID, normal sample
   ID, output directory, and exactly one reference mode (`--fasta` or
   `--genome`).
2. Validate identity and schema constraints.
   Reject IDs with spaces and enforce samplesheet semantics:
   `patient,sample,status,bam,pbi` where tumor=`1`, normal=`0`.
3. Validate required file paths.
  Reject missing local BAM paths. If optional `pbi` is provided, reject missing
   `pbi` files.
4. Resolve runtime settings.
   Respect user-selected profile, executor, scheduler resources, and optional
   runtime setup settings.
5. Call `scripts/run_pacsomatic.py`.
  Use helper options to generate samplesheet CSV, params YAML, and launch
   script.
6. Stop for dry-run mode.
   If `--dry-run` is set without `--run`, stop after validation and artifact
   generation.
7. Execute only in run mode.
   If `--run` is set, execute locally or submit through selected executor.
8. Report run status and next checks.
  Return launcher output, job ID when available, and first triage targets.

## Quick Start

Use the helper script directly.

Dry-run example:

```bash
python scripts/run_pacsomatic.py \
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

Scheduler run example (Slurm):

```bash
python scripts/run_pacsomatic.py \
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

This helper script:

- validates required run inputs
- enforces pacsomatic samplesheet semantics
- writes samplesheet and generated params config
- writes a launch script for Nextflow execution
- supports local and scheduler execution backends
- supports dry-run validation without execution
- captures launcher output and extracts scheduler job IDs when available

## Operational Guidance

- Use `--dry-run` before the first execution in a new environment.
- Use explicit profile selection such as `singularity,sanger`.
- Use `--use-generated-params-file` when helper-generated YAML should be passed
  with `-params-file`.
- Do not invent nf-core parameters.
- Do not use `-c` for pipeline parameters.
- For run mode, verify runtime prerequisites:
  - Java 17+
  - Nextflow
  - profile-dependent tools (`docker`, `singularity|apptainer`, `conda|mamba`)
- Scheduler command requirements in `--run` mode:
  - LSF: `bsub`
  - Slurm: `sbatch`
  - PBS/SGE: `qsub`

## Troubleshooting

- Missing BAM or optional `pbi` files:
  fix input paths and rerun validation.
- Missing runtime tools:
  install/load required tools for chosen profile before `--run`.
- Submission failure:
  inspect launcher stderr/stdout and verify queue/account/resources.
- Pipeline-level failure:
  inspect `.nextflow.log`, then `pipeline_info`, then failed tasks under
  `work/`.
- Outputs unclear:
  check MultiQC and primary result folders under output directory.

## Agent Response Requirements

The agent response should include:

- exact command (or generated script path) used
- confirmation that validation was performed
- whether run type was dry-run or execution
- scheduler job ID when available
- clear next triage step

## References

- Agent workflow: [references/agent-playbook.md](references/agent-playbook.md)
- Config and outputs: [references/config-and-output.md](references/config-and-output.md)
- Extended usage notes: [references/pacsomatic_guide.md](references/pacsomatic_guide.md)
- Helper script: [scripts/run_pacsomatic.py](scripts/run_pacsomatic.py)
