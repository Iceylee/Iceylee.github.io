
## 命令
#### 常用命令
`snakemake -np` 查看全部流程
`snakemake --snakemakefile myfile.py`  指定snakefile
`snakemake --snakefile res-snake.py -np -R dedup_realn` 重跑rule dedup_realn（dry run），注意如果snakemake认为需要跑的step（rull all输出文件推断）还是会跑的
#### 参数
`--snakefile, -s` 指定Snakefile，否则是当前目录下的Snakefile
`--dryrun, -n`  不真正执行，一般用来查看Snakefile是否有错
`--printshellcmds, -p`   输出要执行的shell命令
`--reason, -r`  输出每条rule执行的原因,默认FALSE
`--cores, --jobs, -j`  指定运行的核数，若不指定，则使用最大的核数
`--force, -f` 重新运行第一条rule或指定的rule
`--forceall, -F` 重新运行所有的rule，不管是否已经有输出结果
`--forcerun, -R` 重新执行Snakefile，执行列出的rule，当更新了rule时候使用此命令

#### 可视化
`$ snakemake --dag | dot -Tpdf > dag.pdf`

#### 集群投递
`snakemake --cluster "qsub -V -cwd -q 节点队列" -j 10
 --cluster` 集群运行指令
 `qusb -V -cwd -q` 表示输出当前环境变量(-V),在当前目录下运行(-cwd), 投递到指定的队列(-q), 如果不指定则使用任何可用队列
` --local-cores N`: 在每个集群中最多并行N核
 `--cluster-config/-u FILE`: 集群配置文件


## 规则
1. Snakemake基于规则执行命令，规则一般由input, output,shell三部分组成。除了rule all，其余必须有output
2. Snakemake可以自动确定不同规则的输入输出的依赖关系，根据时间戳来判断文件是否需要重新生成
3. Snakemake以{sample}.fa形式进行文件名通配，用{wildcards.sample}获取sample的实际文件名
4. Snakemake用expand()生成多个文件名，本质是Python的列表推导式
5. Snakemake可以在规则外直接写Python代码，在规则内的run里也可以写Python代码。
6. Snakefile的第一个规则通常是rule all，根据all里的文件决定执行哪些rule。如上面的例子，注释掉all里的input则不执行第二条rule，（推断未尝试：rule all里定义最终的输出文件，程序也能执行，那么rule里的输出文件在什么时候会被删除，是在所有rule运行完之后，还是在判断出该输出文件不会被用到的时候？）
7. 在output中的结果文件可以是未存在目录中的文件,这时会自动创建不存在的目录（不需要事先建文件夹，这个功能实在是方便）

## wildcards
用来获取通配符匹配到的部分，例如对于通配符”{dataset}/file.{group}.txt”匹配到文件101/file.A.txt，则{wildcards.dataset}就是101，{wildcards.group}就是A。


