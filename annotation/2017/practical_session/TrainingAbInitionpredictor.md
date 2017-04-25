---
layout: default
title:  'Training Ab-initio'
---

# Training ab-initio predictor

From this maker run evidence base we can train our abinitio softwares and then use them for the second run of annotation. 
You will need a set of genomic sequences with gene structures (sequence coordinates of starts and ends of exons and genes) and the most important part is selected the right set of genes. 
In many cases, or as a first step towards modelling complete genes, it is sufficient to have only the coding parts of the gene structure (CDS).
We will only train augustus today as it is one the best ab-initio predictor and one of the hardest to train.
Maker also support SNAP (Works good, easy to train, not as good as others ab-initio especially on longer intron genomes), GeneMark (Self training, no hints, buggy, not good for fragmented genomes or long introns).
FGENESH (Works great, costs money even for training) and now EVM.


## Training Augustus

First you need to write the librabries path you will need in .bash_profile to perform the following analyses.

*/home/login/annotation_course/course_material/lib/install_perllib_missing.sh*

*source ~/.bash_profile*

Then load all modules that we will need to train Augustus

*module load bioinfo-tools  
module load perl/5.18.4  
module load perl_module/5.18.4  
module load BioPerl/1.6.923_Perl5.18.4   
module load cufflinks/2.2.1*

Create project folder

We create a new folder in which to store all the configuration and input files. To do so, type:

*mkdir train_augustus
cd train_augustus*

You will need to symlink all data you will need such as the gff files from the first run of maker and the chromosome 4 fasta sequence.

*ln -s ~/practical2/maker_no_abinitio/annotationByType/maker.gff dmel_results_noAbinitio.gff*

## Compile a set of training and test genes

First step is to select only the coding genes from the maker.gff file and remove all tRNA

*/home/login/course_material/git/GAAS/annotation/Tools/Maker/maker_gff3manager_JD_v8.pl -f dmel_results_noAbinitio.gff -o dmel_results_noAbinitio_clean*

*cd dmel_results_noAbinitio_clean*

In this folder you will need to create different folders

*mkdir filter  
mkdir protein  
mkdir nonredundant  
mkdir blast_recursive  
mkdir gff2genbank*

Next step, we need to filter the best genes we will use for the training, we need complete genes, models with a distance with an other model over 500 nt and a AED score over 0.3 (those are our default parameters).

*~/annotation_course/course_material/scripts/filter_sort.pl -file dmel_results_clean/codingGeneFeatures.gff -F 4.fa -o filter/codingGeneFeatures.filter.gff -c -r -d 500 -a 0.3*

we also need to select the longest ORF

*~/annotation_course/course_material/scripts/find_longest_CDS.pl -f filter/codingGeneFeatures.filter.gff -o codingGeneFeatures.filter.longest_cds.gff*

There are different ways of proceding after the first selection and we are using the approched of spliced alignments of protein sequences of the same or a very closely related species against the assembled genomic sequence.
In order to do so, we translate our coding genes into proteins, format the protein fasta file to be able to run a recursive blast and then select the best ones.
Indeed, each sequence can contain one or more genes, the genes can be on either strand. However, the genes must not overlap, and only one transcript per gene is allowed.

*gffread -y codingGeneFeatures.filter.longest_cds.tmp -g 4.fa codingGeneFeatures.filter.longest_cds.gff  
~/annotation_course/course_material/scripts/fix_fasta.sh codingGeneFeatures.filter.longest_cds.tmp > protein/codingGeneFeatures.filter.longest_cds.proteins.fa*  

*rm codingGeneFeatures.filter.longest_cds.tmp*

*module load blast  

*makeblastdb -in protein/codingGeneFeatures.filter.longest_cds.proteins.fa -dbtype prot*  

*blastp -query protein/codingGeneFeatures.filter.longest_cds.proteins.fa -db codingGeneFeatures.filter.longest_cds.proteins.fa -outfmt 6 -out blast_recursive/codingGeneFeatures.filter.longest_cds.proteins.fa.blast_recursive*

*~/annotation_course/course_material/git/GAAS/annotation/Tools/Util/gff/gff_filter_by_mrna_id.pl --gff codingGeneFeatures.filter.longest_cds.gff --blast blast_recursive/codingGeneFeatures.filter.longest_cds.proteins.fa.blast_recursive --outfile nonredundant/codingGeneFeatures.nr.gff*

*module load augustus/3.2.3*

The sequences need to be converted in a simple genbank format.

*gff2gbSmallDNA.pl nonredundant/codingGeneFeatures.nr.gff 4.fa 500 gff2genbank/codingGeneFeatures.nr.gbk*

In order for the test accuracy to be statistically meaningful the test set should also be large enough (100-200 genes). 
You should split the set of gene structures randomly.

*randomSplit.pl gff2genbank/codingGeneFeatures.nr.gbk 100*

- What happenned? how can you solve it? what might be the consequences of it? 


## Train Augustus

Now that you have created a det of gene to train augustus, let's train it!

Augustus need a set of parameters that are provided :

*cp -r ~/annotation_course/course_material/augustus_path/ .*

*new_species.pl --AUGUSTUS_CONFIG_PATH=augustus_path --species=dmel_login*

*AUGUSTUS_CONFIG_PATH=augustus_path*

*etraining --species=dmel_login gff2genbank/codingGeneFeatures.nr.gbk.train* 

*augustus --species=dmel gff2genbank/coding.nr.gbk.test | tee run.log* 

- Look at the accuracy report, what does it mean? why?