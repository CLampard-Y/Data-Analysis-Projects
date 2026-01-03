
# FIFA 21 球员数据清洗及分析项目 (Data Analysis Project)

## 📌 项目背景
这是一个基于 FIFA 21 球员数据集的数据清洗与分析项目。原始数据包含大量非结构化文本、混合单位和格式错误。
本项目的目标是利用 Excel 和 Power Query 将脏数据转换为可用于分析的标准格式。

---

## 🧹数据清洗



## 🧹 数据清洗策略：身高与体重 (双重验证法)

为了确保数据的准确性并展示不同工具在处理脏数据时的优势，本项目采用了 **Excel 原生函数** 与 **Power Query (M语言)** 两种截然不同的方法对 `Height` (身高) 和 `Weight` (体重) 列进行了清洗与交叉验证。

### 方法一：Excel 原生函数 (逻辑原型与快速验证)
在探索性数据分析 (EDA) 初期，我首先使用 Excel 嵌套函数构建了清洗逻辑。这种方法透明度高，便于对单行数据进行快速逻辑校验。

**身高清洗公式 (处理 5'7" 与 170cm 混合格式):**
```excel
=IF(ISNUMBER(SEARCH("'",[@Height])),
   LEFT([@Height],FIND("'",[@Height])-1)*30.48 + MID([@Height],FIND("'",[@Height])+1,LEN([@Height])-FIND("'",[@Height])-1)*2.54,
   LEFT([@Height],LEN([@Height])-2))
```
方法二：Power Query / M 语言 (生产级自动化 ETL)
为了构建可复用、可扩展的数据管道 (Pipeline)，我将清洗逻辑迁移至 Power Query。相比普通函数，M 语言在处理大数据量和空值异常时更具鲁棒性。

身高清洗 M 代码 (Advanced Editor):

代码段

if Text.Contains([Height], "'") then 
    Number.From(Text.BeforeDelimiter([Height], "'")) * 30.48 + 
    Number.From(Text.Select(Text.AfterDelimiter([Height], "'"), {"0".."9", "."})) * 2.54 
else 
    Number.From(Text.Select([Height], {"0".."9", "."}))
Power Query 方案的优化点：

代码可读性：使用 Text.BeforeDelimiter 代替复杂的索引计算，逻辑一目了然。

异常处理：引入了对 null 值的预处理逻辑，防止整个查询因空行而报错中断。

✅ 结果验证
经过对比：[Height_Excel] == [Height_PQ]，两种方法计算结果完全一致。