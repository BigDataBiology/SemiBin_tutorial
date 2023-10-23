# SemiBin2 tutorial

**Important note**: this is a toy dataset that _should not_ be used to draw any biological conclusions! It was produced artificially so that it returns a small number of bins in a short amount of time but that is all.

## Creating an environment with SemiBin2 installed

```bash
conda create -n semibin_tutorial
conda install -n semibin_tutorial -c conda-forge -c bioconda semibin
conda activate semibin_tutorial
```

Installing any recent version of the `semibin` package will install both a `SemiBin` (corresponding to the old SemiBin1) and a new `SemiBin2` commands. You can test that the installation works by using

```bash
SemiBin2 check_install
```

## How to run SemiBin2

### Single sample mode

**Step 1.** Generate features (15 sec)

```bash
SemiBin2 generate_sequence_features_single \
    --input-fasta single_sample_binning/single.fasta \
    --input-bam single_sample_binning/single.bam \
    --output single_output
```

**Step 2. (optional)** Train a model (2-10 mins)

```bash
SemiBin2 train_self \
    --data single_output/data.csv \
    --data-split single_output/data_split.csv \
    --output single_output
```

The time taken by this step can vary by quite a lot depending on your hardware.

You can add `--epochs 1` for testing (but the model will not be very good!).

**Step 3 (option 1: use pretrained model).** binning (30 sec)

With a pretrained model, you do not need to train a model.

```bash
SemiBin2 bin_short \
    --environment human_gut \
    --data single_output/data.csv \
    --input-fasta single_sample_binning/single.fasta \
    --output single_output
```

**Step 3 (option 1: use model trained in step 2).** binning (30 secs)

```bash
SemiBin2 bin_short \
    --model single_output/model.h5 \
    --data single_output/data.csv \
    --input-fasta single_sample_binning/single.fasta \
    --output single_output
```

## Variations

Within the basic framework above, we now consider a few different variations, which can lead to better results

### Variation 1: Training a model fom multiple samples

You can train from many samples, using the `--train-from-many` flag. This will (1) take longer, (2) lead to better models.

In this case, we are using the same sample twice, as a demonstration only:

```bash
SemiBin2 train_self \
    --train-from-many \
    --data single_output/data.csv single_output/data.csv \
    --data-split single_output/data_split.csv single_output/data_split.csv \
    --output single_output
```

### Variation 2: Bin a single set of contigs using multiple samples for abundance estimation.

This can be meaningful if you have either co-assembled the samples or performed cross-mappings (_i.e._, mapped reads from multiple samples to your sample of interest).

You still use the `generate_sequence_features_single` subcommand to generate the features.

```bash
SemiBin2 generate_sequence_features_single \
    --input-fasta coassembly_binning/coassembly.fasta \
    --input-bam coassembly_binning/*.bam \
    --output coassembly_output
```

### Variation 3: Multi-sample binning (1 min)

This is a more complex approach, but can obtain very good results.

**Step 1.** Generate a combined FASTA file

```bash
SemiBin2 concatenate_fasta \
    --input-fasta multi_sample_binning/S*.fna \
    --output multi_output
```

**Step 2.** Map all your samples to that combined FASTA file (code not shown)

**Step 3.** Generate features for multi-sample binning

```bash
SemiBin2 generate_sequence_features_multi \
    --input-fasta multi_sample_binning/concatenated.fa.gz \
    --input-bam multi_sample_binning/*.bam \
    --output multi_output
```

SemiBin2 will generate data.csv and data_split.csv for every sample.

**Step 4.** Training

You need to train a model for every sample. For example for sample `S1`:

```bash
SemiBin2 train_self \
    --data multi_output/samples/S1/data.csv \
    --data-split multi_output/samples/S1/data_split.csv \
    --output S1_output
```

This will take a long time. You can add `--epochs 1` for testing (but the model will not be very good!).

You can also run this in a loop with bash

```bash
for sample in S1 S2 S3 S4 S5 ; do
    SemiBin2 train_self \
        --data multi_output/samples/${sample}/data.csv \
        --data-split multi_output/samples/${sample}/data_split.csv \
        --output ${sample}_output
done
```

**Step 5.** Binning

Again, you run it separately for every sample:

```bash
SemiBin2 bin_short \
    --input-fasta multi_sample_binning/S1.fna \
    --model S1_output/model.h5 \
    --data multi_output/samples/S1/data.csv \
    --output output
```

or in a loop

```bash
for sample in S1 S2 S3 S4 S5 ; do
    SemiBin2 bin_short \
        --input-fasta multi_sample_binning/${sample}.fna \
        --model ${sample}_output/model.h5 \
        --data multi_output/samples/${sample}/data.csv \
        --output output
done
```
