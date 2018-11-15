# CAM-PIPELINE

Pipeline

A pipeline utilizes the pipe symbol (|) to string together various commands to play out a particular job. What a pipeline does is cuts the commands into small parts which is separated by a pipe and then these commands are attached together so that the output of one becomes as input of other which is basically called Piping .These pipelines are utilized for finding a specific data or facts on one’s system. Normally what we do is we take the data, give that as an input and the result we get is the output. Here, Pipe allows us to change this pattern where the output of one file becomes the input of other file.

For an instance, there could be a significant number of folders recorded, and it's very conceivable that a portion of the outcomes may have looked off the highest point of the screen, before anyone could see them. Yet, one could settle that by sending the yield of the ls command to the less command, which would enable them to look through the outcomes, similarly as though they were utilizing less to peruse a text document.

Each procedure takes contribution from the past procedure and produces yield for the following procedure by means of standard streams. Each " | " advises the shell to interface the standard yield of the order on the left to the standard contribution of the direction on the privilege by a between procedure correspondence component called a pipe, actualized in the working framework. 

Processa | Processb | Processc 

The standard errors of the procedures in a pipeline are not gone on through the pipe; rather, they are combined and coordinated to the console. A pipe is settled in size and does have minimum like 4,096 bytes.



Overview of the workflow:

Firstly the Raw reads are taken which are in Fastq format, which are mapped to reference (SAM) using BWA-MEM and then sorted by coordinate and converted to BAM using Picard giving out Mark duplicates , Alignment metrics and Insert size metrics which are text file and histogram respectively.

These Mark duplicates are then converted to Build BAM Index using Picard , then are used to create the realignment targets which are used to realign around indels(BAM) by GATK

Then this BAM is converted to VCF file by variant calling


##script
#!/bin/bash
echo "hi welcome to cam pipeline"
#ref gives the  command which is used to download the reference genome
REF= wget ftp://ftp.ncbi.nlm.nih.gov/genomes/archive/old_genbank/Eukaryotes/vertebrates_mammals/Homo_sapiens/GRCh38/seqs_for_alignment_pipelines/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.bowtie_index.tar.gz

