# sonnei_genotype.py

This script assigns genotypes to *Shigella sonnei* genomes and detectes known mutations in the quinolone-resistance determining region (QRDR) of genes *gyrA* and *parC*. 

It is based on the [GenoTyphi](https://github.com/katholt/genotyphi) code developed for genotyping *Salmonella* Typhi.

Inputs are BAM or VCF files (mapped to the *Shigella sonnei* 53G reference genome, [NC_016822](https://www.ncbi.nlm.nih.gov/nuccore/NC_016822.1)).

For short read data, we recommend using the raw alignments (BAM files) as input (via --bam), since VCFs generated by different pipelines may have filtered out important data at the genotyping loci. Note you also need to supply the reference sequence that was used. If you don't have BAMs or you are really confident in the quality of your SNP calls in existing VCF files, you can provide your own VCFs (generated by read mapping) via --vcf.

For assemblies, we recommend using [ParSNP](http://harvest.readthedocs.org/) (version 1.0.1) to align genomes to the CT18 reference. The resulting multi-sample VCF file(s) can be passed directly to this script via --vcf_parsnp.

Dependencies: Python 2.7.5+ ([SAMtools](http://samtools.sourceforge.net/) and [BCFtools](https://samtools.github.io/bcftools/) are also required if you are working from BAM files.  Genotyphi has been tested with versions 1.1, 1.2, and 1.3 of both SAMtools and bcftools, subsequently we advise using the same version of both of these dependencies together i.e. SAMtools v1.2 and bcftools v1.2).

[Basic Usage](https://github.com/katholt/sonneityping/#basic-usage---own-bam-recommended-if-you-have-reads)

[Options](https://github.com/katholt/sonneityping/#options)

[Outputs](https://github.com/katholt/sonneityping/#outputs)

[Generating input BAMS from reads](https://github.com/katholt/sonneityping/#generating-input-bams-from-reads)

[Generating input VCFs from assemblies](https://github.com/katholt/sonneityping/#generating-input-vcfs-from-assemblies)

## Basic Usage - own BAM (recommended if you have reads)

Note the BAM files must be sorted (e.g. using samtools sort)

```
python sonnei_genotype.py --mode bam --bam *.bam --ref NC_016822.fasta --ref_id NC_016822.1 --allele_table alleles.txt --output genotypes.txt
```

## Basic Usage - own VCF

```
python sonnei_genotype.py --mode vcf --vcf *.vcf --ref_id NC_016822 --allele_table alleles.txt --output genotypes.txt
```

## Basic Usage - assemblies aligned with ParSNP (recommended if you only have assembly data available and no reads)

```
python sonnei_genotype.py --mode vcf_parsnp --vcf parsnp.vcf --allele_table alleles.txt --output genotypes.txt
```

## Options

### Required options

```
-- mode            Run mode, either bam, vcf or vcf_parsnp
```

### Mode specific options

#### --mode bam

Requires [SAMtools](http://samtools.sourceforge.net/) and [BCFtools](https://samtools.github.io/bcftools/)

```
--bam              Path to one or more BAM files, generated by mapping reads to the S. sonnei 53G reference genome (NC_016822)
                   Note the SNP coordinates used here for genotyping are relative to S. sonnei 53G (NC_016822) 
                   so the input MUST be a BAM obtained via mapping to this reference sequence.

--ref              Reference sequence file used for mapping (fasta).

--ref_id           Name of the S. sonnei 53G chromosome reference in your VCF file.
                   This is the entry in the first column (#CHROM) of the data part of the file.
                   This is necessary in case you have mapped to multiple replicons (e.g. chromosome and
                   plasmid) and all the results appear in the same VCF file.

--samtools_location     Specify the location of the folder containing the SAMtools installation if not standard/in path [optional]

--bcftools_location     Specify the location of the folder containing the bcftools installation if not standard/in path [optional]
```

#### --mode vcf

```
--vcf              Path to one or more VCF files, generated by mapping reads to the S. sonnei 53G reference genome (NC_016822)
                   Note the SNP coordinates used here for genotyping are relative to S. sonnei 53G (NC_016822) 
                   so the input MUST be a VCF obtained via mapping to this reference sequence.
```

#### --mode vcf_parsnp

```
--vcf              Path to one or more VCF files generated by mapping assemblies to the S. sonnei 53G genome (NC_016822)
                   using ParSNP (--ref_id is optional, default value is '1').
```

### Other options

```
--phred                 Minimum phred quality to count a variant call vs 53G as a true SNP (default 20)

--min_prop              Minimum proportion of reads required to call a SNP (default 0.1)

--output                Specify the location of the output text file (default genotypes_[timestamp].txt)

--allele_table           Location of allele table containing marker SNPs

```

## Outputs

Output is to standard out, in tab delimited format:

| File | Genotype | Genotype_support | Genotype_name | Subclade | Clade | Lineage | Support_Subclade | Support_Clade | Support_Lineage | QRDR mutations |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| str1.vcf | 2.10.2 | 1 | Latin America IIa | 2.10.2 | 2.1 | 2 | 1 | 1 |   |   |
| str2.vcf | 3.7.29 | 1 | Global III VN | 3.7.29 | 3.7 | 2 | 1 | 1 | 1 |   |   |
| str3.vcf | 3.6.3 | 1 | Central Asia III | 3.6.3 | 3.6 | 2 | 1 | 1 | 1 | gyrA_D87Y |

The script works by looking for SNPs that define nested levels of phylogenetic lineages for _S. sonnei_: lineages, clades and subclades. These are named in the form 1.2.3, which indicates Lineage 1, clade 1.2, and subclade 1.2.3. 

All genotypes implicated by the detected SNPs will be reported (comma separated if multiple are found). 

Usually Lineage, clade and subclade SNPs will be compatible, i.e. you would expect to see 1.2.3 in the subclade column, 1.2 in the clade column and 1 in the Lineage column.

The first column 'Final_call' indicates the highest resolution genotype (subclade > clade > lineage) that could be called.

As recombination is extremely rare in S. sonnei, it is unlikely that DNA isolated from a single S. sonnei culture would have high quality SNPs that are compatible with multiple genotypes, or incompatible clade/subclade combinations... therefore if you see this in the output, it is worth investigating further whether you have contaminated sequence data or a genuine recombination. To provide very basic assistance with this the output file indicates, for each genotype-defining SNP detected, the proportion of reads that support the genotype call. A genuine recombinant strain would have p=1 for multiple incompatible SNPs, whereas a mixed DNA sample might have p~0.5. Note that if you are genotyping from assemblies, we have no information on read support to work with; instead you will see an 'A' in the Final_call_support column to indicate this result comes from a genome assembly.

WARNING: Note the reference genome 53G has the genotype 2.8.2. It is therefore possible that unexpected behaviour (e.g. wrong reference, unspecified reference name, or problems that results in overall poor quality SNP calls in the VCF) can sometimes result in calls of 2.8.2 ... so if you see lots of strains being assigned this genotype, it is worth investigating further.

## Generating input BAMS from reads

We recommend using [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) to align reads to the 53G reference, and [SAMtools](http://http://samtools.sourceforge.net/) to convert the *.sam file to the *.bam format.  The resulting bam file(s) can be passed directly to this script via --bam.  Please note the differences in the commands listed below when using SAMtools v1.1/1.2 vs. SAMtools v1.3.1 to generate bam files.

## Generating input VCFs from assemblies

We recommend using [ParSNP](http://harvest.readthedocs.org/) to align genomes to the 53G reference. The resulting multi-sample VCF file(s) can be passed directly to this script via --vcf_parsnp.
