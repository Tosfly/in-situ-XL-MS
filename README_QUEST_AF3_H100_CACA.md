# MS_659 Cα–Cα distances with AlphaFold 3 on Quest H100 GPUs

This bundle implements the recommended production strategy without a separate
CPU-only AlphaFold array.

## What the workflow does

1. The helper filters out decoy CSMs, retains intralinks plus heteromeric and
   homomeric interlinks, prefers a shared UniProt accession for ambiguous
   intralinks, and writes one AF3 system per unique protein/protein pair.
2. A **base H100 array** runs one seed with five diffusion samples for every
   system using `af3_full`.
3. For heterodimers and homodimers, a **supplemental H100 array** reuses the
   MSA/template-enriched AF3 data JSON and runs seeds 2 and 3 separately with
   `af3_gpu`. Each supplemental task therefore produces five additional models.
4. The collector reads all seed/sample models, rejects failed or low-confidence
   models, and uses the median accepted Cα–Cα distance for each XL. It does not
   select the model with the shortest XL distance.

The sampling target is:

- monomer/intralink: 1 seed × 5 samples = 5 models;
- heterodimer or homodimer: 3 seeds × 5 samples = 15 models.

## Four H100s and the 48-hour limit

AlphaFold 3 inference uses one GPU per process. The workflow therefore runs four
independent array elements concurrently (`%4`), each requesting one H100. It does
not request four GPUs for one protein system.

Every GPU array element requests `47:30:00`, below Quest's 48-hour General Access
GPU limit. The workflow also limits standard systems to 5,120 modeled residues.
To reduce timeout risk:

- the base array runs only one seed per system;
- each supplemental complex seed is its own array element;
- larger systems are placed first in each array;
- completed seed/sample outputs are detected and skipped on resubmission;
- a shared JAX compilation cache is enabled.

Important: there is no separate CPU-only AF3 job. However, `af3_full` still runs
AF3's genetic/template-search data pipeline, which is CPU-based internally. The
H100 task requests eight CPU cores for that portion. Starting from raw protein
sequences cannot be both reliable and literally CPU-free.

## Bundle files

- `plot_nondecoy_ca_ca_distance_from_alphafold3.py`: prepare and multi-model
  distance collector.
- `02_run_ms659_af3_h100_base_array.slurm`: one-seed full AF3 run per system.
- `03_run_ms659_af3_h100_extra_seed_array.slurm`: one supplemental complex seed
  per H100 task, reusing the enriched data JSON.
- `04_collect_ms659_af3_distances.slurm`: lightweight four-core postprocessing.
- `submit_ms659_af3_h100_workflow.sh`: prepares inputs and submits the dependent
  arrays.
- `quest_af3_helper_environment.yml` and `setup_af3_helper_env_quest.sh`: helper
  environment.
- `00_validate_quest_af3_h100.sh`: live Quest feature/module validation.
- `AF3_V300_COMPATIBILITY_FIXES.patch`: focused diff for the v3.0.0 output and
  GPU-flag corrections.
- `VALIDATION_REPORT.txt` and `SHA256SUMS.txt`: test summary and file hashes.

## One-time setup on Quest

Put the bundle, the merged CSM file, and preferably the exact FASTA used for the
Scout search in a project directory:

```bash
cd /projects/<account>/MS_659_AF3_CaCa
bash setup_af3_helper_env_quest.sh \
  /projects/<account>/MS_659_AF3_CaCa/pythonenvs/xl_caca_af3
```

Edit `submit_ms659_af3_h100_workflow.sh`, or export the equivalent variables:

```bash
QUEST_ACCOUNT="p12345"
AF3_MODEL_DIR="/projects/p12345/alphafold3_model_parameters"
SEQUENCE_FASTA="/projects/p12345/databases/scout_search.fasta"
```

AlphaFold 3 model parameters must be obtained separately and stored in the path
specified by `AF3_MODEL_DIR`.

## Run the required pilot first

The default mode submits a representative 20-system, one-seed pilot:

```bash
RUN_MODE=pilot bash submit_ms659_af3_h100_workflow.sh
```

Monitor it with:

```bash
squeue -u "$USER"
sacct -X -j <base_array_job_id> \
  --format=JobID,State,Elapsed,Timelimit,AllocTRES,MaxRSS,ExitCode
```

Review the largest pilot tasks. If any one-seed base task approaches 47.5 hours,
do not start the complete production run until that system is shortened,
precomputed AF3 data are supplied, or it is excluded. A supplemental seed cannot
run until the corresponding base task has produced its enriched data JSON.

