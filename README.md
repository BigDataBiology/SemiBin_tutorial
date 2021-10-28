# SemiBin tutorial data

## Creating an environment with SemiBin installed

```bash
conda create -n semibin_tutorial
conda install -n semibin_tutorial -c conda-forge -c bioconda semibin
```

**Important note**: this is a toy dataset that _should not_ be used to draw any
biological conclusions!

## Instructions to run SemiBin

### Gnerating data.csv and data_split.csv for training

#### Single-sample (30 sec)

```bash
SemiBin generate_data_single -i single_sample_binning/single.fasta -b single_sample_binning/single.bam -o single_output 
```

#### Co-assembly ( 30 sec)

```bash
SemiBin generate_data_single -i coassembly_binning/coassembly.fasta -b coassembly_binning/*.bam -o coassembly_output
```

#### Multi-sample (1 min)

```bash
SemiBin generate_data_multi -i multi_sample_binning/combined.fasta -b multi_sample_binning/*.bam -s : -o multi_output
```

For multi-sample binning, after this command, SemiBin will generate data.csv and data_split.csv for every sample. 

For the following commands, SemiBin will deal with every sample in the same way with single-sample, coassembly binning and multi-sample binning. So we just use single-sample binning as an example.

### Constraints generation (30 min, we have provided the output of this command, you can skip this step)

```bash
SemiBin predict_taxonomy -i single_sample_binning/single.fasta -o single_output -r $HOME/.cache/SemiBin/mmseqs2-GTDB/GTDB  
```

### Training

#### Train from one sample (5 min)

```bash
SemiBin train --data single_output/data.csv --data-split single_output/data_split.csv -c single_sample_binning/cannot.txt --mode single -i single_sample_binning/single.fasta -o single_output
```

#### Train from two samples (5 min)

```bash
SemiBin train --data single_output/data.csv single_output/data.csv --data-split single_output/data_split.csv single_output/data_split.csv -c single_sample_binning/cannot.txt single_sample_binning/cannot.txt --mode several -i single_sample_binning/single.fasta single_sample_binning/single.fasta -o single_output
```

### Binning

#### Binning with trained model before (30 sec)

```bash
SemiBin bin --data single_output/data.csv --recluster --model single_output/model.h5 -i single_sample_binning/single.fasta -o single_output
```

#### Binning with pretrained model (30 sec)

```bash
SemiBin bin --data single_output/data.csv --recluster --environment human_gut -i single_sample_binning/single.fasta -o single_output
```

