### 1. ChIP-seq的QC

#### （1）mapping ratio（比对率）
> 测序读数（reads）成功比对到参考基因组的比例。比对率越高，说明测序数据与参考基因组的一致性越高。  
> 一般而言，高比对率（＞80%）被认为样本质量好，适合进一步分析。低比对率提示可能出现样本污染，或者参考基因组不适合当前样本的来源物种。  
> 比对工具（Bowtie2，BWA）的输出文件和质控工具（FastQC）会对比对率有详细的报告。

mapping ratio = $\frac{成功比对到参考基因组的reads的数目}{测序数据所有的reads的数目}$



#### （2）read depth（测序深度）
> 基因组上每个碱基被测序的平均次数。
> 反映实验是否有足够的数据来检测蛋白质-DNA的结合位点，特别是在峰值区域（peak）处，需要更高的测序深度来保证检测的可靠性。
> 高测序深度通常能够提高信噪比（如何理解这边的信噪比），从而更准确的识别真正的peak。
> 在比较不同样本时，通常需要保持类似的测序深度，以确保数据的可比性（有标准化的方法来处理这一问题吗？）。
> 参考标准：＞10 million唯一比对的读数（sharp-mode peak）
>           ＞40 million唯一比对的读数（broad-mode peak，平均片段更长）

read depth = $\frac{成功比对到基因组上的碱基的数目}{基因组的大小}$




#### （3）Library complexity（文库复杂度）
> 测序文库中独立，非重复片段的多样性和丰富程度（这一点如何评估？就是片段重复的比例多不多？）。  
> 高复杂度意味着文库中有大量不同的DNA片段，能够检测到更多的独特的结合位点。  
> 低复杂度的文库可能包含大量重复的片段（我们要留心一下这一点），可能因为起始DNA量少，PCR扩增次数过多等。
> 参考标准：＞0.8 for 10 million mapped reads
> 常用工具：Picard工具，使用“EstimateLibraryComplexity”功能计算文库的复杂度。  


评估方法：
a. PCR重复率：测序数据中的重复读取的片段比例。较高的PCR重复率可能表明样本复杂度较低。  
（关于这一点，我一直没咋想明白，怎么区分PCR重复和生物学意义的重复？PCR的过程中怎么引入了重复？）  
b. 非冗余的读段的数目：去除重复的测序读数后，剩下的唯一的读数的数目。数目越高，文库复杂度越高。


#### （4）The normalized strand coefficient（NSC）

#### （5）Background uniformity（Bu）

#### （6）GC summit bias


### 2. pipeline


| Software     | Description | Code    | notes | 
|----|----|----|---- |
| prefetch      |Step1:prefetch .sra files from database       | `prefetch XXX`   | download |
| fastq-dump   |Step2:split files from .sra files        | `fastq-dump XXX`      | download|
| trim_galore        |Step3:filtering          | `trim_galore --fastqc --three_prime_clip_R1 4  -o ./0-clean_data/ SRR14104347/SRR14104347.fastq.gz` | preprocessing |
| bowtie2 |Step4:alignment |`bowtie2 -x /home/xxzhang/Resource/Reference/Genome/homo_sapiens/hg38/bowtie2/hg38_dm6 -p 30 -U "/home/xxzhang/workplace/project/CRISPRa/chip_seq/0-clean_data/SRR14104347_trimmed.fq.gz" >./0-align/SRR14104347.aligned.sam `|preprocessing |
|gatk |Step5:revome duplicates |` ` | preprocessing|
|samtools |Step5:revome duplicates |`samtools rmdup` | preprocessing|
| | |``|format conversion|
|bamcoverage|Step:normalization|`bamCoverage -p $nThread -b bam/${sample}.rmdup.bam --skipNAs --normalizeUsing CPM -o bw/${sample}.CPM.bw`| processing |
|MACS |Step6:call peak |`macs2 callpeak -t <sample>._sort.bam -c <sample>._sort.bam` | processing|
|computeMatrix|scale|`computeMatrix scale-regions -S <sample1>.bw  input.<sample2>.bw -R <overlap>.bed -a 1000 -b 1000 -o ip_input.computeMatrix`
`computeMatrix reference-point --referencePoint center -b <sample>.bed -S <sample>.bw <sample>.bw  --skipZeros -o <sample>.gz --outFileNameMatrix <sample>`|compute|
|plotHeatmap||||
|plotProfile||||











