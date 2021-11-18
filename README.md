# genomepy

[![bioconda-badge](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat)](http://bioconda.github.io)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/genomepy/badges/downloads.svg)](https://anaconda.org/bioconda/genomepy)
[![PyPI version](https://badge.fury.io/py/genomepy.svg)](https://badge.fury.io/py/genomepy)
[![star this repo](https://githubbadges.com/star.svg?user=vanheeringen-lab&repo=genomepy&style=flat)](https://github.com/vanheeringen-lab/genomepy)

[![Build Status](https://travis-ci.org/vanheeringen-lab/genomepy.svg?branch=master)](https://travis-ci.org/vanheeringen-lab/genomepy)
[![Maintainability](https://api.codeclimate.com/v1/badges/c4476820f1d21a3e0569/maintainability)](https://codeclimate.com/github/vanheeringen-lab/genomepy/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/c4476820f1d21a3e0569/test_coverage)](https://codeclimate.com/github/vanheeringen-lab/genomepy/test_coverage)

[![status](http://joss.theoj.org/papers/df434a15edd00c8c2f4076668575d1cd/status.svg)](http://joss.theoj.org/papers/df434a15edd00c8c2f4076668575d1cd)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1010458.svg)](https://doi.org/10.5281/zenodo.1010458)

Install and use genomes & gene annotations the easy way!

genomepy is designed to provide a _simple_ and _straightforward_ way to download and use genomic data.
This includes (1) searching available data, (2) showing the available metadata,
(3) automatically downloading, preprocessing and matching data and
(4) generating optional aligner indexes.
All with sensible, yet controllable defaults.
Currently, genomepy supports UCSC, Ensembl and NCBI. 

[![asciicast](https://asciinema.org/a/eZttBuf5ly0AnjFVBiEIybbjS.png)](https://asciinema.org/a/eZttBuf5ly0AnjFVBiEIybbjS)

**Pssst, hey there!** Is genomepy not doing what you want? Does it fail? Is it clunky?
Is the documentation unclear? Have any other ideas on how to improve it?
Don't be shy and [let us know](https://github.com/vanheeringen-lab/genomepy/issues)!

## Table of Contents
1.  [Installation](#installation)
2.  [Quick usage](#quick-usage)
3.  [Plugins and indexing](#plugins-and-indexing)
4.  [Configuration](#configuration)
5.  [Usage](#usage)
    * [Command line](#command-line)
    * [Python](#python)
6.  [Known issues](#known-issues)
7.  [Getting help](#getting-help)
8.  [Citation](#citation)
9.  [Contributing](#contributing)
10.  [License](#license)

## Installation

genomepy requires Python 3.6+

You can install genomepy via [bioconda](https://bioconda.github.io/):

```
$ conda install genomepy
``` 

Or via pip:

```
$ pip install genomepy
```

Or via git:

```
$ git clone https://github.com/vanheeringen-lab/genomepy.git
$ cd genomepy
$ conda env update -f environment.yml
$ python setup.py install
```

With Pip installation, you will have to install additional dependencies, and make them available in your PATH.

To read/write bgzipped genomes you will have to install `tabix`.

If you want to use gene annotation features, you will have to install the following utilities:

* `genePredToBed`
* `genePredToGtf`
* `bedToGenePred`
* `gtfToGenePred`
* `gff3ToGenePred`

You can find the binaries [here](http://hgdownload.cse.ucsc.edu/admin/exe/).

## Quick usage

1.  Find your genome: `$ genomepy search zebrafish`

  Console output:
  ```
  name      provider    accession           tax_id     annotation    species        other_info
  GRCz11    Ensembl     GCA_000002035.4     7955       ✓             Danio rerio    2017-08-Ensembl/2018-04
   ^
   Use name for genomepy install
  ```

2.  Install your genome (with annotation): `$ genomepy install --annotation GRCz11 --provider ensembl `

  Default genome directory: `~/.local/share/genomes/`

## Plugins and indexing

By default genomepy generates support files, including a genome index,
chromosome sizes and gap locations (Ns in the sequence).

For some genomes genomepy can download blacklist files (generated by the Kundaje lab). 
This will only work when installing these genomes from UCSC.
Enable this plugin to use it.

```
$ genomepy plugin enable blacklist
```

You can also create indices for some widely using aligners.
Currently, genomepy supports:

* [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
* [bwa](http://bio-bwa.sourceforge.net/) 
* [gmap](http://research-pub.gene.com/gmap/)
* [hisat2](https://ccb.jhu.edu/software/hisat2/index.shtml)
* [minimap2](https://github.com/lh3/minimap2)
* [star](https://github.com/alexdobin/STAR)

Note 1: these programs are not installed by genomepy and need to be
installed separately for the indexing to work.

Note 2: splice-aware indexing is performed by Hisat2 and STAR.
Splice-aware indexing requires the annotation to be downloaded as well. 
You will receive a warning if indexing is performed without annotation for these aligners.

Note 3: STAR can further improve mapping to (novel) splice junctions by indexing again (2-pass mapping mode). 
The second pass is not supported by genomepy.

You can configure the index creation using the `genomepy plugin` command (see below)

## Configuration

To change the default configuration, generate a personal config file:

```
$ genomepy config generate
Created config file /home/simon/.config/genomepy/genomepy.yaml
```

### Genome location

By default genomes will be saved in `~/.local/share/genomes`. 

To set the default genome directory, to `/data/genomes` for instance,
edit `~/.config/genomepy/genomepy.yaml` and change the following line:

```
genomes_dir: ~/.local/share/genomes/
```

to:

```
genomes_dir: /data/genomes
```

The genome directory can still be overwritten via the command-line.

### Compression

Optionally genome FASTA files can be saved using bgzip compression.
This means that the FASTA files will take up less space on disk.
To enable this use the flag `--bgzip` on the command line, or add the following line to your config file:

```
bgzip: True
```

Most tools are able to use bgzip-compressed genome files. 
One notable exception is `bedtools getfasta`.
As an alternative, you can use the `faidx` command-line script from [pyfaidx](https://github.com/mdshw5/pyfaidx)
which comes installed with genomepy.

## Usage

### Command line

All functions come with a short explanation when appended with `--help`.

```
$ genomepy --help
Usage: genomepy [OPTIONS] COMMAND [ARGS]...

Options:
  --version   Show the version and exit.
  -h, --help  Show this message and exit.

Commands:
  annotation  show 1st lines of each annotation
  clean       remove provider data
  config      manage configuration
  genomes     list available genomes
  install     install a genome & run active plugins
  plugin      manage plugins
  providers   list available providers
  search      search for genomes
```

#### Install a genome.

Find the name of your desired genome:

```
$ genomepy search xenopus tropicalis
name                       provider    accession           tax_id    annotation     species               other_info
                                                                      n r e k
Xenopus_tropicalis_v9.1    Ensembl     GCA_000004195.3       8364        ✓          Xenopus tropicalis    2019-04-Ensembl/2019-12
xenTro1                    UCSC        na                    8364     ✗ ✗ ✗ ✗       Xenopus tropicalis    Oct. 2004 (JGI 3.0/xenTro1)
xenTro2                    UCSC        na                    8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Aug. 2005 (JGI 4.1/xenTro2)
xenTro3                    UCSC        GCA_000004195.1       8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Nov. 2009 (JGI 4.2/xenTro3)
xenTro7                    UCSC        GCA_000004195.2       8364     ✓ ✓ ✗ ✗       Xenopus tropicalis    Sep. 2012 (JGI 7.0/xenTro7)
xenTro9                    UCSC        GCA_000004195.3       8364     ✓ ✓ ✓ ✗       Xenopus tropicalis    Jul. 2016 (Xenopus_tropicalis_v9.1/xenTro9)
Xtropicalis_v7             NCBI        GCF_000004195.2       8364        ✓          Xenopus tropicalis    DOE Joint Genome Institute
Xenopus_tropicalis_v9.1    NCBI        GCF_000004195.3       8364        ✓          Xenopus tropicalis    DOE Joint Genome Institute
UCB_Xtro_10.0              NCBI        GCF_000004195.4       8364        ✓          Xenopus tropicalis    University of California, Berkeley
ASM1336827v1               NCBI        GCA_013368275.1       8364        ✗          Xenopus tropicalis    Southern University of Science and Technology
 ^
 Use name for genomepy install
```

You can search by genome name (case-insensitive), taxonomy ID or assembly accession ID.
Additionally, you can limit the search result to one provider with `-p`/`--provider`.

```
$ genomepy search 8364 -p ucsc
name                    provider    accession           tax_id    annotation     species               other_info
                                                                   n r e k
xenTro1                 UCSC        na                    8364     ✗ ✗ ✗ ✗       Xenopus tropicalis    Oct. 2004 (JGI 3.0/xenTro1)
xenTro2                 UCSC        na                    8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Aug. 2005 (JGI 4.1/xenTro2)
xenTro3                 UCSC        GCA_000004195.1       8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Nov. 2009 (JGI 4.2/xenTro3)
xenTro7                 UCSC        GCA_000004195.2       8364     ✓ ✓ ✗ ✗       Xenopus tropicalis    Sep. 2012 (JGI 7.0/xenTro7)
xenTro9                 UCSC        GCA_000004195.3       8364     ✓ ✓ ✓ ✗       Xenopus tropicalis    Jul. 2016 (Xenopus_tropicalis_v9.1/xenTro9)
 ^
 Use name for genomepy install
```

Lets say we want to download the latest *Xenopus tropicalis* genome from UCSC.

If you are interested in the gene annotation as well, you might want to check which gene annotation suits your needs.
Because we're looking at UCSC there are several options for us to choose from.
In the search results, `n r e k ` denotes which UCSC annotations are available. 
These stand for **n**cbiRefSeq, **r**efGene, **e**nsGene and **k**nownGene, respectively.

We can quickly inspect these with the `genomepy annotation` command:

```
$ genomepy annotation xenTro9 -p ucsc
12:04:41 | INFO | UCSC ncbiRefSeq
chr1    genomepy        transcript      133270  152620  .       -       .       gene_id "LOC100490505"; transcript_id "XM_012956089.1";  gene_name "LOC100490505";
chr1    genomepy        exon    133270  134186  .       -       .       gene_id "LOC100490505"; transcript_id "XM_012956089.1"; exon_number "1"; exon_id "XM_012956089.1.1"; gene_name "LOC100490505";
12:04:45 | INFO | UCSC refGene
chr1    genomepy        transcript      193109390       193134311       .       +       .       gene_id "pias2"; transcript_id "NM_001078987";  gene_name "pias2";
chr1    genomepy        exon    193109390       193109458       .       +       .       gene_id "pias2"; transcript_id "NM_001078987"; exon_number "1"; exon_id "NM_001078987.1"; gene_name "pias2";
12:04:49 | INFO | UCSC ensGene
chr1    genomepy        transcript      133270  152620  .       -       .       gene_id "ENSXETG00000030302.2"; transcript_id "ENSXETT00000061673.2";  gene_name "ENSXETG00000030302.2";
chr1    genomepy        exon    133270  134186  .       -       .       gene_id "ENSXETG00000030302.2"; transcript_id "ENSXETT00000061673.2"; exon_number "1"; exon_id "ENSXETT00000061673.2.1"; gene_name "ENSXETG00000030302.2";
```

Here we can see that the `refGene` annotation has actual HGNC gene names, so lets go with this annotation.

Copy the name returned by the search function to install.
For UCSC we can also select the annotation type.

```
$ genomepy install xenTro9 --UCSC-annotation refGene
```

Since we did not specify the provider here, genomepy will use the first provider it can find with `xenTro9`.
Since we learned in `genomepy search` that only UCSC uses this name, it will be UCSC.
We can also specify genomepy to use UCSC by giving it the provider name with `-p`/`--provider`:

```
$ genomepy install xenTro9 -p UCSC
Downloading genome from http://hgdownload.soe.ucsc.edu/goldenPath/xenTro9/bigZips/xenTro9.fa.gz...
Genome download successful, starting post processing...

name: xenTro9
local name: xenTro9
fasta: /data/genomes/xenTro9/xenTro9.fa
```

Next, the genome is downloaded to the directory specified in the config file.
To choose a different directory, use the `-g`/`--genomes_dir` option:

```
$ genomepy install sacCer3 -p UCSC -g /path/to/my/genomes
Downloading genome from http://hgdownload.soe.ucsc.edu/goldenPath/sacCer3/bigZips/chromFa.tar.gz...
Genome download successful, starting post processing...

name: sacCer3
local name: sacCer3
fasta: /path/to/my/genomes/sacCer3/sacCer3.fa
```

You can use a regular expression to filter for matching sequences
(or non-matching sequences by using the `--no-match` option).
For instance, the following command downloads hg38 and saves only the major chromosomes:

```
$ genomepy install hg38 -p UCSC -r 'chr[0-9XY]+$'
downloading from http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz...
done...
name: hg38
local name: hg38
fasta: /data/genomes/hg38/hg38.fa

$ grep ">" /data/genomes/hg38/hg38.fa
>chr1
>chr10
>chr11
>chr12
>chr13
>chr14
>chr15
>chr16
>chr17
>chr18
>chr19
>chr2
>chr20
>chr21
>chr22
>chr3
>chr4
>chr5
>chr6
>chr7
>chr8
>chr9
>chrX
>chrY
```

By default, sequences are soft-masked. Use `-m hard` for hard masking, or `-m none` for no masking.

The chromosome sizes are saved in file called `<genome_name>.fa.sizes`.

You can choose to download gene annotation files with the `--annotation` option. 
These will be saved in (gzipped) BED and GTF format. 

```
$ genomepy  install hg38 -p UCSC --annotation
```

To facilitate the downloading of genomes not supported by either NCBI, UCSC, or Ensembl, genomes 
can also be downloaded directly from an url:

```
$ genomepy install -p url https://research.nhgri.nih.gov/hydra/download/assembly/\Hm105_Dovetail_Assembly_1.0.fa.gz
```
This installs the genome under the filename of the link, but can be changed with the `--localname` option

If you add the `--annotation` flag, genomepy will search the remote directory for an annotation file as well.
Should this fail, you can also add a url to the annotation with `--URL-to-annotation`.

Finally, in the spirit of reproducibility all selected options are stored in a `README.txt`. 
This includes the original name, download location and other genomepy operations (such as regex filtering and time).

#### Manage plugins.

Use `genomepy plugin list` to view the available plugins.

```
$ genomepy plugin list
plugin              enabled
bowtie2             
bwa                 
gmap                
hisat2              
minimap2            
star
blacklist
```

Enable plugins as follows:

```
$ genomepy plugin enable bwa hisat2
Enabled plugins: bwa, hisat2
```

And disable like this:

```
$ genomepy plugin disable bwa
Enabled plugins: hisat2
```

#### Search for a genome.

You can search by genome name (case-insensitive), taxonomy ID or assembly accession ID.
Additionally, you can limit the search result to one provider with `-p`/`--provider`.

```
$ genomepy search xenopus tropicalis
name                       provider    accession           tax_id    annotation     species               other_info
                                                                      n r e k
Xenopus_tropicalis_v9.1    Ensembl     GCA_000004195.3       8364        ✓          Xenopus tropicalis    2019-04-Ensembl/2019-12
xenTro1                    UCSC        na                    8364     ✗ ✗ ✗ ✗       Xenopus tropicalis    Oct. 2004 (JGI 3.0/xenTro1)
xenTro2                    UCSC        na                    8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Aug. 2005 (JGI 4.1/xenTro2)
xenTro3                    UCSC        GCA_000004195.1       8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Nov. 2009 (JGI 4.2/xenTro3)
xenTro7                    UCSC        GCA_000004195.2       8364     ✓ ✓ ✗ ✗       Xenopus tropicalis    Sep. 2012 (JGI 7.0/xenTro7)
xenTro9                    UCSC        GCA_000004195.3       8364     ✓ ✓ ✓ ✗       Xenopus tropicalis    Jul. 2016 (Xenopus_tropicalis_v9.1/xenTro9)
Xtropicalis_v7             NCBI        GCF_000004195.2       8364        ✓          Xenopus tropicalis    DOE Joint Genome Institute
Xenopus_tropicalis_v9.1    NCBI        GCF_000004195.3       8364        ✓          Xenopus tropicalis    DOE Joint Genome Institute
UCB_Xtro_10.0              NCBI        GCF_000004195.4       8364        ✓          Xenopus tropicalis    University of California, Berkeley
ASM1336827v1               NCBI        GCA_013368275.1       8364        ✗          Xenopus tropicalis    Southern University of Science and Technology
 ^
 Use name for genomepy install
```

Only search a specific provider:

```
$ genomepy search tropicalis -p ucsc
name                    provider    accession           tax_id    annotation     species               other_info
                                                                   n r e k
xenTro1                 UCSC        na                    8364     ✗ ✗ ✗ ✗       Xenopus tropicalis    Oct. 2004 (JGI 3.0/xenTro1)
xenTro2                 UCSC        na                    8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Aug. 2005 (JGI 4.1/xenTro2)
xenTro3                 UCSC        GCA_000004195.1       8364     ✗ ✓ ✓ ✗       Xenopus tropicalis    Nov. 2009 (JGI 4.2/xenTro3)
xenTro7                 UCSC        GCA_000004195.2       8364     ✓ ✓ ✗ ✗       Xenopus tropicalis    Sep. 2012 (JGI 7.0/xenTro7)
xenTro9                 UCSC        GCA_000004195.3       8364     ✓ ✓ ✓ ✗       Xenopus tropicalis    Jul. 2016 (Xenopus_tropicalis_v9.1/xenTro9)
 ^
 Use name for genomepy install
```

Note that searching doesn't work flawlessly, so try a few variations if you don't get any results.

#### List available providers

```
$ genomepy providers
Ensembl
UCSC
NCBI
URL
```

#### List available genomes

You can constrain the genome list by using the `-p` option to search only a 
specific provider. 

```
$ genomepy genomes -p UCSC
name                    provider    accession          tax_id     annotation     species                                     other_info
                                                                   n r e k
ailMel1                 UCSC        GCF_000004335.2      9646      ✓ ✗ ✓ ✗       Ailuropoda melanoleuca                      Dec. 2009 (BGI-Shenzhen 1.0/ailMel1)
allMis1                 UCSC        GCA_000281125.1      8496      ✗ ✓ ✗ ✗       Alligator mississippiensis                  Aug. 2012 (allMis0.2/allMis1)
anoCar1                 UCSC        na                  28377      ✗ ✗ ✓ ✗       Anolis carolinensis                         Feb. 2007 (Broad/anoCar1)
```

#### Manage configuration

List the current configuration file that genomepy uses:

```
$ genomepy config file
/home/simon/.config/genomepy/genomepy.yaml
```

To show the contents of the config file:

```
$ genomepy config show
# Directory were downloaded genomes will be stored
genomes_dir: ~/.local/share/genomes/

plugin:
 - blacklist
```

To generate a personal configuration file (existing file will be overwritten):

```
$ genomepy config generate
Created config file /home/simon/.config/genomepy/genomepy.yaml
```

#### Local cache. 

Note that the first time you run `genomepy search` or `list`
the command will take a while as the genome lists have to be downloaded.
The lists are cached locally, which will save time later.
The cached files are stored in `~/.cache/genomepy` and expire after 7 days.
You can also delete this directory to clean the cache using `genomepy clean`.

### Python

Check out our [Python API documentation here](https://vanheeringen-lab.github.io/genomepy/)

```
>>> import genomepy
>>> for row in genomepy.search("GRCh38"):
...    print(row)
...    
['GRCh38.p13', 'Ensembl', 'GCA_000001405.28', 9606, True, 'Homo sapiens', '2014-01-Ensembl/2021-03']
['hg38', 'UCSC', 'GCA_000001405.15', 9606, [True, True, False, True], 'Homo sapiens', 'Dec. 2013 (GRCh38/hg38)']
['GRCh38', 'NCBI', 'GCF_000001405.26', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p1', 'NCBI', 'GCF_000001405.27', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p2', 'NCBI', 'GCF_000001405.28', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p3', 'NCBI', 'GCF_000001405.29', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p4', 'NCBI', 'GCF_000001405.30', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p5', 'NCBI', 'GCF_000001405.31', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p6', 'NCBI', 'GCF_000001405.32', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p7', 'NCBI', 'GCF_000001405.33', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p8', 'NCBI', 'GCF_000001405.34', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p9', 'NCBI', 'GCF_000001405.35', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p10', 'NCBI', 'GCF_000001405.36', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p11', 'NCBI', 'GCF_000001405.37', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p12', 'NCBI', 'GCF_000001405.38', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']
['GRCh38.p13', 'NCBI', 'GCF_000001405.39', 9606, True, 'Homo sapiens', 'Genome Reference Consortium']

>>> genomepy.install_genome("hg38", "UCSC", genomes_dir="./data/genomes")
Downloading genome from UCSC.
Target URL: http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz...
Genome download successful, starting post processing...
name: hg38
local name: hg38
fasta: ./data/genomes/hg38/hg38.fa

>>> g = genomepy.Genome("hg38", genomes_dir="./data/genomes")
>>> g["chr6"][166502000:166502100]
>chr6:166502001-166502100
tgtatggtccctagaggggccagagtcacagagatggaaagtggatggcgggtgccgggggctggggagctactgtgcagggggacagagctttagttct
```

The `genomepy.Genome()` method returns a Genome object. This has all the
functionality of a `pyfaidx.Fasta` object, 
see the [documentation](https://github.com/mdshw5/pyfaidx) for more examples on how to use this.

## Known issues

Genomepy utilizes external databases to obtain your files.
Unfortunately this sometimes causes issues.
Here are some of the more common issues, with solutions.

Let us know if you encounter issues you cannot solve by creating [a new issue](https://github.com/vanheeringen-lab/genomepy/issues).

### Provider is offline
Occasionally one of the providers experience connection issues, which can last anywhere between minutes to hours.
When this happens genomepy will warn that the provider appears offline, or that the URL seems broken.

If the issue does not pass, you can try to reset genomepy.
Simply run `genomepy clean` on the command line, or run `genomepy.clean()` in Python.

### This genome is missing
Genomepy stores provider data on your computer to rerun it faster later.
If a provider was offline during this time, it may miss (parts of) the data.

To re-download the data, remove the local data with `genomepy clean`, then `search` for your genome again.

### This genome is STILL missing/URL is broken
Sadly, not everything (naming, structure, filenames) is always consistent on the provider end.
Contact the provider to get it fixed!
One notable group are Ensembl fungi, which seems to be mostly mislabelled.

In the meantime, you can still use the power of genomepy by manually retrieving the URLs,
and downloading the files with `genomepy install GENOME_URL -p url --url-to-annotation ANNOTATION_URL`.

### Which genome/gene annotation to use
Each provider has its pros and cons:
* Ensembl has excellent gene annotations, but their chromosome names can cause issues with some tools.
* UCSC has an excellent genome browser, but their gene annotations vary in format.
* NCBI allows public submissions, and so has the latest versions, although not always complete or error free.

Use `genomepy search` to see your options, and `genomepy annotation` to check the quality of the gene annotation(s).

### The genomepy config was corrupted
You can create a new one with `genomepy config generate` on command line,
or `genomepy.manage_config("generate")` in Python.

## Getting help

If you want to report a bug or issue, or have problems with installing or running the software please create [a new issue](https://github.com/vanheeringen-lab/genomepy/issues).
This is the preferred way of getting support.
Alternatively, you can [mail me](mailto:simon.vanheeringen@gmail.com).

## Citation

If you use genomepy in your research, please cite it: [10.21105/joss.00320](http://dx.doi.org/10.21105/joss.00320).

## Contributing

Contributions welcome! Send me a pull request or [get in touch](mailto:simon.vanheeringen@gmail.com).

When contributing a PR, please use the [develop](https://github.com/vanheeringen-lab/genomepy/tree/develop) branch.

### Quick development setup: 
1. Fork & download this repo. 
2. `cd` into your local repo. 
3. `git checkout develop`
4. `conda env create python=3.6 -f environment.yaml`
5. `conda activate genomepy`
6. `python setup.py develop`
7. `python setup.py build`
8. `git checkout -b` your_develop_branch

The command line and python imports will now use the code in your local repo. 
To test your changes locally, run the following command:
`pytest -vv --disable-pytest-warnings`

## Contributors

- Siebren Frölich - [@siebrenf](https://github.com/siebrenf)
- Simon van Heeringen - [@simonvh](https://github.com/simonvh)
- Maarten van der Sande - [@Maarten-vd-Sande](https://github.com/Maarten-vd-Sande)
- Dohoon Lee - [@dohlee](https://github.com/dohlee)
- Jie Zhu - [@alienzj](https://github.com/alienzj)

## License

This module is licensed under the terms of the [MIT license](https://opensource.org/licenses/MIT).
