# BiScoReTo

BiScoReTo = Bin Scoring and Refinement Tool

BiScoReTo is a fast tool for the scoring and refinement of metagenomic bins from different binning tools. A preprint is available via BioRXiv.

In brief, BiScoReTo uses GTDBtk rel 207 (v2) marker genes to score completeness and contamination of metagenomic bins, to iteratively select the best metagenome-assembled genomes (MAGs) in a dataset. In addition, BiScoReTo can merge overlapping metagenomic bins from multiple binning inputs and add these hybrid bins for scoring and refinement to the set of candidates MAGs.

# Usage

## Run BiScoReTo as Docker Container

The easiest way to run BiScoReTo is via the Docker Container using the metagenomic contigs and output of binning tools as input.

BiScoReTo will do the ORF calling, marker gene annotation and bin refinement.


```
docker run biscoreto:latest -i example.contig_to_bin.tsv -f example.megahit.fa --threads 8

```

## Stand alone

### 1. Single Copy Marker identification (GTDBtk rel 207)

```
biscoreto_folder="/path/to/biscoreto"

### Fast parallel annotation with prodigal
cd $biscoreto_folder/example
mkdir -p tmp_workfolder
cat example.megahit.fasta | ~/Isilon/software/bin/parallel -j 8 --block 999k --recstart '>' --pipe prodigal -p meta -a tmp_workfolder/example.{#}.faa -d tmp_workfolder/example.{#}.ffn -o tmpfile
cat tmp_workfolder/example.*.faa > example.prodigal.faa
cat tmp_workfolder/example.*.ffn > example.prodigal.ffn
rm -r tmp_workfolder tmpfile

### annotation of protein sequences using HMMer and GTDBtk r207 marker genes
hmmsearch -o example.hmm.tigr.out --tblout example.hmm.tigr.hit.out --noali --notextw --cut_nc --cpu 8 $biscoreto_folder/hmm/tigrfam.hmm example.prodigal.faa
hmmsearch -o example.hmm.pfam.out --tblout example.hmm.pfam.hit.out --noali --notextw --cut_nc --cpu 8 $biscoreto_folder/hmm/Pfam-A.hmm example.prodigal.faa

cat example.hmm.tigr.hit.out | grep -v "^#" | awk '{print $1"\t"$3"\t"$5}' > example.tigr
cat example.hmm.pfam.hit.out | grep -v "^#" | awk '{print $1"\t"$4"\t"$5}' > example.pfam
cat example.pfam example.tigr > example.hmm
```

### 2. Formatting of contig to bin file

individual contig to bin files from binning algorithms

Format: 	`BIN<tab>CONTIG`

E.g.:	`metabat_bin_01<tab>contig_20`

```
awk '{print $1"\t"$2"\tvamb"}'  example.vamb.contigs_to_bin.tsv > example.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tconcoct"}'  example.concoct.contigs_to_bin.tsv >> example.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tmetabat2"}'  example.metabat2.contigs_to_bin.tsv >> example.contigs_to_bin.tsv
awk '{print $1"\t"$2"\tmaxbin2"}'  example.maxbin2.contigs_to_bin.tsv >> example.contigs_to_bin.tsv
```

Final contig to bin file for all algorithms

Format:	`BIN<tab>CONTIG<tab>BINNER`

E.g:		`metabat_bin_01<tab>contig_20<tab>metabat`


### 3. BiScoReTo bin scoring and refinement

```
Rscript $biscoreto_folder/BiScoReTo.R -i example.contigs_to_bin.tsv --hmm example.hmm
```

Final contig to bin file for all algorithms

Format:	`BIN<tab>CONTIG<tab>BINNER`

E.g:		`metabat_bin_01<tab>contig_20<tab>metabat`


### Example

```
biscoreto_folder="/path/to/BiScoReTo"
cd $biscoreto_folder/example

Rscript $biscoreto_folder/BiScoReTo.R -i example.contigs_to_bin.tsv --hmm example.hmm

```


# Installation

## Docker versions


## Standalone scripts

```
conda create -n biscoreto_env
conda activate biscoreto_env

conda install mamba
mamba install -c conda-forge -c bioconda -c r r-base r-optparse r-dplyr r-readr r-funr hmmer prodigal parallel

git clone https://github.com/mruehlemann/BiScoReTo
```
