# Pacsomatic Skill

Agent-oriented skill assets for operating the nf-core/pacsomatic matched tumor-normal pipeline.

This repository provides a runnable helper script and agent guidance documents. Start with this README for execution basics. Use [SKILL.md](SKILL.md) for the full agent execution contract.

## Minimal Inputs

Required run inputs:

- tumor BAM path
- normal BAM path
- output directory
- one reference mode: `--fasta` or `--genome`

## Quick Start

Dry-run validation command:

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

Run mode command (local example):

```bash
python scripts/run_pacsomatic.py \
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

## Repository Layout

- [SKILL.md](SKILL.md): agent operating protocol and troubleshooting logic
- [scripts/run_pacsomatic.py](scripts/run_pacsomatic.py): helper entrypoint
- [environment/nextflow-env.yml](environment/nextflow-env.yml): environment template
- [references/agent-playbook.md](references/agent-playbook.md): concise operation playbook
- [references/config-and-output.md](references/config-and-output.md): config/output mapping
- [references/pacsomatic_guide.md](references/pacsomatic_guide.md): extended usage notes

## Notes

- Run dry-run first in new environments.
- Scheduler execution is selected with `--executor` (`lsf`, `slurm`, `pbs`, `sge`).
- Full behavior contract, triage order, and response requirements are defined in [SKILL.md](SKILL.md).
