# Singularity container for MitoFinder		
Allio, R., Schomaker-Bastos, A., Romiguier, J., Prosdocimi, F., Nabholz, B., & Delsuc, F. (2020) Mol Ecol Resour. 20, 892-905. ([publication link](https://doi.org/10.1111/1755-0998.13160))

<p align="center">
  <img src="/image/logo.png" alt="Drawing" width="250"/>
</p>

**MitoFinder** is a pipeline to **assemble** mitochondrial genomes and **annotate** mitochondrial genes from trimmed 
read sequencing data.
  
**MitoFinder** is also designed to **find** and **annotate** mitochondrial sequences in existing genomic assemblies (generated from Hifi/PacBio/Nanopore/Illumina sequencing data...)  
  
**MitoFinder** is distributed under the [license](https://github.com/RemiAllio/MitoFinder/blob/master/License/LICENSE).  
  
# Requirements

This container is suitable for all systems with [Singularity version >= 3.0](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed.

# Table of content

1. [Installation guide for MitoFinder](#installation-guide-for-mitofinder)
	- [Get and install Singularity](#get-and-install-singularity)
	- [Get MitoFinder's container](#get-mitofinders-container)
2. [How to use MitoFinder's container](#how-to-use-mitofinders-container)
	- [Mitochondrial genome assembly and annotation](#mitochondrial-genome-assembly-and-annotation)
	- [Find and/or annotate a mitochondrial genome](#find-andor-annotate-a-mitochondrial-genome)
	- [Restart](#restart)
	- [Test cases](#test-cases)
3. [Detailed options](#detailed-options)
4. [INPUT FILES](#input-files)
5. [OUTPUT FILES](#output-files)
6. [Particular cases](#particular-cases)
7. [UCE annotation](#uce-annotation)
8. [How to cite MitoFinder](#how-to-cite-mitofinder)
9. [How to get reference mitochondrial genomes from ncbi](#how-to-get-reference-mitochondrial-genomes-from-ncbi)
10. [How to submit your annotated mitochondrial genome(s) to GenBank NCBI](#how-to-submit-your-annotated-mitochondrial-genomes-to-ncbi-genbank)

# Installation guide for MitoFinder

## Get and install Singularity

To run the container, you just need to have [Singularity version >= 3.0](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed in your system.

To check if a right version of Singularity is actually installed:  
```shell
singularity version
```

### Installation of Singularity version >= 3.0

System dependencies:

```shell
sudo apt-get update && sudo apt-get install -y \
    build-essential \
    libssl-dev \
    uuid-dev \
    libgpgme11-dev \
    squashfs-tools \
    libseccomp-dev \
    wget \
    pkg-config \
    git
```

Go (v>=1.13):

```shell
export VERSION=1.17 OS=linux ARCH=amd64
cd /tmp
wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz

sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
echo 'export GOPATH=${HOME}/go' >> ~/.bashrc
echo 'export PATH=/usr/local/go/bin:${PATH}:${GOPATH}/bin' >> ~/.bashrc
source ~/.bashrc
```

Get and install singularity:

```shell
export VERSION=3.10.1 && # adjust this as necessary \
wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz && \
tar -xzf singularity-ce-${VERSION}.tar.gz && \
cd singularity-ce-3.10.1 && ./mconfig
cd builddir && make
sudo make install
```

Check singularity version:

```shell
singularity version
```

## Get MitoFinder's container 

The singularity container is available [here](https://cloud.sylabs.io/library/_container/61557b8a7163109769dfaf38).
Since you have [Singularity version >= 3.0](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed, you can clone MitoFinder's container directly using singularity with the following command:

```shell 
singularity pull --arch amd64 library://remiallio/default/mitofinder:v1.4.1 
```

and then run it as follows:

```shell
singularity run mitofinder_v1.4.1.sif -h  
```

or:

```shell
mitofinder_v1.4.1.sif -h  
```

### Add MitoFinder's container to your path

```shell
cd PATH/TO/MITOFINDER/
p=$(pwd)
echo -e "\n#Path to MitoFinder \nexport PATH=\$PATH:$p" >> ~/.bashrc 
source ~/.bashrc  
```
  
**WARNING**: If you previously installed MitoFinder on your system and want to install a new version, you should replace the old MitoFinder PATH by the updated one in your ~/.bashrc file. To do so, you need to edit your ~/.bashrc file, remove the lines that add MitoFinder to the PATH, and close your terminal. Then, you should open a new terminal and re-execute the command lines from above.  
**TIP**: If you are connected to cluster, you can use either ```nano``` or ```vi``` to edit the ~/.bashrc file.

To check if the right version of MitoFinder is actually in your PATH:  
```shell
mitofinder_v1.4.1.sif -v
```
  
# How to use MitoFinder's container

### Assembler
First, you can choose the assembler using the following options:  
-- megahit 				(default: faster)  
-- metaspades			(recommended: a bit slower but more efficient (see associated paper). WARNING: Not compatible with single-end reads)  
-- idba  

### tRNA annotation step
Second, you can choose the tool for the tRNA annotation step of MitoFinder using the -t option:  
-t mitfi	(default, MiTFi: slower but really efficient)  
-t arwen	(ARWEN: faster)  
-t trnascan	(tRNAscan-SE)  

## Mitochondrial genome assembly and annotation 

TIP: use MitoFinder_v1.4 --example to print basic usage examples  

### Trimmed paired-end reads
```shell
mitofinder_v1.4.1.sif -j [seqid] -1 [left_reads.fastq.gz] -2 [right_reads.fastq.gz] -r [genbank_reference.gb] -o [genetic_code] -p [threads] -m [memory]   
```

### Trimmed single-end reads
```shell
mitofinder_v1.4.1.sif -j [seqid] -s [SE_reads.fastq.gz] -r [genbank_reference.gb] -o [genetic_code] -p [threads] -m [memory]
```

## Find and/or annotate a mitochondrial genome  

MitoFinder can also be run directly on a previously computed assembly (one or several contig.s in fasta format)
```shell
mitofinder_v1.4.1.sif -j [seqid] -a [assembly.fasta] -r [genbank_reference.gb] -o [genetic_code] -p [threads] -m [memory]
```

## Restart
Use the same command line.  
**WARNING**: If you want to compute the assembly again (for example because it failed) you have to remove the assembly results' directory (--override option). If not, MitoFinder will skip the assembly step.  

### Mitochondrial reference(s)  
Depending on the proximity of your reference, you can play with the following parameters : nWalk; --blast-eval; --blast-identity-nucl; --blast-identity-prot; --blast-size 

## Test cases  

If you want to run a test with assembly and annotation steps:    
```shell
cd PATH/TO/MITOFINDER_CONTAINER/test_case/
../mitofinder_v1.4.1.sif -j Aphaenogaster_megommata_SRR1303315 -1 Aphaenogaster_megommata_SRR1303315_R1_cleaned.fastq.gz -2 Aphaenogaster_megommata_SRR1303315_R2_cleaned.fastq.gz -r reference.gb -o 5 -p 5 -m 10   
```  
  
If you want to run a test with only the annotation step:  
```shell
cd PATH/TO/MITOFINDER_CONTAINER/test_case/
../mitofinder_v1.4.1.sif -j Hospitalitermes_medioflavus_NCBI -a Hospitalitermes_medioflavus_NCBI.fasta -r Hospitalitermes_medioflavus_NCBI.gb -o 5
```  
    
# Detailed options  
  
```shell
usage: mitofinder_v1.4.1.sif [-h] [--megahit] [--idba] [--metaspades] [-t TRNAANNOTATION]
                  [-j PROCESSNAME] [-1 PE1] [-2 PE2] [-s SE] [-c CONFIG]
                  [-a ASSEMBLY] [-m MEM] [-l SHORTESTCONTIG]
                  [-p PROCESSORSTOUSE] [-r REFSEQFILE] [-e BLASTEVAL]
                  [-n NWALK] [--override] [--adjust-direction] [--ignore]
                  [--new-genes] [--allow-intron] [--numt]
                  [--intron-size INTRONSIZE] [--max-contig MAXCONTIG]
                  [--cds-merge] [--out-gb] [--min-contig-size MINCONTIGSIZE]
                  [--max-contig-size MAXCONTIGSIZE] [--rename-contig RENAME]
                  [--blast-identity-nucl BLASTIDENTITYNUCL]
                  [--blast-identity-prot BLASTIDENTITYPROT]
                  [--blast-size ALIGNCUTOFF] [--circular-size CIRCULARSIZE]
                  [--circular-offset CIRCULAROFFSET] [-o ORGANISMTYPE] [-v]
                  [--example] [--citation]

Mitofinder is a pipeline to assemble and annotate mitochondrial DNA from
trimmed sequencing reads.

optional arguments:
  -h, --help            show this help message and exit
  --megahit             Use Megahit for assembly. (Default)
  --idba                Use IDBA-UD for assembly.
  --metaspades          Use MetaSPAdes for assembly.
  -t TRNAANNOTATION, --tRNA-annotation TRNAANNOTATION
                        "arwen"/"mitfi"/"trnascan" tRNA annotater to use.
                        Default = mitfi
  -j PROCESSNAME, --seqid PROCESSNAME
                        Sequence ID to be used throughout the process
  -1 PE1, --Paired-end1 PE1
                        File with forward paired-end reads
  -2 PE2, --Paired-end2 PE2
                        File with reverse paired-end reads
  -s SE, --Single-end SE
                        File with single-end reads
  -c CONFIG, --config CONFIG
                        Use this option to specify another Mitofinder.config
                        file.
  -a ASSEMBLY, --assembly ASSEMBLY
                        File with your own assembly
  -m MEM, --max-memory MEM
                        max memory to use in Go (MEGAHIT or MetaSPAdes)
  -l SHORTESTCONTIG, --length SHORTESTCONTIG
                        Shortest contig length to be used (MEGAHIT). Default =
                        100
  -p PROCESSORSTOUSE, --processors PROCESSORSTOUSE
                        Number of threads Mitofinder will use at most.
  -r REFSEQFILE, --refseq REFSEQFILE
                        Reference mitochondrial genome in GenBank format
                        (.gb).
  -e BLASTEVAL, --blast-eval BLASTEVAL
                        e-value of blast program used for contig
                        identification and annotation. Default = 0.00001
  -n NWALK, --nwalk NWALK
                        Maximum number of codon steps to be tested on each
                        size of the gene to find the start and stop codon
                        during the annotation step. Default = 5 (30 bases)
  --override            This option forces MitoFinder to override the previous
                        output directory for the selected assembler.
  --adjust-direction    This option tells MitoFinder to adjust the direction
                        of selected contig(s) (given the reference).
  --ignore              This option tells MitoFinder to ignore the non-
                        standart mitochondrial genes.
  --new-genes           This option tells MitoFinder to try to annotate the
                        non-standard animal mitochondrial genes (e.g. rps3 in
                        fungi). If several references are used, make sure the
                        non-standard genes have the same names in the several
                        references
  --allow-intron        This option tells MitoFinder to search for genes with
                        introns. Recommendation : Use it on mitochondrial
                        contigs previously found with MitoFinder without this
                        option.
  --numt                This option tells MitoFinder to search for both
                        mitochondrial genes and NUMTs. Recommendation : Use it
                        on nuclear contigs previously found with MitoFinder
                        without this option.
  --intron-size INTRONSIZE
                        Size of intron allowed. Default = 5000 bp
  --max-contig MAXCONTIG
                        Maximum number of contigs matching to the reference to
                        keep. Default = 0 (unlimited)
  --cds-merge           This option tells MitoFinder to not merge the exons in
                        the NT and AA fasta files.
  --out-gb              Do not create annotation output file in GenBank
                        format.
  --min-contig-size MINCONTIGSIZE
                        Minimum size of a contig to be considered. Default =
                        1000
  --max-contig-size MAXCONTIGSIZE
                        Maximum size of a contig to be considered. Default =
                        25000
  --rename-contig RENAME
                        "yes/no" If "yes", the contigs matching the
                        reference(s) are renamed. Default is "yes" for de novo
                        assembly and "no" for existing assembly (-a option)
  --blast-identity-nucl BLASTIDENTITYNUCL
                        Nucleotide identity percentage for a hit to be
                        retained. Default = 50
  --blast-identity-prot BLASTIDENTITYPROT
                        Amino acid identity percentage for a hit to be
                        retained. Default = 40
  --blast-size ALIGNCUTOFF
                        Percentage of overlap in blast best hit to be
                        retained. Default = 30
  --circular-size CIRCULARSIZE
                        Size to consider when checking for circularization.
                        Default = 45
  --circular-offset CIRCULAROFFSET
                        Offset from start and finish to consider when looking
                        for circularization. Default = 200
  -o ORGANISMTYPE, --organism ORGANISMTYPE
                        Organism genetic code following NCBI table (integer):
                        1. The Standard Code 2. The Vertebrate Mitochondrial
                        Code 3. The Yeast Mitochondrial Code 4. The Mold,
                        Protozoan, and Coelenterate Mitochondrial Code and the
                        Mycoplasma/Spiroplasma Code 5. The Invertebrate
                        Mitochondrial Code 6. The Ciliate, Dasycladacean and
                        Hexamita Nuclear Code 9. The Echinoderm and Flatworm
                        Mitochondrial Code 10. The Euplotid Nuclear Code 11.
                        The Bacterial, Archaeal and Plant Plastid Code 12. The
                        Alternative Yeast Nuclear Code 13. The Ascidian
                        Mitochondrial Code 14. The Alternative Flatworm
                        Mitochondrial Code 16. Chlorophycean Mitochondrial
                        Code 21. Trematode Mitochondrial Code 22. Scenedesmus
                        obliquus Mitochondrial Code 23. Thraustochytrium
                        Mitochondrial Code 24. Pterobranchia Mitochondrial
                        Code 25. Candidate Division SR1 and Gracilibacteria
                        Code
  -v, --version         Version 1.4.1
  --example             Print getting started examples
  --citation            How to cite MitoFinder
```

# INPUT FILES

MitoFinder needs several files to run depending on the method you have choosen (see above):  
- [x] **Reference_file.gb**				containing at least one mitochondrial genome of reference extracted from [NCBI](https://www.ncbi.nlm.nih.gov/)
- [ ] **left_reads.fastq.gz**				containing the left reads of paired-end sequencing  
- [ ] **right_reads.fastq.gz**				containing the right reads of paired-end sequencing  
- [ ] **SE_reads.fastq.gz** 				containing the reads of single-end sequencing  
- [ ] **assembly.fasta**				containing the assembly on which MitoFinder have to find and annotate mitochondrial contig.s   

# OUTPUT FILES
 
### Results' folder  
MitoFinder returns several files for each mitochondrial contig found:  
- [x] **[Seq_ID]_final_genes_NT.fasta**				containing the nucleotides sequences of the final genes selected from all contigs found by MitoFinder   
- [x] **[Seq_ID]_final_genes_AA.fasta**				containing the amino acids sequences of the final genes selected from all contigs found by MitoFinder   
- [x] **[Seq_ID]_mtDNA_contig.fasta**				containing a mitochondrial contig  
- [x] **[Seq_ID]_mtDNA_contig.gff**				containing the final annotation for a given contig (GFF3 format) 
- [x] **[Seq_ID]_mtDNA_contig.tbl**				containing the final annotation for a given contig (Genbank submission format)
- [x] **[Seq_ID]_mtDNA_contig.gb** 				containing the final annotation for a given contig (Genbank format for visualization)
- [x] **[Seq_ID]_mtDNA_contig_genes_NT.fasta** 				containing the nucleotide sequences of annotated genes for a given contig    
- [x] **[Seq_ID]_mtDNA_contig_genes_AA.fasta** 				containing the amino acids sequences of annotated genes for a given contig    
- [x] **[Seq_ID]_mtDNA_contig.png** 				schematic representation of the annotation of the mtDNA contig    
- [x] **[Seq_ID]_mtDNA_contig.infos** 				containing the initial contig name, the length of the contig and the GC content   


# Particular cases

/!\ Close reference required /!\  

For the particular cases below, we recommend using MitoFinder in two different steps. First, you can use it to assemble and/or identify mitochondrial-like contigs, then use it in a second step to annotate these particular contigs (option -a) with the corresponding additional options.  
Also, these options are recommended for cases in which a (really) close reference is available.  

## Annotation of mitochondrial genes containing intron(s)

/!\ Close reference required /!\  

In some taxa (e.g. fungi), it's possible to find mitochondrial genes containing intron(s). In these cases, we add the --allow-intron option (combined with --intron-size and --cds-merge).
However, it is important to note that, despite the search for start and stop codons is functional for this option, there is no search for intronic boundaries. The exon annotation is based only on the similarity with the reference. That's why a close reference is necessary and even with a good reference, we recommend to double check the exon annotation.  


## Annotation of NUMTs

/!\ Close reference required /!\  

Once you have identified nuclear contigs that may contain NUMTs, you can use MitoFinder to find the NUMTs using the --numt option. Basically, this option allows MitoFinder to find the same gene several times in a contig. Given that the NUMTs can be full of stop codons, we recommand to limit the number of walks (--nwalk 0) that MitoFinder can do to improve the annotation (looking for start and stop codons).  


# UCE annotation

MitoFinder starts by assembling both mitochondrial and nuclear reads using de novo metagenomic assemblers. It is only in a second step that mitochondrial contigs are identified and extracted.
MitoFinder thus provides UCE contigs that are already assembled and the annotation can be done from the following file:  
- **[Seq_ID]\_link\_[assembler].scafSeq** 	containing all assembled contigs from raw reads. 

To do so, we recommend the use of the PHYLUCE pipeline which is specifically designed to annotate ultraconserved elements (Faircloth  2015; Tutorial: https://phyluce.readthedocs.io/en/latest/tutorial-one.html#finding-uce-loci).  
You can thus use the file **[Seq_ID]\_link\_[assembler].scafSeq** and start the Phyluce pipeline at the **"Finding UCE"** step.  
  
# How to cite MitoFinder  
  
If you use MitoFinder, please cite:  
  
- Allio, R, Schomaker‐Bastos, A, Romiguier, J, Prosdocimi, F, Nabholz, B, Delsuc, F. (2020). **MitoFinder**: Efficient automated large‐scale extraction of mitogenomic data in target enrichment phylogenomics. Mol Ecol Resour. 20, 892-905. https://doi.org/10.1111/1755-0998.13160     

If you use this container, please also cite:

- Kurtzer, G. M., Sochat, V., & Bauer, M. W. (2017). **Singularity**: Scientific containers for mobility of compute. PloS one, 12(5), e0177459.
  
Please also cite the following references depending on the option chosen for the assembly step in MitoFinder:    
  
- Li, D., Luo, R., Liu, C. M., Leung, C. M., Ting, H. F., Sadakane, K., Yamashita, H. & Lam, T. W. (2016). **MEGAHIT v1.0**: a fast and scalable metagenome assembler driven by advanced methodologies and community practices. Methods, 102(6), 3-11.  
- Nurk, S., Meleshko, D., Korobeynikov, A., & Pevzner, P. A. (2017). **metaSPAdes**: a new versatile metagenomic assembler. Genome research, 27(5), 824-834.  
- Peng, Y., Leung, H. C., Yiu, S. M., & Chin, F. Y. (2012). **IDBA-UD**: a de novo assembler for single-cell and metagenomic sequencing data with highly uneven depth. Bioinformatics, 28(11), 1420-1428.  
  
For tRNAs annotation, depending on the option chosen:  
- **MiTFi**: Juhling, F., Putz, J., Bernt, M., Donath, A., Middendorf, M., Florentz, C., & Stadler, P. F. (2012). Improved systematic tRNA gene annotation allows new insights into the evolution of mitochondrial tRNA structures and into the mechanisms of mitochondrial genome rearrangements. Nucleic acids research, 40(7), 2833-2845.  
- Laslett, D., & Canback, B. (2008). **ARWEN**: a program to detect tRNA genes in metazoan mitochondrial nucleotide sequences. Bioinformatics, 24(2), 172-175.  
- Chan, P. P., & Lowe, T. M. (2019). **tRNAscan-SE**: searching for tRNA genes in genomic sequences. In Gene Prediction (pp. 1-14). Humana, New York, NY.  

For UCEs extraction:  
  
- Faircloth, B. C. (2016). **PHYLUCE** is a software package for the analysis of conserved genomic loci. Bioinformatics, 32(5), 786-788.  


# HOW TO GET REFERENCE MITOCHONDRIAL GENOMES FROM NCBI  

## Using the NCBI web interface

1. Go to [NCBI](https://www.ncbi.nlm.nih.gov/)  
2. Select "Nucleotide" in the search bar  
3. Search for mitochondrion genomes:  
- [x] RefSeq (if available)  
- [x] Sequence length from 12000 to 20000  
4. Download complete record in GenBank format  

Depending on the proximity of your reference, you can play with the following parameters : nWalk; --blast-eval; --blast-identity-nucl; --blast-identity-prot; --blast-size 

![](/image/NCBI.png)

## Using entrez-direct utilities

1. Install entrez-direct utilities (instructions [here](https://www.ncbi.nlm.nih.gov/books/NBK179288/), or using `conda -c bioconda entrez-direct`)
2. Use the following in a shell to batch download all mitochondrial genomes associated with Carnivora:
```sh
taxa=Carnivora 
esearch -db nuccore -query "\"mitochondrion\"[All Fields] AND (\"${taxa}\"[Organism]) AND (refseq[filter] AND mitochondrion[filter] AND (\"12000\"[SLEN] : \"20000\"[SLEN]))" | efetch -format gbwithparts > reference.gb
```

# How to submit your annotated mitochondrial genome(s) to NCBI GenBank   

## Submission with BankIt

If you have few mitochondrial genomes to submit, you should be able to do it with [BankIt](https://submit.ncbi.nlm.nih.gov/about/bankit/) through the NCBI submission portal.  

## Submission with tbl2asn

If you want to submit several complete or partial mitogenomes, we designed MitoFinder to strealine the submission process using [tbl2asn](https://www.ncbi.nlm.nih.gov/genbank/tbl2asn2/).  
tbl2asn requires:  
- [x] **Template file**				containing a text ASN.1 Submit-block object (suffix .sbt). [Create submission template](https://submit.ncbi.nlm.nih.gov/genbank/template/submission/).   
- [x] **Nucleotide sequence data**				containing the mitochondrial sequence(s) and associated information (suffix .fsa).  
- [x] **Feature Table**				containing annotation information for the mitochondrial sequence(s).  
- [ ] **Comment file**				containing assembly and annotation method information (assembly.cmt). [Create comment template](https://submit.ncbi.nlm.nih.gov/structcomment/nongenomes/)  
  
### Creating a compatible FASTA file
Because tbl2asn requires the FASTA file to contain information associated with the data, we wrote a script to create a FASTA file containing the mitochondrial contig(s) found by MitoFinder for each species (Seq_ID) with the associated information.
This script and the associated example files can be found in the MitoFinder directory named "NCBI_submission".

#### INPUT
- [x] index_file.csv				A CSV file (comma-delimited table) containing the metadata information.

The headers of the index file are as follows:
Directory path, Seq ID, organism, location, mgcode, SRA, keywords ...

The first two columns are mandatory and the names cannot be changed but you can complete the index file with the different [source modifiers](https://www.ncbi.nlm.nih.gov/Sequin/modifiers.html) of NCBI by adding columns in the index file.  

The directory path correponds to the path where the [Seq_ID]_mtDNA_contig.fasta file, or [Seq_ID]_mtDNA_contig_\*.fasta files if you have several contigs for the same individual, could be found. If left blank, the script will search for the contig in the directory where you run the script from (./).  
 
```shell 
/PATH/TO/MITOFINDER_CONTAINER/NCBI_submission/create_tbl2asn_files.py -i index_file.csv
```

**TIPS**:  
(1) You can copy or link (symbolic links) all your FASTA and TBL contig files in the same directory and run the script from this directory.  
(2) You can leave blanks in the index file if some species do not need a given source modifier.  


#### OUTPUT
- [x] **[Seq_ID].fsa**				new FASTA file containing all mtDNA contigs and the information for a given [Seq_ID]
- [x] **[Seq_ID].tbl**				new TBL file containing all mtDNA contigs and the information for a given [Seq_ID]


### Command line to run tbl2asn
Once your FASTA and TBL files have been created, you can run [tbl2asn](https://www.ncbi.nlm.nih.gov/genbank/tbl2asn2/) (download here: ftp://ftp.ncbi.nih.gov/toolbox/ncbi_tools/converters/by_program/tbl2asn/) as follows:

```shell 
tbl2asn -t template.sbt -i [Seq_ID].fsa -V v -w assembly.cmt -a s
```

This command will create several files:
- [x] **[Seq_ID].sqn**				Submission file (.sqn) to be sent by e-mail to gb-sub@ncbi.nlm.nih.gov
- [x] **[Seq_ID].val**				Containing ERROR and WARNING values associated with tbl2asn. (ERROR explanations [here](https://www.ncbi.nlm.nih.gov/genbank/genome_validation/))

If you don't have any error and you are happy with the annotation, you can submit your mitochondrial contig(s) by sending the .sqn files to gb-sub@ncbi.nlm.nih.gov


