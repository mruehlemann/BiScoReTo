# BiScoReTo

BiScoReTo = Bin Scoring and Refinement Tool

BiScoReTo is a fast tool for the scoring and refinement of metagenomic bins from different binning tools.

## Installation

```
conda create -n biscoreto_env
conda activate biscoreto_env

conda install mamba
mamba install -c conda-forge -c bioconda -c r r-base r-optparse r-dplyr r-readr r-this.path hmmer prodigal parallel
```

## Usage
### 1. Single Copy Marker identification (GTDBtk rel 207)

```
biscoreto_folder="/path/to/biscoreto"

### Fast parallel annotation with prodigal
cat sample.contigs.fa | ~/Isilon/software/bin/parallel -j 8 --block 999k --recstart '>' --pipe prodigal -p meta -a tmp_workfolder/sample.{#}.faa -d tmp_workfolder/sample.{#}.ffn -o tmpfile
cat tmp_workfolder/sample.*.faa > $workfolder/samples/${s}/${s}.prodigal.faa
cat tmp_workfolder/sample.*.ffn > $workfolder/samples/${s}/${s}.prodigal.ffn

### annotation of protein sequences using HMMer and GTDBtk r207 marker genes
hmmsearch -o sample.hmm.tigr.out --tblout sample.hmm.tigr.hit.out --noali --notextw --cut_nc --cpu 4 $biscoreto_folder/hmm/tigrfam.hmm sample.prodigal.faa
hmmsearch -o sample.hmm.pfam.out --tblout sample.hmm.pfam.hit.out --noali --notextw --cut_nc --cpu 4 /$biscoreto_folder/hmm/Pfam-A.hmm sample.prodigal.faa

cat sample.hmm.tigr.hit.out | grep -v "^#" | awk '{print $1"\t"$3"\t"$5}' > sample.tigr
cat sample.hmm.pfam.hit.out | grep -v "^#" | awk '{print $1"\t"$4"\t"$5}' > sample.pfam
cat sample.pfam sample.tigr > sample.hmm
```

### 2. Formatting of contig to bin file

individual contig to bin files from binning algorithms
Format: 	`BIN<tab>CONTIG`
E.g.:	`metabat_bin_01<tab>contig_20`

```
awk '{print $1"\t"$2"\tvamb"}'  sample.vamb.contigs_to_bin.tsv > sample.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tconcoct"}'  sample.concoct.contigs_to_bin.tsv >> sample.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tmetabat2"}'  sample.metabat2.contigs_to_bin.tsv >> sample.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tmaxbin2"}'  sample.maxbin2.contigs_to_bin.tsv >> sample.contigs_to_bin.tsv
```

Final contig to bin file for all algorithms
Format:	`BIN<tab>CONTIG<tab>BINNER`
E.g:		`metabat_bin_01<tab>contig_20<tab>metabat`

#### 3. Run bin-refinement

```
Rscript $biscoreto_folder/BiScoReTo.R -i sample.contigs_to_bin.tsv --hmm sample.hmm -o sample
```
