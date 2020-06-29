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

`cat nanopore_reads.fq | \
NanoFilt -q 10 -l 150 --headcrop 75 --tailcrop 75 > \
nanopore_reads_nanofilt.fq`

2) 
