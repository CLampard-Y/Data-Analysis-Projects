
# FIFA 21 球员数据分析项目 (Data Analysis Project)

## 📌 项目背景
这是一个基于 FIFA 21 球员数据集的数据清洗与分析项目。原始数据包含大量非结构化文本、混合单位和格式错误。
本项目的目标是利用 Excel 和 Power Query 将脏数据转换为可用于分析的标准格式,随后在此基础上对数据进行详细分析。

---

## 🧹数据清洗
### 数据类型定义与架构标准化
原始数据集缺乏统一的架构规范（Schema）。日期字段存储为文本，列命名风格不统一（混合了空格、特殊符号与缩写），这极大地增加了后续使用 SQL 或 Python 进行分析时的引用成本。
**对策 (Engineering Strategies)**：
1. **时序数据解析** :将 `Joined` 等时间字段强制转换为标准 `Date` 格式，支持时间序列分析。
2. **蛇形命名重构** :为了符合数据工程的最佳实践，将所有列名重构为对代码友好的格式。
	(包括但不限于以下列,后续对特定列的清洗中也会进行相应的重命名)
	- `↓OVA`(原数据是按照OVA列降序排列) $\rightarrow$ `Overall_Rating`
    - `Preferred Foot` $\rightarrow$ `Preferred_Foot`
    - `W/F` $\rightarrow$ `weak_foot_rate`
    - `OVA` $\rightarrow$ `overall_rating`
    - `PAC` $\rightarrow$ `Pace`
    - `SHO` $\rightarrow$ `Shooting`
    - `PAS` $\rightarrow$ `Passing`
    - `DRI` $\rightarrow$ `Dribbling`
    - `DEF` $\rightarrow$ `Defending`
    - `PHY` $\rightarrow$ `Physical`

### 身高与体重字段的标准化 (The Dual-Validation Approach)
在该数据集中，`Height` (身高) 和 `Weight` (体重) 列呈现出高度的**非结构化特征**。例如：英制单位 (`5'7"` / `127lbs`) 与公制单位 (`170cm` / `72kg`) 在同一列中深度混合，且包含非数值字符。

							身高,体重脏数据(节选)
<img src="https://raw.githubusercontent.com/CLampard-Y/Data-Analysis-Projects/refs/heads/main/01_FIFA21_Data_Analysis/assets/Dirty_Data(Height%26Weight).png"  />

为了确保清洗逻辑的绝对准确性（Robustness）并展示不同工具在处理脏数据时的优势，本项目构建了**“双重交叉验证系统”**，分别使用 **Excel 原生函数** 与 **Power Query (M语言)** 两种截然不同的方法进行清洗与比对。
#### 方法一：Excel 原生函数 (逻辑原型与快速验证)
在探索性数据分析 (EDA) 初期，我首先使用 Excel 嵌套函数构建了清洗逻辑。这种方法透明度高，便于对单行数据进行快速逻辑校验。

