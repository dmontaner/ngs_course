% NGS data analysis tutorial: <br> Variant Calling
% [Estudios in silico en Biomedicina](http://www.uv.es/bioinfor/) <br> _Máster en Bioinformática, Universidad de Valencia_
% [David Montaner](http://www.dmontaner.com) <br> _2014-11-04_


<!-- Common URLs: Tools -->

[bwa]:http://bio-bwa.sourceforge.net/ "Burrows-Wheeler Aligner"
[samtools]:http://samtools.sourceforge.net/ "SAMtools old site"
[samtools-new]:http://www.htslib.org/ "SAMtools new site"

[samtools-man]:http://samtools.sourceforge.net/samtools.shtml "SAMtools online manual"

[bcftools]:http://samtools.sourceforge.net/samtools.shtml#4  "BCFtools online manual (section of the SAMtools online manual)"

[igv]:http://www.broadinstitute.org/igv/ "Integrative Genomics Viewer home page"


[SAMstat]:http://sourceforge.net/projects/samstat/ "SAMstat home page"


<!-- Common URLs: File Formats -->

[sam]:http://samtools.github.io/hts-specs/SAMv1.pdf   "SAM/BAM specification"
[vcf]:http://samtools.github.io/hts-specs/VCFv4.2.pdf "VCF specification"

[allformats]:https://github.com/samtools/hts-specs  "SAM/BAM and VCF actual specifications"

[sam-sum]:http://samtools.sourceforge.net/samtools.shtml#5   "SAM format summary"
[vcf-sum]:http://samtools.sourceforge.net/samtools.shtml#6   "VCF format summary"


<!-- External URLs -->



Preliminaries
================================================================================

Software used in this practical:
----------------------------------------

- [SAMtools] : Samtools is a suite of programs for interacting with high-throughput sequencing data; mostly with SAM/BAM.
  Find the manual page [here][samtools-man].
- [bcftools] : Part of the SAMtools suit for handling BCF files
- [IGV] : Integrative Genomics Viewer
- [SAMstat] : some statistics of the mapped reads.

File formats explored in this practical:
----------------------------------------

- [SAM/BAM][SAM] : Sequence Alignment/Map format. A TAB-delimited text format storing the alignment information. A _header section_ is optional. BAM is the binary format.  
  See [SAM format summary][sam-sum]
- [VCF] : Variant Calling Format  
  See [VCF format summary][vcf-sum]
- __pileup__ : Is an intermediate file format generated by `samtools mpileup` to store variant information.  
  [See samtools mpileup description](http://samtools.sourceforge.net/samtools.shtml#3)
- __BCF__ : binary call format a binary version of the __pileup__ file format.


Data used in this practical:
----------------------------------------

- __s050_sorted.bam__ : the BAM file generated using [BWA] in the previous sessions. It has been sorted using [SAMtools].
- __f000_chr21_ref_genome_sequence.fa__ : the reference genome.


Overview
================================================================================

- We will use [SAMstat] to get some statistics of our alignment.
- We will use `samtools` to get a VCF file of our data.


Exercise 1
================================================================================

Use `samstat` to get a statistical report of our reads:

    samstat s050_sorted.bam

Explore the report generated.

- Does the bam file need to be sorted?
- Try with the unsorted data.


Exercise 2
================================================================================

Retrieve the __s050_sorted.bam__ file generated in the previous practices.

<!-- data_test directory to run my examples
    rm -r data_test
    mkdir data_test
    cp data/s050_sorted.bam data_test
    cp data/f000_chr21_ref_genome_sequence.fa data_test
-->

    cd data_test


Explore the BAM file
--------------------------------------------------------------------------------

The header section

    samtools view -H s050_sorted.bam

The alignment section

    samtools view s050_sorted.bam | head

Is the file sorted?

    samtools view s050_sorted.bam | cut -f 4 | head


Create a PILEUP file
--------------------------------------------------------------------------------

We can use `samtools mpileup` to explore the variants in our data.

This command will send the information to the _standard output_

    samtools mpileup -f f000_chr21_ref_genome_sequence.fa s050_sorted.bam

So we can redirect it to a file doing 

    samtools mpileup -f f000_chr21_ref_genome_sequence.fa s050_sorted.bam > s060_pileup.txt

- Why does `mpileup` need the reference genome? <!-- to know the alternative to each match or mismatch; such information is not in the sam file -->
- Why is there a _.fai_ file now in the directory? <!-- the reference index has been created by samtools -->
- What would happen if the BAM file was not  sorted?  
  Try ```samtools mpileup -f f000_chr21_ref_genome_sequence.fa s040_reads.bam```


Explore the _pileup_ file generated.

    head s060_pileup.txt 

A description of the file may be found in the [samtools mpileup](http://samtools.sourceforge.net/samtools.shtml#4) manual.

- Which is the first position _covered_ by our sequences?
- How does it match with the information in the bam file?
- How many "consecutive" bases are there in the firs _run_  
  Try ```head -n 101 s060_pileup.txt``
- How did we know we had to set `-n 101` in the command above?
- What are the meanings of the symbols in the fifth column of the _pileup_ file?


Binary Format
--------------------------------------------------------------------------------

The information in the _pileup_ file may also be kept in a _binary_ and __compressed__ format
by using the option `-g` in `samtools mpileup`.

    samtools mpileup -g -f f000_chr21_ref_genome_sequence.fa s050_sorted.bam > s060_pileup.bcf


- Can we create a binary but not compressed file?
- When may this be useful?  
  (See the `samtools mpileup` help)


Create a VCF file
--------------------------------------------------------------------------------

Form the BCF file we can now get a [VCF] file: 

    bcftools view s060_pileup.bcf > s070_variants.vcf


Try to concatenate the _mpileup_ and the _bcftools_ step

    samtools mpileup -u -f f000_chr21_ref_genome_sequence.fa s050_sorted.bam | bcftools view -> my.vcf

Use `diff` or `colordiff` to find out the differences between the two VCF files created. 

- Why may this be useful? <!-- Because the BCF file is not really needed -->


Explore the VCF file created.






<!-- ---------------------------------------------------------------------------

    bcftools view s060_pileup_g.bcf > s070_pileup_g.vcf
    bcftools view s060_pileup_u.bcf > s070_pileup_u.vcf

    colordiff s070_pileup_u.vcf s070_pileup_g.vcf 

each line represents a genomic position

| bcftools view bvcg

samtools mpileup [-EBug] [-C capQcoef] [-r reg] [-f in.fa] [-l list] [-M capMapQ] [-Q minBaseQ] [-q minMapQ] in.bam [in2.bam [...]]
Generate BCF or pileup for one or multiple BAM files. Alignment records are grouped by sample identifiers in @RG header lines. If sample identifiers are absent, each input file is regarded as one sample.

In the pileup format (without -u or -g),
each line represents a genomic position,
consisting of
1. chromosome name,
2. coordinate,
3. reference base,
4. read bases,
5. read qualities and
6. alignment mapping qualities.

Information on match, mismatch, indel, strand, mapping quality and start and end of a read are all encoded at the __read base column__.
At this column,
a __dot__ stands for a __match__ to the reference base on the __forward__ strand,
a __comma__ for a __match__ on the __reverse__ strand,
a ’>’ or ’<’ for a reference skip,
‘ACGTN’ for a __mismatch__ on the __forward__ strand and
‘acgtn’ for a __mismatch__ on the __reverse__ strand.
A pattern ‘\+[0-9]+[ACGTNacgtn]+’ indicates there is an __insertion__ between this reference position and the next reference position.
The length of the insertion is given by the integer in the pattern, followed by the inserted sequence.
Similarly, a pattern ‘-[0-9]+[ACGTNacgtn]+’ represents a __deletion__ from the reference.
The deleted bases will be presented as ‘*’ in the following lines. Also at the read base column, a symbol ‘^’ marks the start of a read. The ASCII of the character following ‘^’ minus 33 gives the mapping quality. A symbol ‘$’ marks the end of a read segment.


SOLO PARA LAS REGIONES DE COBERTURA

-------------------------------------------------------------------------------------------

Perdona David, se me olvidó pasarte el comando más sencillo de todos para hacer conteos con samtools:

samtools idxstats BAM > OUTPUT

el BAM debe estar ordenado con sort e indexado con index (generar el BAI)

Como resultado se obtiene un archivo tabulado con el gen, longitud, lecturas mapeadas y lecturas no mapeadas

Un saludo

-------------------------------------------------------------------------------------------

Hola David,

Te mando unos ejemplos de samttols para obtener el coverage, el coverage por región y hacer un recuento de lecturas mapeadas:

1. samtools -S -O -f REF BAM > OUTFILE (coverage por base). REF en formato fasta
2. coverageBed -split -abam BAM -b BED > OUTFILE (coverage por región, indica el número de lecturas mapeadas por región o gen). REF en  formato BED
3. samtools view -c BAM (número total de alineamientos)
4. samtools view -c -F 4 BAM (número de lecturas mapeadas)
5. samtools view -c -f 4 BAM (número de lecturas no mapeadas)
6. samtools view -c -f 1 -F12 BAM (número de lecturas mapeadas en datos paired-end)

Un saludo



-------------------------------------------------------------------------------------------

Hola David,

Te mando los comandos de samstat y el variant calling con samtools:

1. El informe con samstat se genera de una manera bastante sencilla:

samstat <BAM>

En el mismo directorio se generará un archivo HTML con el mismo nombre del BAM


2. Para realizar el variant calling con samtools:
    2.1. Hay que generar el BCF con samtools
    2.2. Covertir el BCF en VCF

outdir = variant_calling.bcf
outdir2 = variant_calling.vcf
REF = Genoma e referencia indexado (bowtie, bwa...)

samtools mpileup -uD -f REF BAM | bcftools view bvcg -> outdir

-u genera un BCF no comprimido
-D genera un output por muestra
-f genoma de referencia

bcftools view outdir | /usr/share/samtools/./vcfutils.pl varFilter -Q20 -d30 -D1000 > outdir2

-Q quality score (mapping)
-d profundidad minima
-D profundidad máxima

-->