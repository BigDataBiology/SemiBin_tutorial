# Mapping to concatenated samples

If you have created a concatenated FASTA file as in the main tutorial (using `SemiBin2 concatenate_fasta`), you can use your favorite pipeline to map the reads.

For completeness, we demonstrate here how to do it with [NGLess](https://ngless.embl.de/).

The script is provided (`map-to-concatenated.ngl`), so you can just use it.

1. Install `NGLess`:

```bash
conda install -c conda-forge -c bioconda ngless
```

2. Run it 5 times

We need to run it once per sample:

```bash
for i in $(seq 5) ; do
    ngless map-to-concatenated.ngl
done
```

The map-to-concatenated.ngl script should be self-explanatory:

```python
ngless "1.5"
import "parallel" version "1.1"

FAFILE = "multi_output/concatenated.fa.gz"
SAMPLES = ["S1", "S2", "S3", "S4", "S5"]

sample = run_for_all(SAMPLES)

input = fastq("multi_sample_binning" </> sample + ".fq.gz")
input = preprocess(input) using |read|:
    read = substrim(read, min_quality=25)

mapped = map(input, fafile=FAFILE)

write(mapped, ofile="multi_output/mapped_" + sample + ".bam")
```
