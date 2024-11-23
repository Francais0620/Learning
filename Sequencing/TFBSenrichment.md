## 判断转录因子的结合位点是否富集在特定的基因组区域（L1HS，L1PA2等）| 生信分析

> 项目需求：  
> 最近关注ZBTB18这个基因，发现其在L1PA1上有特异性的motif，但下载相关的chip-seq的数据做分析时，总是信号比较杂。  
> 我想知道这个转录因子的结合位点是否特异性的富集在L1PA1上，这样的话就为我们motif的结果提供一些可能性的解释。

### Step1：原理学习

在文献（PMID:27852650）中，对131 C2H2-ZF蛋白的chip-seq进行分析。   
https://genome.cshlp.org/content/26/12/1742/F2.expansion.html   
For proteins whose motif-containing peaks overlap significantly with repeat elements at FDR <0.01, the repeat type with the most significant enrichment (i.e., the smallest P-value) is shown on the right. The squares on the left of the proteins represent effector domains. 

#### 1. 数据准备
 - [x] (1) ChIP-seq Peaks
  获取转录因子ChIP-seq的peaks文件（通常是BED格式）。
  每个peak定义一个基因组区域，代表转录因子潜在的结合位点。
-  [x] (2) 重复序列注释
  使用基因组的重复序列注释文件（如RepeatMasker输出文件，通常是GTF或BED格式），标注不同类型的重复序列（如LINEs、SINEs、LTRs、DNA转座子等）。
  确保这些注释与ChIP-seq实验使用的参考基因组版本一致（如GRCh38或mm10）。
-  [x] (3) 背景基因组区域
  定义背景区域（background regions），通常包括以下选项：
  随机基因组区域：从全基因组随机生成一组长度和数量与peaks一致的区域。
  可及基因组区域（accessible genome）：从ATAC-seq等实验数据或参考基因组定义的可及区域中提取。
  转录因子偏好区域：结合转录因子绑定的偏好区域，如启动子、增强子。

#### 2. 富集分析方法  
 - [x] (1) 计算重叠
使用工具（如BEDTools或其他基因组操作工具）计算ChIP-seq peaks与重复序列的重叠：
将peaks与重复序列进行交集计算，得到每种重复序列类型与peaks的重叠数（observed overlaps）。
同时计算背景区域与重复序列的重叠数（expected overlaps）。

 - [x] (2) 统计测试:进行富集显著性测试，常见方法包括：

    - a) 超几何检验（Hypergeometric Test）,检测某种重复序列的重叠是否显著超过随机期望。
  
        ![image](https://github.com/user-attachments/assets/0b0fedf1-955a-4dba-9ced-8b6730d062b5)
  
    - b) 如果样本较小，可以使用Fisher精确检验替代超几何检验。
  
    - c) Chi-square Test用于大样本的类别数据，比较观察频数和期望频数是否显著不同。

 - [x] (3) 多重检验校正
由于基因组中重复序列类型众多（可能有几十种），需要对多重假设检验进行校正：
常见校正方法：Benjamini-Hochberg（BH）法，用于控制假发现率（FDR）。


#### 3. 富集分析工具
可以直接使用一些现成的工具或软件包来进行重复序列富集分析：

**BEDTools intersect：** 计算peaks与重复序列的重叠。  
**HOMER：** 支持ChIP-seq peaks的重复序列富集分析（annotatePeaks.pl）。  
**R/Bioconductor包：**  
regioneR：支持基因组区域的随机化与富集分析。  
LOLA：适用于区域与注释集合的比较分析。  

#### 4. 结果解释与可视化

 - [x] (1) 显著性排名
根据P值或校正后的FDR值，对重复序列类型进行排序，标记最显著富集的类型。

 - [x] (2) 富集倍数（Fold Enrichment）
计算富集倍数，帮助量化富集强度：
![image](https://github.com/user-attachments/assets/af396c08-1965-4de3-a9e6-1ed67de79d79)

 - [x] (3) 可视化
使用柱状图或热图展示每种重复序列类型的富集倍数或显著性。
例如：
横轴：重复序列类型。
纵轴：富集倍数或-log10(𝑃)。

#### 5. 示例：具体分析流程
假设你想分析某转录因子的ChIP-seq peaks是否富集于LINEs：

用BEDTools计算peaks与LINEs的交集，得到重叠数量（观察值）。
用随机区域模拟，计算预期重叠数量。
使用超几何检验或Fisher检验计算P值。
校正多重检验的P值。
输出LINEs的富集倍数和显著性。
总结
通过对ChIP-seq peaks和重复序列的重叠计算及显著性测试，可以判断转录因子结合位点是否显著富集于某种重复序列。结合富集倍数和P值排名，可以直观地展示最显著的重复序列类型。



### Step2：代码实现

### Step3：demo测试

### Step4：应用于自己的数据