#### 理解
在rule all的输出文件的时候，定义了我需要的文件名。比如ALL_FASTQ已经将详细的文件名指出。
![image.png](https://upload-images.jianshu.io/upload_images/7901162-22b284a30de63d8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在rule merge_fastqs里，output处会和ALL_FASTQ的文件名进行对应，
![image.png](https://upload-images.jianshu.io/upload_images/7901162-0db61329dded0a87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从而对应{sample}。每一个sample都会进行一个rule程序。

为啥在input处不能直接用{sample}
![image.png](https://upload-images.jianshu.io/upload_images/7901162-f7a1eaa125954c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/7901162-011e52a7a002bf10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### doc
>If the rule’s output matches a requested file, the substrings matched by the wildcards are propagated to the input files and to the variable wildcards, that is here also used in the shell command. The wildcards object can be accessed in the same way as input and output, which is described above.
rule的output匹配某一个需要的文件（在rull all的input中写明），匹配的子字符串会应用到input文件以及这个变量wildcards{sample}上。

![image.png](https://upload-images.jianshu.io/upload_images/7901162-9c80677eb33bb32c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据output文件匹配得到group和dataset值，然后应用到input和shell。注意input可以直接{group},而shell需要{wildcards.group}
## snakemake 命令

temp: 通过temp方法可以在所有rule运行完后删除指定的中间文件，eg.output: temp(“f1.bam”)。

protected: 用来指定某些中间文件是需要保留的，eg.output: protected(“f1.bam”)。

expand : 相当于列表推导式
![](https://upload-images.jianshu.io/upload_images/7901162-9de2cd7bf063a65b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 流程考虑
1. 路径全部为绝对路径。OUT_DIR和子目录名写在文档最前面。rule中全部引用。

#### 目前流程出现问题
1. circos 在res节点报错：缺少GD模块
在mu01节点可以正常出图
2. 每一步step跑之前将原文件都删除 再重新跑
3. 第一步包含QC和bwa比对 所以如果要QC 需要选择step1并加-QC 
4. bwa 比对：使用00preprare的fastq会报错。但使用原始的fastq.gz不会报错 https://www.biostars.org/p/254155/
    `[mem_sam_pe] paired reads have different names: "ST-E00600:400:HKT7FALXX:2:2102:32298:30937", "ST-E00600:400:HKT7FA
LXX:2:2102:9218:31548"`
    目前流程用原始fastq gz
5. breakdancer 在res 节点报错 GD。其他节点都可以跑。提交到rna跑成功了。

## 总结
#### wildcards {sample}
第一条rule里input，{sample}还没有得到，因此要定义一个匿名函数，wildcards来推迟。等sample被匹配到之后，再来确定input。

shell中不能直接使用{sample}，而要用{wildcards.sample}来获取sample值。

需要分sample跑的rule，就一定要用到{sample}。
#### snakemake逻辑
snakemake根据rule中的输入文件来设定job顺序。如果输入文件已经得到，这个rule就会开始跑。因此rule的输入文件需要将前rule得到的核心文件指定。

snakemake先读取rule all的全部文件，然后来判定要跑那些rule。将文件与rule的输出文件匹配，匹配到了的就跑该rule，因此如果rule all中没有写的，某个rule就完全被忽略了。

参数设定`-R 某rule`后，snakemake会跑自己认为会跑的step+某rule

## 集群
`snakemake -j 99 --debug --immediate-submit --cluster-config cluster.json --cluster 'bsub_cluster.py {dependencies}'`

`snakemake --cluster "qsub -q res -cwd  -o logs -e logs" --jobs 32`
the maximum number of jobs to be queued or executed at the same time
可最大提交32的jobs同时跑。在cpu允许范围内，jobs越大跑的越快

snakemake -j32 --drmaa ' -n {threads} -b n -S /bin/bash'
threads在每个rule的参数中被设定

![image.png](https://upload-images.jianshu.io/upload_images/7901162-538dd38b82433a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 报错
1. 
![image.png](https://upload-images.jianshu.io/upload_images/7901162-c387b207195d29e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
检查rule的output和rule all的input是否一一对应

2.



## 脚本学习
#### samples获取
黑色部分为snakefile内容。
samples.json文件 写入样本信息
![image.png](https://upload-images.jianshu.io/upload_images/7901162-8123f31c245388c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在snakefile中读入json。得到的是一个字典
![image.png](https://upload-images.jianshu.io/upload_images/7901162-1830e7c0900317aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/7901162-ee38da1c2545f2c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/7901162-942c5cac7b8613eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
ALL_SAMPLES得到列表 ['A','B']
ALL_FASTQ的expand为snakemake中的用法。这里相当于得到
![image.png](https://upload-images.jianshu.io/upload_images/7901162-daa54f122b67f823.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### shell脚本和python结合
```
rule kallisto_quant:
    input:
        r1 = lambda wildcards: FILES[wildcards.sample]['R1'],
        r2 = lambda wildcards: FILES[wildcards.sample]['R2'],
        index = rules.kallisto_index.output.index
    output:
        join(OUT_DIR, '{sample}', 'abundance.tsv'),
        join(OUT_DIR, '{sample}', 'run_info.json')
    version:
        KALLISTO_VERSION
    threads:
        4
    resources:
        mem = 4000
    run:
        fastqs = ' '.join(chain.from_iterable(zip(input.r1, input.r2)))
        shell('kallisto quant'
              ' --threads={threads}'
              ' --index={input.index}'
              ' --output-dir=' + join(OUT_DIR, '{wildcards.sample}') +
              ' ' + fastqs)
```

## 参考
1. [生信分析流程构建的几大流派](https://life2cloud.com/cn/2018/11/pipelines-styles/)
2. [snakemake框架](https://github.com/inodb/snakemake-workflows)
3. [snakemake pyflow-ATACseq流程](https://github.com/crazyhottommy/pyflow-ATACseq)
4. [https://github.com/AAFC-BICoE/snakemake-mothur](https://github.com/AAFC-BICoE/snakemake-mothur)
5. [使用GATK和snakemake框架的总结](https://qizhengyang2017.github.io/2019/02/20/snakemake/)
6. [yaml语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)
7. [snakemake 官方文档](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html)
8. [https://github.com/inodb/snakemake-parallel-bwa/blob/master/Snakefile](https://github.com/inodb/snakemake-parallel-bwa/blob/master/Snakefile)

## 目录
[TOC]