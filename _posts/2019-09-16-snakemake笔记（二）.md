---
layout:     post 
title:     snakemake笔记（二）		
date:       2019-09-16	
author:     琼脂糖					
header-img: img/post-bg-2015.jpg 
catalog: true 				
tags:				
    - snakemake
---
## 常问问题
**snakemake 原理**
![](http://pxlp1m31j.bkt.clouddn.com/mweb/15685961780014.jpg)

**建议的框架**
```
├── config.yaml
├── environment.yaml
├── scripts
│   ├── script1.py
│   └── script2.R
└── Snakefile
```
**修改代码后，重跑改变的rule**
```python
snakemake -n -R `snakemake --list-input-changes`
snakemake -n -R `snakemake --list-params-changes`
snakemake -n -R `snakemake --list-code-changes`
```
        
**删除snamake 输出的文件** 。
仅删除rule在output中写的文件。最好先和`--dry-run `跑确认一下。同样的还有`--delete-temp-output`

```
 snakemake some_target --delete-all-output
```
例子

```python
snakemake index --delete-all-output -np --snakefile res-snake.py
```
```bash
Building DAG of jobs...
Would delete /02snakeReq/00.prepare/ref/salmonella.dict
Would delete /02snakeReq/00.prepare/ref/salmonella.fa
```

**workflow太大了，不看output，只看最终结果**
`snakemake -n --quiet`

打开zsh关于snakemake的自动完成
将以下代码放在~/.zshrc中
`compdef _gnu_generic snakemake`