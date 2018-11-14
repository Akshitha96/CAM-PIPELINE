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

Then this BAM is converted to VCF file by variant calling which extracts SNPs and Indels and to filter SNPs and Filter Indels by GATK and VCF respectively .

They are converted to Predict Variant Effects using the VFC,text and HTML files which makes the CSV report and gets the coverage data by CSV and bedgraph.

The filter SNPs and filter Indels together form the Base quality Score Recalibration using the GATK which gives out the final file, Analyse Covariates(PDF) and also Apply recalibration which is a BAM file which takes you back to the variant calling

