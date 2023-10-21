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

## Instructions to run SemiBin2

### Gnerating data.csv and data_split.csv for training

#### Single-sample (30 sec)

```bash
SemiBin2 generate_sequence_features_single \
    --input-fasta single_sample_binning/single.fasta \
    --input-bam single_sample_binning/single.bam \
    --output single_output
```

#### Co-assembly ( 30 sec)

```bash
SemiBin2 generate_sequence_features_single \
    --input-fasta coassembly_binning/coassembly.fasta \
    --input-bam coassembly_binning/*.bam \
    --output coassembly_output
```

#### Multi-sample (1 min)

```bash
SemiBin2 generate_sequence_features_multi \
    --input-fasta multi_sample_binning/combined.fasta \
    --input-bam multi_sample_binning/*.bam \
    --separator : \
    --output multi_output
```

For multi-sample binning, after this command, SemiBin2 will generate data.csv and data_split.csv for every sample.

For the following commands, SemiBin2 will deal with every sample in the same way with single-sample, coassembly binning and multi-sample binning. So we just use single-sample binning as an example.

### Training

#### Train from one sample (5 min)

```bash
SemiBin2 train_self \
    --data single_output/data.csv \
    --data-split single_output/data_split.csv \
    --output single_output
```

#### Train from two samples (5 min)

```bash
SemiBin2 train_self \
    --train-from-many \
    --data single_output/data.csv single_output/data.csv \
    --data-split single_output/data_split.csv single_output/data_split.csv \
    --output single_output
```

### Binning

#### Binning with model trained earlier (30 sec)

```bash
SemiBin2 bin_short \
    --model single_output/model.h5 \
    --data single_output/data.csv \
    --input-fasta single_sample_binning/single.fasta \
    --output single_output
```

#### Binning with pretrained model (30 sec)

With a pretrained model, you do not need constraints generation and model training.

```bash
SemiBin2 bin_short \
    --environment human_gut \
    --data single_output/data.csv \
    --input-fasta single_sample_binning/single.fasta \
    --output single_output
```