## Run production

After the pilot is satisfactory:

```bash
RUN_MODE=full bash submit_ms659_af3_h100_workflow.sh
```

The production submission reuses completed pilot outputs. It submits:

1. all base systems, with at most four H100 tasks active;
2. seeds 2 and 3 for every complex, again with at most four H100 tasks active;
3. the lightweight collector after the GPU arrays finish.

Set a Scout Score cutoff when needed:

```bash
RUN_MODE=full SCORE_CUTOFF=0.25 \
  bash submit_ms659_af3_h100_workflow.sh
```

Disable automatic collection, while keeping both H100 arrays, with:

```bash
RUN_MODE=full SUBMIT_COLLECTION=0 \
  bash submit_ms659_af3_h100_workflow.sh
```

## Reliability criteria used by the collector

Defaults:

- linked-residue pLDDT ≥ 70 for both residues;
- complex chain-pair ipTM, or global ipTM fallback, ≥ 0.6;
- `has_clash=true` models rejected;
- at least 3 accepted models for monomers;
- at least 5 accepted models for complexes;
- median distance across accepted models;
- ambiguous CSM mappings selected by confidence/stability, not by shortest
  distance;
- duplicate CSM observations of the same XL collapsed by the median.

The collector writes per-model, per-candidate, unresolved, and job-completeness
audits in addition to the panel-E-style PNG/PDF plot.

## Resume behavior

Rerun the same command after a timeout or transient failure. Base and
supplemental tasks count existing `seed-*_sample-*` model files and skip completed
seeds. AF3 output directories from supplemental runs retain the base job-name
prefix, and the collector aggregates all matching directories.

## Oversized systems

Systems above `MAX_AF3_TOTAL_RESIDUES=5120` are not submitted by default. They
remain in the jobs manifest with `job_status=too_large`. Analyze those separately
only after confirming the accession/isoform mapping and developing a suitable
large-system strategy.


## Quest H100 constraint used in this fixed bundle

Your live `sinfo -p gengpu -N -o '%N|%G|%f' | grep -i h100` output lists H100 nodes with features `quest13,sxm`; it does **not** list `rhel8`. Therefore this fixed bundle uses:

```bash
--gres=gpu:h100:1
--constraint=sxm
```

Do not request an `rhel8` constraint unless a future `sinfo` check explicitly shows `rhel8` as a feature on the target H100 nodes. The orchestrator exposes `GPU_CONSTRAINT=sxm` by default. To submit with only the H100 GRES request, run with `GPU_CONSTRAINT=""`.

Before production, run:

```bash
bash 00_validate_quest_af3_h100.sh
```

The base and supplemental H100 scripts now also fail fast if `af3_full` or `af3_gpu` is missing after `module load alphafold/3.0.0`.

Pilot mode uses relaxed accepted-model thresholds (`MIN_ACCEPTED_MODELS_MONOMER=1`, `MIN_ACCEPTED_MODELS_COMPLEX=1`) unless you override them, because the pilot intentionally runs only seed 1. Full production keeps stricter defaults: monomer >=3 accepted models, complex >=5 accepted models.

## AlphaFold 3.0.0 output/flag compatibility

Quest's `alphafold/3.0.0` module follows the AF3 v3.0.0 layout in which each
seed/sample directory contains unprefixed files:

```text
seed-<seed>_sample-<sample>/model.cif
seed-<seed>_sample-<sample>/summary_confidences.json
```

The root-level top-ranked copy remains `<job_name>_model.cif`. The collector and
completion counters in this bundle accept both the v3.0.0 unprefixed layout and
the prefixed per-sample layout used by newer AF3 releases.

AF3 v3.0.0 has no `--gpu_device` command-line flag. The Slurm allocation exposes
one H100 through `CUDA_VISIBLE_DEVICES`, and AF3 v3.0.0 selects the first locally
visible GPU. Therefore the GPU scripts deliberately do not pass `--gpu_device`.

The exact tagged v3.0.0 CLI also does not expose `--num_diffusion_samples`.
It produces five diffusion samples per seed by default. Because this workflow
uses five samples per seed, the GPU scripts do not pass that flag and fail fast
if a manifest requests another sample count. This avoids a second unknown-flag
failure on an unpatched v3.0.0 installation.

The GPU array scripts also omit a hard-coded `#SBATCH --constraint` directive.
The orchestrator alone supplies `--constraint=${GPU_CONSTRAINT}` (default:
`sxm`), so setting `GPU_CONSTRAINT=""` genuinely disables the constraint.