#REF=$1 #reference genome
## Modules Used in this script
BEDTOOLS='bedtools/intel/2.25.0'
BWA='bwa/intel/0.7.17'
PICARD='picard/1.4.2-1'
GATK='gatk/4.0.11.0'
R='r/intel/3.5.2'
SAMTOOLS='samtools/intel/1.9.0'
## Paths to Modules
PICARD_JAR='/mnt/c/Users/chait/Desktop/picard/1.4.2-1/picard-1.4.2-1.jar'
GATK_JAR='/mnt/c/Users/chait/Desktop/gatk/4.0.11.0/gatk.jar'
BEDTOOLS='/mnt/c/Users/chait/Desktop/bedtools/2.25.0/bedtools.jar'
## Get the current working directory
WD="$(pwd)"
## Steps of the Workflow
## using bwa for aligned the reads 
pre_process(){
com="cd $FWD && \
module load $BWA && \
bwa mem ref.fa > aligned_reads.sam \
$REF \
$INPUT_1\
> ${ID}_aligned_reads.sam"
response=\
stringarray=($response)
alignment=${stringarray[-1]}
echo $aligning > $ID.log
echo "ALIGNMENT: " $aligning
echo "Alignment done by using bwa"
##using picard for sorting the reads aligned by  bwa
com="cd $FWD && \ 
module load $PICARD && \
java -jar $PICARD_JAR \
SortSam \
INPUT=${ID}_aligned_reads.sam \
OUTPUT=${ID}_sorted_reads.bam \
SORT_ORDER=coordinate"
response=\
stringarray=($response)
samToSortedBam=${stringarray[-1]}
echo $samToSortedBam >> $ID.log
echo "SAMTOSORTEDBAM: " $samToSortedBam
echo "samToSortedBam Submitted by using picard"
##using picards for getting the metrics
com="cd $FWD && \
module  load $PICARD && \
module load $R && \
module load $SAMTOOLS && \
java -jar $PICARD_JAR \
CollectAlignmentSummaryMetrics \
R=$REF \
I=${ID}_sorted_reads.bam \
O=${ID}_alignment_metrics.txt && \
java -jar $PICARD_JAR \
CollectInsertSizeMetrics \
INPUT=${ID}_sorted_reads.bam \
OUTPUT=${ID}_insert_metrics.txt \
HISTOGRAM_FILE=${ID}_insert_size_histogram.pdf && \
samtools depth -a ${ID}_sorted_reads.bam > ${ID}_depth_out.txt"
response=\
stringarray=($response)
getMetrics=${stringarray[-1]}
echo $getMetrics >> $ID.log
echo "GETMETRICS: " $getMetrics
echo "getMetrics Submitted"
##using picard for markin the duplicates
com="cd $FWD && \ 
rm ${ID}_aligned_reads.sam && \
module load $PICARD && \
java -jar $PICARD_JAR \
MarkDuplicates \
INPUT=${ID}_sorted_reads.bam \
OUTPUT=${ID}_delup_reads.bam \
METRICS_FILE=${ID}_metrics.txt" 
response=\
stringarray=($response)
markDuplicates=${stringarray[-1]}
echo $markDuplicates >> $ID.log
echo "MARKDUPLICATES: " $markDuplicates
echo "Mark Duplicates Submitted"
com="cd $FWD && \ 
module load $PICARD && \
java -jar $PICARD_JAR \
BuildBamIndex \
INPUT=${ID}_delup_reads.bam" 
response=\
stringarray=($response)
buildBamIndex=${stringarray[-1]}
echo $buildBamIndex >> $ID.log
echo "BUILDBAMINDEX: " $buildBamIndex
echo "BuildBamIndex Submitted"
com="cd $FWD && \ 
rm ${ID}_sorted_reads.bam && \
module load $GATK && \
java -jar $GATK_JAR \
-T RealignerTargetCreator \
-R $REF \
-I ${ID}_delup_reads.bam \
-o ${ID}_realignment_targets.list"
response=\
stringarray=($response)
realignTargetCreator=${stringarray[-1]}
echo $realignTargetCreator >> $ID.log
echo "REALIGNTARGETCREATOR: " $realignTargetCreator
echo "RealignTargetCreator Submitted"
com="cd $FWD && \
module load $GATK && \
java -jar $GATK_JAR \
-T IndelRealigner \
-R $REF \
-I ${ID}_delup_reads.bam \
-targetIntervals ${ID}_realignment_targets.list \
-o ${ID}_realigned_reads.bam"
response=\
stringarray=($response)
realignIndels=${stringarray[-1]}
echo $realignIndels >> $ID.log
echo "REALIGNINDELS: " $realignIndels
echo "RealignIndels Submitted"	
}
call_variants(){
	ROUND=$1
	if [[ $ROUND -eq 1 ]];then
		INPUT=${ID}_realigned_reads.bam
		OUTPUT=${ID}_raw_variants.vcf
		AFTEROK=$realignIndels
	fi
	if [[ $ROUND -eq 2 ]];then
		INPUT=${ID}_recal_reads.bam
		OUTPUT=${ID}_raw_variants_recal.vcf
		AFTEROK=$applyBqsr
	fi
com="cd $FWD && \
module load $GATK && \
java -jar $GATK_JAR \
-R $REF \
-I $INPUT \
-o $OUTPUT" 
response=\
stringarray=($response)
callVariants=${stringarray[-1]}
echo $callVariants >> $ID.log
echo "CALLVARIANTS: " $callVariants
echo "Variant Calling Round $ROUND Submitted"	
	if [[ $ROUND -eq 1 ]];then
		callVariants_1=$callVariants
	fi
	if [[ $ROUND -eq 2 ]];then
		callVariants_2=$callVariants
	fi
}
	
## Build Report Header and Create File
REPORT_HEADER="ID,# reads,aligned reads,% aligned,aligned bases,read length,% paired,mean insert size"
echo $REPORT_HEADER > report.csv
for file in $(ls fastq.gz | perl -pe 's/^.+n0\d_(.+)\.fastq\.gz/$1/g' | sort | uniq) 
do

	INPUT_1=$(ls n01_$file.fastq.gz)
	ID=$file
        OUTPUT=$(ls raw_variants.vcf)

	echo "Processing files: " $INPUT_1 "
	FWD=$WD/$ID
	mkdir $FWD
	cd $FWD

	## CALL WORKFLOWs
	pre_process # Only Done Once
	call_variants 1 # Call Variants Round 1
	extract_snps 1 # Round 1. Extracts snps AND indels, separately
	filter_snps 1 # Round 1
	filter_indels 1 # Round 1
	do_bqsr 1 # Do BQSR Round 1
	do_bqsr 2 # Do BQSR Round 2
	analyze_covariates # Only Done Once
	apply_bqsr # Only Done Once
	call_variants 2 # Call Variants Round 2

done