**身高清洗(处理 5'7" 与 170cm 混合格式):** 利用 `SEARCH` 定位，结合 `LEFT` 函数提取数值并进行通过`CONVERT`函数单位换算。
```excel
=IF(ISNUMBER(FIND("'",[@Height])),CONVERT(LEFT([@Height],FIND("'",[@Height])-1),"ft","cm")*1+CONVERT(MID([@Height],FIND("'",[@Height])+1,LEN([@Height])-FIND("'",[@Height])-1),"in","cm")*1,SUBSTITUTE([@Height],"cm","")*1)
```

**体重清洗 (处理 183lbs 与 102kg 混合格式):** 
```excel
=IF(ISNUMBER(FIND("lbs",[@Weight])),CONVERT(LEFT([@Weight],FIND("l",[@Weight])-1),"lbm","kg")*1,SUBSTITUTE([@Weight],"kg","")*1)
```

#### 方法二：Power Query / M 语言 (工业级自动化加工)
为了实现数据处理的**可重复性 (Reproducibility)** 与**自动化**，我在生产环境中使用 Power Query 构建了清洗流水线。相比函数法，M 语言更适合处理百万级数据，且易于维护。

**身高清洗 M 代码 (Advanced Editor):** 通过 `Text.Contains` 定位，结合 `Text.BeforeDelimiter` 以及`Text.Select`提取数值(此时仍为文本)并进行通过`Number.From`换算为数值,最后进行对应的单位换算
```
if Text.Contains([Height],"'") then
	Number.From(Text.BeforeDelimiter([Height],"'"))*30.48+
	Number.From(Text.Select(Text.AfterDelimiter([Height],"'"),{"0".."9","."}))*2.54
else
	Number.From(Text.Select([Height],{"0".."9","."}))
```

**体重清洗 M 代码 (Advanced Editor):** 
```
if Text.Contains([Weight],"lbs") then
	Number.From(Text.Select([Weight],{"0".."9","."}))/2.2046
else
	Number.From(Text.Select([Weight],{"0".."9","."}))
```

#### ✅ 清洗结果与交叉验证 (Audit Results)
经过上述两种方法的独立处理，最终生成的清洗字段（`Height_cm`, `Weight_kg`）经比对**完全一致**。这证明了清洗逻辑的数学严谨性，同时消除了单位混用导致的数据分布异常。


### 身价、薪资与解约金标准化 (The Dual-Validation Approach)
在该数据集中，`Value` (身价) 、 `Wage` (薪资)与`Release Clause`(解约金) 列呈现出高度的**非结构化特征**：薪资单位(`€103.5M`/`€275K`/`€900`) 在同一列中深度混合，且包含非数值字符。

							身价,薪资,解约金脏数据(节选)
<img src="https://raw.githubusercontent.com/CLampard-Y/Data-Analysis-Projects/refs/heads/main/01_FIFA21_Data_Analysis/assets/Dirty_Data(Height%26Weight).png"  />

同身高体重的清洗思路类似,仍通过使用 **Excel 原生函数** 与 **Power Query (M语言)** 构建了**“双重交叉验证系统”,以下仅列出相关代码,不进一步详细解释。

**Excel代码(以Value为例):** 
```excel
=IF(SEARCH("M",[@Value]),SUBSTITUTE(SUBSTITUTE([@Value],"€",""),"M","")*1000000,IF(SEARCH("K",[@Value]),SUBSTITUTE(SUBSTITUTE([@Value],"€",""),"K","")*1000,SUBSTITUTE([@Value],"€","")*1))
```

**Power Query代码(以Value为例)**
```
if Text.Contains([Height],"'") then
	Number.From(Text.BeforeDelimiter([Height],"'"))*30.48+
	Number.From(Text.Select(Text.AfterDelimiter([Height],"'"),{"0".."9","."}))*2.54
else
	Number.From(Text.Select([Height],{"0".."9","."}))
```

### 合同信息清洗
`Contract` (合同)列是本数据集中结构最复杂的字段，严重违反了数据库设计的“原子性原则”。它在同一列中混合了三种完全不同的业务逻辑，导致无法直接进行定量分析：

1. **时间区间**：如 `2019 ~ 2024`（大多数球员）。
    
2. **特定日期**：如 `Jun 30, 2021 On Loan`（租借球员，包含归队日期）。
    
3. **文本标签**：如 `Free`（自由球员）。

							合同脏数据(节选)
<img src="https://raw.githubusercontent.com/CLampard-Y/Data-Analysis-Projects/refs/heads/main/01_FIFA21_Data_Analysis/assets/Dirty_Data(Height%26Weight).png"  />

为了将非结构化文本转化为可计算的商业指标（如“剩余合同年限”），本项目采用了**基于规则的分层解析 (Rule-Based Parsing)** 策略，将单一字段解耦为三个独立特征：`Contract_Start`(合同开始年份)、`Contract_End`(合同结束年份)和 `Contract_Length`(合同年限)。

#### 1. 状态分类与清洗流水线 (Power Query )

考虑到 Excel 公式在处理多重嵌套逻辑时的可读性较差，本项目在生产环节使用 Power Query 构建了标准化的 ETL 处理管道：

- 步骤 A：状态标记 (Tagging)
    
    利用条件列 (Conditional Column) 逻辑，优先识别并标记特殊状态：
    
    - `if [Contract] contains "Free" then "Free"`
        
    - `if [Contract] contains "Loan" then "Loan"`
        
    - `else "Active Contract"`
        
						Power Query条件列设置
    img src="Contract_set1" wid="" /

- 步骤 B：文本拆分与清洗
    
    针对标记为 Active Contract 的数据，使用分隔符 ~ 将年份拆分为起始与结束两列。
    
- 步骤 C：异常值处理
    
    对于 Free 和 Loan 状态的行，将其具体的年份字段置为 null ，确保后续计算平均合同时长时不被异常日期（如租借归队日期）干扰。
    

#### 2. 特征构造 (Feature Construction)

在完成清洗后，通过数学运算构造核心分析指标：

$$Contract\_Length = End\_Year - Start\_Year$$

该指标直接赋能了后续关于**“球队稳定性分析”**与**“转会窗口预测”**的业务洞察。

#### ✅ 清洗结果 (Final Dataset Structure)

经过处理，原始的混乱文本列被成功转化为结构化的数值特征：

> 📍 [埋点 3]：在此处插入“清洗后的最终效果图”
> 
> 建议内容：截图数据表，展示紧挨着的四列：Contract (原列), Contract_Start, Contract_End, Contract_Length。
> 
> 目的：展示从“混乱”到“有序”的完美对比。

							经过清洗后的合同列数据
img src="Cleaned_Data(Contract)",width="" /

### 评级数据的数值化
球员的评级指标`W/F`(Weak Foot弱足能力), `SM`(Skill Move花式技巧), `IR`(International Reputation国际声誉)在原始数据中存储为非结构化文本格式（如 `4 ★` 或 `5 ★`）。 除了可见的特殊符号外，数据中还夹杂着大量**隐性空白符**，这会导致直接的类型转换失败或数据对齐错误。

								评价指标脏数据
img src="" width=""

为了彻底清洗脏数据并规避类型转换风险，本项目在 ETL 流程中实施了标准化的**RTC 清洗流 (Replace-Trim-Convert)**：

1. **字符剥离 (Replace)**：利用 Power Query 的批量替换功能，移除冗余的 Unicode 字符 (`★`)。
2. **空白修整 (Trim)**：**关键步骤**。调用 `Text.Trim()` 函数彻底清除数值前后的空格。这一步确保了数据在进入数据库或进行数学计算前的绝对干净，防止了因空格导致的匹配错误。
3. **类型强制 (Cast)**：将清洗后的纯净文本强制转换为 `Int64` (整数) 类型。
4. **评价列重命名(Rename)**:为使得评价数据列更具有可读性,将`W/F`,`SM`及`IR`分别重命名为`Weak_Foot`,`Skill_Move`及`International_Reputattion`

**清洗结果与业务价值 (Impact)**： 转化后的整数数据直接支持了后续的多维相关性分析。

- **应用场景**：通过计算 `Skill_Moves`（花式技巧）与 `Value`（身价）的皮尔逊相关系数，我们量化了“球星效应”带来的市场溢价。

								清洗后评价列数据
img src="" width="" /

### 热度指标的单位归一化与异常值防御
`Hits` (热度) 列是数据清洗中典型的“数学陷阱”列，存在双重挑战：
1. **量纲混乱 (Unit Inconsistency)**：数据混合了普通整数（如 `372`）与“千分位缩写”文本（如 `1.6K`）。直接转换会导致 `1.6K` 变为 `1.6`，引发严重的**数量级坍塌 **。
2. **空值陷阱 (The Null Trap)**：该列中不仅存在标准的数据库 `null` 值，还混杂了长度为 0 的空字符串 `""`。简单的逻辑判断极易在处理空字符串时报错或返回错误结果。

							热度列的脏数据
img src="" width="" /

为了构建健壮的清洗流水线，本项目拒绝使用简单的查找替换，而是基于 **Power Query (M 语言)** 编写了具备**防御性逻辑 (Defensive Logic)** 的条件计算公式：
1. **异常拦截 (Guard Clause)**：首先构建双重逻辑门，同时拦截 `null` 和空文本 `""`，将其统一定义为 `0`（代表无热度），防止后续计算报错。
2. **单位换算 (Unit Conversion)**：利用 `Text.Contains` 识别千位标记 "K"，提取数值部分并 $\times 1000$。
3. **常规解析 (Parsing)**：对普通数字进行标准类型转换。

**核心代码 (Robust Code Implementation)**：
```
// 逻辑说明：优先处理空值/空文本，再处理单位换算，最后处理普通数值
if [Hits] = null or [Hits] = "" then 
    0 
else if Text.Contains([Hits], "K") then 
    Number.From(Text.BeforeDelimiter([Hits], "K")) * 1000 
else 
    Number.From([Hits])
```

通过上述逻辑，成功将离散的混合文本转化为**连续的数值变量**，并完美解决了空值填充问题。
- **数据验证**：`1.6K` 被正确转化为 `1600`；所有空白单元格被安全地填充为 `0`，消除了后续聚合计算（如平均热度）中的分母偏差风险。

								清理后的热度列
img src="" width="" /

### 文本数据的规范化与噪声剔除
`Club` (俱乐部) 列存在严重的**格式噪声**。原始数据中，俱乐部名称前后夹杂了大量不可见的**控制字符 (Control Characters)**，主要是换行符 (`\n`) 和冗余空白(`" "`)。 这种噪声会导致数据可视化时的标签错位，以及在进行 `Group By` 聚合分析时出现无法匹配的问题。

								清洗前的俱乐部列
img src="" width="" /

为了实现文本的标准化存储，本项目使用了 Power Query 的**双重清洗指令**：
1. **清除 (Clean)**：调用 `Text.Clean()` 函数，移除所有非打印字符（如 Line Feeds, Carriage Returns）。
2. **修整 (Trim)**：调用 `Text.Trim()` 函数，剥离字符串首尾的残留空格。
**清洗结果**： 确保了所有分类变量（Categorical Variables）的字符串一致性，为后续按俱乐部维度的薪资结构分析打下基础。

								清洗后的俱乐部列
img src="" width="" /

### 球员投入度特征提取
原始数据集中的 `A/W` (Attacking Work Rate) 和 `D/W` (Defensive Work Rate) 列存在两个工程隐患：
1. **命名违规**：列名中包含特殊符号 `/` (Slash)，这在 SQL 查询或 Python 属性访问中属于非法字符，增加了代码引用的复杂度。
2. **定序文本**：数据存储为 `High`/`Medium`/`Low` 文本格式，无法直接参与加权计算或相关性分析。

为了提升数据集的**编程友好度 (Code-Friendliness)** 与分析能力，本项目实施了以下处理：
1. 元数据重构 (Schema Refactoring)：
    将列名严格遵循蛇形命名法 (Snake Case) 进行重命名，消除特殊符号干扰：
    - `A/W` $\rightarrow$ `Attacking_Work_Rate`
    - `D/W` $\rightarrow$ `Defensive_Work_Rate`
2. 定序变量编码 (Ordinal Encoding)：
    将文本评级映射为数值权重，以便量化球员的战术执行力：
    - Mapping Strategy: `{High: 3, Medium: 2, Low: 1}`
    - 转换后的数值可直接用于计算“攻防投入比”等高阶指标。








![Pic Test](https://raw.githubusercontent.com/CLampard-Y/Data-Analysis-Projects/refs/heads/main/01_FIFA21_Data_Analysis/assets/QQ%E5%9B%BE%E7%89%8720260103171607.jpg)


<img src="https://raw.githubusercontent.com/CLampard-Y/Data-Analysis-Projects/refs/heads/main/01_FIFA21_Data_Analysis/assets/QQ%E5%9B%BE%E7%89%8720260103171607.jpg" width="300" />