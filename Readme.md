# MAGE：元奠基群辅助基因组估计
多群体基因组加性 - 显性亲缘关系矩阵构建软件 V1.0 用户手册
## 一、软件介绍
本软件是一款面向畜禽群体的基因组亲缘关系矩阵计算工具，可对多个无亲缘关系群体及其杂交后代的亲缘关系进行整合计算。
软件能建立无亲缘群体之间、群体与杂交后代之间的遗传关联，基于加性 - 显性效应设计，可同时计算加性和显性亲缘关系矩阵。
计算结果可导入 DMU、PIBLUP 等软件，帮助提升遗传评估的准确性；支持综合利用多群体信息，从而显著提高育种选择效率与行业整体生产效率。
## 二、安装说明
程序基于 CentOS Linux release 7.9.2003 平台开发，依赖 3.10.0-1127.19.1.el7.x86_64 内核版本，可在任意类 Unix 系统运行。
由于程序主要采用 C 语言开发，重新编译后也可在 Windows 或 macOS 系统运行。
### 硬件要求
常规亲缘关系计算需保证系统内存≥10 GiB；
大规模群体需根据群体规模增加内存；
arm64 系统下，最多支持 2,147,483,647 条系谱记录、65,535 条基因组数据记录（需更多数据可修改代码后重新编译）。
### 依赖环境
科学计算依赖 Intel® oneAPI Math Kernel Library（无版本要求）；
依赖 Python 3.6 及以上版本；
编译依赖 python3-devel 库，安装命令：
bash
#### Debian/Ubuntu系统
apt-get depends python2-dev
apt-get depends python3-dev

#### CentOS/RHEL系统
yum install python2-devel
yum install python3-devel
若 Python 为完整编译安装，该依赖已默认存在；若需使用完整编译的 Python 依赖库，需确保 Python 版本≤3.9。
## 三、快速入门
除非另有说明，以下操作均在 Linux 命令行模式下执行。
### 1. 安装步骤
软件提供预编译二进制文件，可直接下载使用：
bash

tar zxvf mage.tar.gz  # 解压压缩包

cd mage               # 进入解压目录

chmod 755 ./bin/mage  # 赋予可执行权限

### 2. 运行软件
在命令行输入以下命令运行：
bash

mage --pedigree=文件路径 --gene=文件路径 [其他选项] ...

--pedigree：指定系谱文件（必填），支持相对 / 绝对路径；

--gene：指定基因组文件（必填），支持相对 / 绝对路径。

### 3. 输入文件要求
格式：ASCII 格式，以空格或制表符为分隔符；
换行：DOS/UNIX 格式均可，推荐 UNIX 格式以保证稳定性；
表头：文件不能包含表头；
详细格式见「四、文件说明」章节。
### 4. 其他参数查询
通过以下命令查看所有参数：
bash

mage -h

参数详情：
参数	说明

-p, --pedigree	系谱文件路径（字符串），支持相对 / 绝对路径

-g, --gene	基因组文件路径（字符串），支持相对 / 绝对路径

-o, --output	输出文件前缀（默认：output），不允许包含路径

-O, --out-dir	输出目录（默认：./output），支持相对 / 绝对路径

-A, --AMatrix	额外输出系谱亲缘关系矩阵（默认关闭）

-G, --GMatrix	额外输出基因组亲缘关系矩阵（默认关闭）

--add	仅计算加性亲缘关系矩阵（与 --dom 冲突）
--dom	仅计算显性亲缘关系矩阵（与 --add 冲突）

-M, --Matrix	输出原矩阵 / 逆矩阵（选项：original/inverse/all，默认：all）

--MFs	强制使用元奠基群模型

--no-cMFs	强制不使用多品种元奠基群模型

-b, --breed	指定计算的品种（默认：all）

-f, --full	输出全存储格式矩阵（默认：半存储格式）

-a, --all	同时输出全存储 + 半存储格式（覆盖 --full）

-m, --maf	次要等位基因频率（默认：0）

-c, --course	保留计算过程中的中间文件

-v, --version	打印软件版本并退出

-q, --quite	不输出到标准输出（不影响日志）

-d, --debug	输出调试信息

-h, --help	打印帮助信息并退出

## 四、文件说明
相关文件包括系谱文件和基因型文件，均为 ASCII 格式，以 1 个或多个空格（或换行）分隔；分类变量名可包含英文字母和数字；不同文件中的个体编号需一致。
### 1. 系谱文件
用于确定加性遗传关系，需包含 5 列（无表头）：
列名	说明

