# transcriptome_tools
Tools for acoel transcriptome assembly from nanopore reads.

## requirements
1) pull docker images:

`docker pull danledinh/c3poa_rev:latest`

`docker pull danledinh/minimap2:latest`

`docker pull danledinh/rattle:latest`

`docker pull ezlabgva/busco:v4.0.5_cv1`

`docker pull ncbi/blast`

2) install nanofilt

see https://github.com/wdecoster/nanofilt

## usage
1) Trim and filter raw reads:

```
cat nanopore_reads.fq | \
NanoFilt -q 10 -l 150 --headcrop 75 --tailcrop 75 > \
nanopore_reads_nanofilt.fq
```

2) Run rattle pipeline (hard-coded script `rattle_pipeline.sh` provided for reference):

- Rattle
```
docker run --rm \
-v <local_dir>:/<local_dir> \
-u $(id -u):$(id -g) \
danledinh/rattle \
/rattle_module.sh \
<input_reads.fq> \
<output_dir> \
<threads> \
<gene|iso>
```

- Trim consensus contigs (hard-coded trim parameters)
```
python3 trim_fa.py \
<untrimmed_transcriptome.fa> \
<untrimmed_transcriptome.bam> \
<trimmed_transcriptome.fa> \
<threads>
```

- Map reads with minimap2
```
docker run --rm \
-v <local_dir>:/<local_dir> \
-u $(id -u):$(id -g) \
danledinh/minimap2 \
/map.sh \
<input_reads.fq> \
<transcriptome.fa> \
<output_dir> \
<threads>
```

3) Transcriptome analysis with BUSCO and BLAST

- BUSCO conserved gene coverage
```
docker run --rm \
-u $(id -u):$(id -g) \
-v <input_dir>:/busco_wd \
ezlabgva/busco:v4.0.5_cv1 \
busco \
-m transcriptome \
-i <transcriptome.fa> \
-o <output_prefix> \
--auto-lineage-euk
```

- BLAST contig annotation 

(need to download and create BLAST database: https://www.ncbi.nlm.nih.gov/books/NBK537770/)

(need to create `txids.txids` taxomony ID acceptance list)


```
docker run --rm \
-u $(id -u):$(id -g) \
-v <blast_db_dir>:/blast/blastdb:ro \
-v <input_dir>:/blast/queries:ro \
-v <output_dir>:/blast/results:rw \
ncbi/blast \
blastx \
-taxidlist <txids.txids> \
-query <transcriptome.fa> \
-db refseq_protein \
-task blastx-fast \
-num_threads <threads> \
-matrix BLOSUM45 \
-out <output.txt> \
-outfmt "6 evalue qcovs qseqid sseqid staxids sscinames scomnames sblastnames sskingdoms stitle" 
```
