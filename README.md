<h1>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/images/monty_logo.png">
    <img alt="charlesfoster/monty" src="docs/images/monty_logo.png">
  </picture>
</h1>

[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A523.04.0-23aa62.svg)](https://www.nextflow.io/)
[![run with conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)

## Purpose of this repository
This repository is intended to provide a simple overview of the MONTY pipeline, including the motivations behind its development, its goals, and a simplified explanation of how it works. 

While the pipeline will be fully open-source, the code has not yet been publicly released while further bug testing + refinements/optimisations are conducted. The release is intended to be by the end of 2024. Please feel free to check in on the following repository in which the pipeline will be available upon release: https://github.com/charlesfoster/monty. Once you stop seeing a 404 error, the pipeline can be considered ready to go!

While waiting for the pipeline to be released, please [click here](https://drive.google.com/file/d/1d_ScBqR2VfB8ippZ0sy6iTS3ueppSkdU/view?usp=drive_link) to enjoy footage of the real life Monty (see: ["What's in a name?"](#whats-in-a-name)).

## Related media
Files related to presentation of this pipeline, either via talks, posters, or other, are as follows:

* ABACBS 2024 poster: [click here](docs/files/abacbs_2024_poster.pdf)

## Pipeline introduction

**MONTY** is a bioinformatics pipeline that has been designed to allow the simple execution of a complex workflow(s) for the analysis of metagenomic sequencing data, with a focus on the virome component. Based on an input spreadsheet, the pipeline can take raw input reads (short reads: single-end or paired-end; long reads: under development) then conduct quality control, kmer-based taxonomy assignment, de novo assembly, virus identification, mapping-based taxonomy assignment, and estimation of taxon relative abundance.

Currently the taxon count matrices output by the pipeline are raw counts, but upcoming development is planned to enable the generation of normalised counts. While virus-focused, the pipeline will also work with any metagenomics dataset, but the databases used within the pipeline (both default and otherwise) will need to be adjusted.

## What's in a name?
Bioinformaticians have a long history of adopting interesting/funky names for their tools to stand out from the crowd and aid in searchability. So, instead of the original generic pipeline name of "viromics", we decided to name the pipeline after one of [our lab's](https://linktr.ee/wyoaustralia) unofficial mascots: my dog Monty. After that, it was just a matter of forcing a [backronym](https://en.wikipedia.org/wiki/Backronym) onto the pipeline, and, hence, the "Metagenomic Analysis of Existing and Novel Threats in Virology" name was birthed.

## Why develop _another_ new metagenomics pipeline?
As the field of metagenomics has grown, so too has the breadth of new bioinformatics pipelines to help analyse metagenomic data. Within our lab group we wanted the ability to control in fine detail how we analyse our data to avoid relying on external pipelines that might not _quite_ do what we want, or might even cease to be supported. Additionally, while the number of metagenomics pipelines that consider viruses is growing, in _some cases_ (certainly not all) the inclusion of virus-focused analyses seems like an afterthought, with the primary focus being the characterisation of bacterial taxa.

## What goals have driven development?
As with other fields in science, conducting metagenomic analyses can seem like a bit of a black box. A given workflow might provide you results, but how reliable are those results? There are many steps that can introduce biases into results, such as the choice of tools and databases. Accordingly, we sought to develop a virus-focused workflow that is transparent, reliable, repeatable, freely available, open-source, highly scalable, and easy to run.

While several excellent workflow languages exist, a natural underlying choice was to implement the workflow using the Nextflow domain-specific language given its [strong growth](https://seqera.io/news/seqera_5th_anniversary_news_release/) of Nextflow, including a [surging uptake](https://seqera.io/blog/the-state-of-the-workflow-2023-community-survey-results/) in the bioinformatics community. We also chose to follow the '[nf-core framework](https://www.nature.com/articles/s41587-020-0439-x)' for Nextflow workflow development, given its adherence to best practices and provision of automated pipeline testing, deployment and synchronization. We do not intend at this stage to propose MONTY as being an official nf-core pipeline given some overlaps with existing nf-core workflows (see: [Credits](#credits)), but continued development will mirror nf-core development procedures.

By developing the pipeline in Nextflow, MONTY will be able to be executed on high-performance computing infrastructures (including cloud-based services like AWS, Google Cloud Batch etc.), as well as having native support for container technologies such as Docker and Singularity.

## Pipeline steps

<picture>
   <source media="(prefers-color-scheme: dark)" srcset="docs/images/simplified_dag.png">
   <img alt="charlesfoster/monty" src="docs/images/simplified_dag.png">
</picture>

### Quality Control
1. Raw read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. Read deduplication (optional) ([`BBtools Clumpify`](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/bb-tools-user-guide/clumpify-guide/))
3. Read filtering/trimming (optional)
   * Short reads: ([`fastp`](https://github.com/OpenGene/fastp))
4. Assessment of virome enrichment (optional) ([`ViromeQC`](https://github.com/SegataLab/viromeqc))
5. Removal of contaminant reads from the host and/or PhiX spike-in (optional) ([`bowtie2`](https://github.com/BenLangmead/bowtie2) or [`hostile`](https://github.com/bede/hostile))
6. Clean read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))

### Kmer-based Taxonomy Assignment
1. Assignment of taxonomy to reads using [`kraken2`](https://github.com/DerrickWood/kraken2) and/or [`centrifuge`](https://github.com/DaehwanKimLab/centrifuge)
    * Re-estimation of `kraken2` counts using [`bracken`](https://github.com/jenniferlu717/Bracken)
2. Visualisation with [`KRONA`](https://github.com/marbl/Krona)

### De Novo Assembly
1. Assembly of reads de novo into contigs/scaffolds using [`MEGAHIT`](https://github.com/voutcn/megahit), [`SPAdes`](https://github.com/ablab/spades) or [`PLASS PENGUIN`](https://github.com/soedinglab/plass)
2. Assessment of assembly quality ([`QUAST`](https://github.com/ablab/quast))

### Virus Identification
1. Identification of virus contigs from de novo assemblies ([`geNomad`](https://github.com/apcamargo/genomad) and/or [`cenotetaker3`](https://github.com/mtisza1/Cenote-Taker3))
2. Assessment of the quality/completeness of identified viruses ([`CheckV`](https://bitbucket.org/berkeleylab/checkv/src/master/))
3. Binning of virus genomes ([`vRhyme`](https://github.com/AnantharamanLab/vRhyme))

### Counts Estimation
1. Assessment of coverage of contigs (either all contigs or just virus contigs) ([`bowtie2`](https://github.com/BenLangmead/bowtie2), [`samtools`](https://github.com/samtools/samtools), [`CoverM`](https://github.com/wwood/CoverM))
2. Taxonomic assignment of reads and contigs using [`diamond`](https://github.com/bbuchfink/diamond) and/or [`mmseqs2`](soedinglab/MMseqs2)
3. Reformatting and cleaning of taxids ([`taxonkit`](https://github.com/shenwei356/taxonkit))
4. Conversion of results into counts matrices aggregated at various user-defined taxonomic levels (custom script)

## Roadmap
* Implementation of a long reads workflow
* Allow users to include some samples with paired-end reads and some samples with single-end reads (currently one 'type' or the other must be used for all samples)
* Improved normalisation of output count matrices
* In-depth benchmarking against existing tools
* Provision of MONTY as an online service via AWS and/or Seqera Platform

## Credits

MONTY was originally written by Dr Charles S.P. Foster.

We thank the maintainers and developers of [nf-core](https://nf-co.re). The development of some pipeline sections and decisions has been inspired by, and overlaps with, several excellent [nf-core](https://nf-co.re) workflows, such as [nf-core/mag](https://github.com/nf-core/mag) and [nf-core/phageannotator](https://github.com/nf-core/phageannotator/tree/dev). By design, these workflows are meant to be run sequentially, e.g. "`nf-core/mag` --> `nf-core/phageannotator` --> `nf-core/differentialabundance` --> ...", lending into the strength of the interoperability of pipelines within the `nf-core` framework. However, in our case we wished to have an end-to-end pipeline with a focus on viruses and implementing additional analytical tools and parameter values (both default values and possible values).

## Suggestions?

If you have any suggestions for development, please feel free to email me: charlesDOTfosterATunswDOTeduDOTau.

## Citations

An extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

This pipeline uses code and infrastructure developed and maintained by the [nf-core](https://nf-co.re) community, reused here under the [MIT license](https://github.com/nf-core/tools/blob/master/LICENSE).

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