ID	模型中随机效应（遗传效应）对应的个体编号

Sire ID	父本编号（未知填 0）

Dam ID	母本编号（未知填 0）

SORT	个体顺序（可填出生日 / 世代，或全填 0，软件会自动计算世代）

BREED	品种信息（纯种品种填字母；杂交个体填 0，但父母必须在系谱中，否则会被丢弃）

示例：
plaintext

1 0 0 0 A

2 0 0 0 A

3 1 2 0 A

4 0 0 0 B

5 0 0 0 B

6 4 5 0 B

7 2 6 0 0

8 3 6 0 0

### 2. 基因组文件
用于构建基因组亲缘关系矩阵，每行对应 1 个个体的所有标记基因型：
第 1 列为个体编号（需与系谱一致，系谱中不存在的基因组信息会被丢弃）；
后续列为各标记的基因型编码；
若无系谱但需保留基因组信息，可将个体添加到系谱并将父母设为 0。
### 基因型编码规则：

0/2：纯合子；

1/3/4：杂合子（1 = 来源未知，仅用于纯血个体；3 = 父本提供显性等位基因；4 = 母本提供显性等位基因）；

5：缺失值。

示例：
plaintext

1 12021022012

2 1022120122

3 0210102210

4 0221020212

5 1021210212

6 2131012212

7 3000304202

8 2033202430

### PLINK 格式转换命令：
bash

plink --bfile $1 --recodeA --out $1.q

awk '{$1="";$3="";$4="";$5="";$6="";print $0}' $1.q.raw | sed '1d' | sed "s/NA/5/g" > $1.genotype.txt

paste -d " " <( awk '{print $1}' $1.genotype.txt) <( awk '{$1="";print $0}' $1.genotype.txt | sed  "s/ //g") > $1.geno.txt

### 3. 输出文件
存储格式：默认输出半存储模式（仅对角线 + 下三角矩阵），可通过--full/--all输出全存储模式；
矩阵类型：默认输出亲缘关系矩阵的逆矩阵，可通过--Matrix指定输出原矩阵；
文件格式：每行代表 1 个矩阵元素，格式为IdRow IdCol value（行个体号、列个体号、值）。
文件名规则：<前缀>_<品种>_<矩阵类型>_<存储格式>.txt
前缀：用户通过--output指定（默认：output）；
品种：系谱中的品种信息（或--breed指定）；
存储格式：full（全存储）/half（半存储）；
矩阵类型（部分）：

aprm：系谱加性亲缘关系矩阵

apim：系谱加性亲缘关系矩阵的逆矩阵

agrm：基因组加性亲缘关系矩阵

ahim：加性 H 矩阵的逆矩阵

dprm：系谱显性亲缘关系矩阵

dhim：显性 H 矩阵的逆矩阵

示例：
plaintext

1 1 1.5

2 1 0.5

2 2 3.64235

3 1 -1

3 2 -2.39239

3 3 3.84481

## 五、典型示例
示例数据集可在example文件夹中找到。
### 1. 默认参数计算
bash

mage --pedigree ped.dat --gene gene.dat
输出：

output_{品种}_ahim_half.txt（加性 H 矩阵的逆矩阵）

output_{品种}_dhim_half.txt（显性 H 矩阵的逆矩阵）
### 2.修改输出路径与前缀
bash

mage --pedigree ped.dat --gene gene.dat --output test --out-dir ~/test

输出文件前缀为test，存储在~/test目录下。

### 3. 额外输出系谱 / 基因组矩阵
bash

mage --pedigree ped.dat --gene gene.dat -A -G

输出 6 类文件（系谱 / 基因组的加性 / 显性矩阵及逆矩阵）。

### 4. 仅计算加性亲缘关系
bash

mage --pedigree ped.dat --gene gene.dat --add

仅输出output_{品种}_ahim_half.txt。

### 5. 仅计算特定品种
bash

mage --pedigree ped.dat --gene gene.dat --breed A

输出品种 A 的加性 / 显性 H 矩阵逆矩阵。

### 6. 额外输出原矩阵
bash

mage --pedigree ped.dat --gene gene.dat --Matrix all

输出原矩阵 + 逆矩阵（共 4 类文件）。

### 7. 全存储模式输出
bash

mage --pedigree ped.dat --gene gene.dat --full

输出全存储格式的加性 / 显性 H 矩阵逆矩阵。
### 8. 输出所有可能结果
bash
mage --pedigree ped.dat --gene gene.dat -A -G --Matrix all --all
输出 24 类结果文件（具体不展开）。
