# Alberta_AER-Analysis
Monthly Data from AB AER Analysis for piplines



## 一、先看最重要的：管道“是谁、是哪一条”
### Licence_Number
```
AER 管道许可证号。
```
可以理解为：
```
这条管道在 AER 系统中的 licence。
```
例如：
```
Licence_Number = 123456
```
同一个 licence 下面可以有多个 segment。
```
NEB_Pipeline_Indicator
```
表示这条管道是否属于当时的 NEB（National Energy Board） 管辖/所有的 80000 系列管道。<br>

一般你做 Calgary 周边普通 Alberta pipeline 筛选时，不是第一优先字段。
```
Segment_Line_Number
```
这是：
```
同一个 pipeline licence 下，某个具有相同 engineering specifications 的管道 segment 编号。
```
例如：
```
Licence 123456

Segment 1
Segment 2
Segment 3
```
AER 特别强调它是 segment，不是整个 pipeline。
```
Licence_Line_Number
```
实际上是：
```
Licence_Number + Segment_Line_Number
```
组成一个具体的 line segment 标识。<br>

所以你可以把它看成：<br>

人比较容易理解的 segment ID。
```
Pipeline_Licence_Segment_Id
```
这个是我非常建议你保留的字段。<br>

它是 AER 给这个 pipeline segment 的唯一 surrogate ID。<br>

为什么不直接用 Licence_Number + Segment_Line_Number？<br>

因为如果 pipeline licence 转让给另一家公司：
```
Company A
   ↓
licence transfer
   ↓
Company B
```
licence number / line number 可能发生变化，但这个 ID 用来追踪同一个 segment。<br>

所以数据库主键优先：
```
Pipeline_Licence_Segment_Id
```
## 二、公司是谁
### Company_Name

这个非常重要。<br>

AER 定义是：
```
business associate 的名称，通常是 legal name。
```
注意：<br>

它不是一定等于“天然气生产商”。<br>

它更接近：<br>

这条 pipeline 在 AER 数据中的 business associate / licensee。<br>

例如你可能看到：
```
Company_Name
TC Energy
Pembina
Enbridge
某 Gas Ltd.
```
但不能直接推断：<br>

“这个公司生产这些天然气。”<br>

它首先说明的是pipeline licence 对应的公司关系。
```
BA_Code
```
这是 Petrinex Business Associate Code。<br>

这个非常适合做数据库 join。<br>

例如：
```
BA_Code = A123
```
你以后可以拿这个 code 去连接其他 Petrinex / AER 数据。<br>

对你的项目：<br>

Company_Name = 给人看的<br>

BA_Code = 给数据库 join 用的<br>

## 三、这是你最关心的：管道到底输什么气
### Substance 1

这个字段非常重要。<br>

它告诉你：<br>

这个 pipeline segment 主要处理/输送什么 substance。<br>

AER 对它有明确的优先级：<br>

如果一条管道可以输送多个 substance，Substance_1 会放优先级最高的 substance。<br>

你会看到例如：
```
Natural gas
Sour Natural Gas
Crude oil
Oil well effluent
Salt water
HVP Products
LVP Products
Fuel gas
Fresh water
Miscellaneous gases
Miscellaneous liquids
```
对你的项目，首先筛：
```
Substance 1 = Natural gas
```
以及：
```
Substance 1 = Sour Natural Gas
```
然后分别分析。<br>

AER 的定义是：
```
Natural Gas

H₂S ≤ 10 mol/kmol

Sour Natural Gas

H₂S > 10 mol/kmol。
```
所以这个字段可以作为第一轮天然气筛选。

## 四、Substance 2 和 Substance 3

就是第二、第三种 substance。<br>

例如一条管道可能：
```
Substance 1 = Natural gas
Substance 2 = Fuel gas
Substance 3 = ...
```
不过对于你的项目：
```
优先级
Substance_1
   ↓
Substance_2
   ↓
Substance_3
```
不用一开始就太纠结后两个。

## 五、你特别关心的“硫”：这里有两个完全不同的字段

这个非常重要。
```
H2S_Content
```
这是：<br>

管道中 substance 的 H₂S 含量<br>

单位是：<br>

mol/kmol<br>

AER 对这个字段的定义非常明确。<br>

这个是你应该重点看的。<br>
```
例如：

H2S_Content = 0.5

相比：

H2S_Content = 8
```
前者明显更符合你“低硫”的目标。

## 六、不要把 H2S_Release_Volume 当成气质

这是非常容易搞错的。
```
H2S_Release_Volume
```
不是：<br>

**天然气里面有多少 H₂S**<br>

而是：<br>

**如果这条管道发生 fluid release，理论上可能释放到大气中的 H₂S 体积。**<br>

单位是：<br>

m³<br>

所以：<br>

研究气质<br>

看：<br>

**H2S_Content** <br>

研究事故风险/应急<br>

看：<br>

**H2S_Release_Volume** <br>

这是两个完全不同的东西。

## 七、H2S_Release_Level

这是根据 H₂S release volume 对应的安全 setback / release level 分类。<br>

它更偏向：
```
安全风险 / 管道规划 / setback
```
而不是天然气商品质量。<br>

所以你现在找“好气”：<br>

优先级低。<br>

## 八、管道“从哪里到哪里”

这几个字段对你现在的项目非常重要。
```
Segment_From_Facility
```
表示：<br>

这个 pipeline segment 起点连接的 facility 类型。
```
Segment_To_Facility
```
表示：<br>

这个 pipeline segment 终点连接的 facility 类型。<br>

例如你可能得到类似：
```
From Facility       To Facility

Gas Plant            Compressor
Compressor           Meter Station
Meter Station        Gas Plant
Battery              Gas Plant
```
这就非常有用了。

## 九、From_Location / To_Location

这两个不是经纬度。<br>

这是：<br>

Legal Subdivision（LSD）相关的位置编号<br>

也就是 AER 的土地位置体系。<br>

AER 定义为：
```
uniquely identifies the legal subdivision where the starting/ending point is located.
```
所以：<br>

不要把它当 latitude / longitude。<br>

如果你想真正画地图，要用 Shapefile 的 geometry。<br>

## 十、Segment_Length

单位：<br>

km<br>

这个对你以后算：<br>

“哪个公司控制/运营最多天然气管道？”<br>

非常有用。<br>

例如：
```
Company A
Natural Gas
Operating
Segment_Length
= 1,250 km
```
你可以按公司 aggregate：
```
Company
    ↓
Natural Gas segments
    ↓
SUM(Segment_Length)
```
得到每家公司天然气管道长度。<br>

但注意：不能简单把所有 segment length 加起来就当成独立管网长度，因为 licence/segment 的历史或空间关系可能导致重复或拆分，需要进一步处理。

## 十一、Segment_Status

这个是你一定要筛的。<br>

主要状态包括：<br>

### **Operating**

最重要。<br>

AER 定义为：<br>

**approved + constructed + licensed + carrying substance**<br>

也就是实际运行中的管道。<br>

### **Permitted**

已经批准建设，但：<br>

不一定已经建成 / 不一定在运行。<br>

所以如果你要找现在可以供气的管道：<br>

❌ 不要把 Permitted 当 Operating。<br>

### **Abandoned**

永久停用。

### **Discontinued**

暂时停用。

### **Removed**

准备拆除/已经进入 removal 状态。<br>

所以你的第一轮筛选：<br>

Segment_Status = Operating

## 十二、管径：Pipe_Outside_Diameter

单位：<br>

mm<br>

这个字段对你寻找“靠谱供气点”其实有一定价值。<br>

一般来说：<br>

管径大 → 往往更可能是较重要的输送线路<br>

但是：<br>

不能用管径直接推断可供气量。<br>

因为真正的 capacity 还受：
```
pressure
compressor
upstream supply
downstream demand
linepack
tariff
contractual capacity
```
影响。

## 十三、Pipe_Max_Operating_Pressure

这个对你特别有用。<br>

它是：<br>

管道允许的最大运行压力。<br>

如果你的最终需求是：<br>

Calgary 工厂需要比较高压力的天然气<br>

这个字段就值得关注。<br>

但是：<br>

最大允许压力 ≠ 当前实际压力。<br>

这个区别非常重要。

## 十四、Pipe_Wall_Thickness

管壁厚度，单位 mm。<br>

偏工程 / 安全属性。<br>

对于你第一轮找便宜天然气：<br>

基本不用看。

## 十五、Pipe_Type

管材制造标准。<br>

可能涉及：
```
CSA
API
ASTM
```
这是工程数据。

## 十六、Pipe_Grade

钢材等级 / alloy / compound specification。<br>

也是工程属性。<br>

第一轮找气源：<br>

不用重点看。

## 十七、Pipe_Material

管道材料。<br>

例如钢材等。<br>

工程分析有用。<br>

商业选气第一轮：<br>

低优先级。

## 十八、Pipe_Stress_Level

管道工作应力相对于 yield strength 的百分比。<br>

例如：<br>

Stress Level = 50%<br>

这个是：<br>

工程安全/设计参数<br>

不是 gas quality。

## 十九、Pipe_Joint_Method

管道连接方式。<br>

例如焊接等。<br>

工程字段。<br>

## 二十、Pipe_Internal_Protection

管道内部保护方式。<br>

例如内部涂层等。<br>

对于你的项目：<br>

低优先级。

## 二十一、Pipeline_External_Protection

外部防腐/保护方式。<br>

同样是：<br>

pipeline integrity<br>

不是 gas quality。

## 二十二、Pipeline_Environment

表示管道是否穿越：
```
Lake
River
Creek
```
例如：
```
RC = River Crossing
LC = Lake Crossing
CC = Creek Crossing
```
这个主要用于环境/工程/风险分析。

## 二十三、Pipeline_Class_Location

这个很重要但不是你目前最重要的。<br>

它按照 CSA Z662 的 class location，根据：<br>

人口密度 + 地理环境等<br>

对管道所在区域分类。<br>

它主要用于：
```
pipeline design / pressure testing / safety
```
不是气质。

## 二十四、Bidirectional_Pipeline_Ind

这个我建议你保留。<br>

如果：<br>

YES<br>

说明：<br>

管道允许两个方向流动。<br>

对于你寻找供气点，这个可能很有意义。<br>

因为：
```
A ───────── B
```
如果只能：
```
A → B
```
和：
```
A ↔ B
```
商业供气逻辑完全不同。<br>

但它仍然不能证明某个点现在就有可用 capacity。

## 二十五、Field_Centre

AER 管理区域的 field centre。<br>

例如某个 pipeline 归哪个 AER field centre 管理。<br>

主要用于：
```
regulatory / emergency / inspection
```
不是商业供气信息。

## 二十六、HDD_Bored_Ind

HDD = Horizontal Directional Drilling。<br>

表示是否采用水平定向钻/顶管方式穿越某些水体。<br>

对你：<br>

不用看。

## 二十七、Liner_Grade / Liner_Type

如果现有 pipeline 内部安装了独立承压 liner：
```
liner 的 grade
liner type
```
属于管道工程数据。<br>

你的 gas sourcing：<br>

不用优先看。

## 二十八、Licence_Approval_Date

管道 licence 最初批准的时间。

## 二十九、Original_Licence_Number

非常重要的历史追踪字段。<br>

例如：
```
Original Licence 1000
       ↓
transfer
       ↓
Current Licence 5000
```
它帮助你追踪：<br>

这条 pipeline segment 最初属于哪个 licence。

## 三十、Original_Pipe_Specification_Id

原始 pipe specification ID。<br>

同样是历史追踪。

## 三十一、Original_Segment_Line_Number

原来的 segment number。<br>

也是为了追踪 licence transfer。

## 三十二、Original_Licence_Issue_Date

原始 licence / installation 的发证时间。<br>

历史字段。

## 三十三、Permit_Approval_Date

建设许可批准日期。

## 三十四、Permit_Expiry_Date

如果 pipeline 是：
```
Permitted
```
这个字段特别重要。<br>

AER 说明通常在 permit date 后一年左右会转换状态，要求在规定时间内开始建设。<br>

你找现在供气的管道时，可以暂时忽略。

## 三十五、Last_Occurrence_Year

这个名字有点容易误解。<br>

它不是：<br>

最后一次事故年份。<br>

而是：<br>

最后一次 construction / test / status change 的年份。<br>

所以不要拿它当事故数据。

## 三十六、Above_Ground_Pipeline

表示管道是否为地面管道。<br>

通常：
```
YES
```
代表 above ground。<br>

对于天然气采购：<br>

低优先级。

# 最后：你真正应该重点看哪些字段？

如果你的目标还是我们前面说的：<br>

Calgary 周边找低硫、低水、便宜、可实际供气的天然气点<br>

我会把这 44 个字段直接分成：

🔴 第一优先级
```
Licence_Number
Pipeline_Licence_Segment_Id
Company_Name
BA_Code

Segment_Status
Segment_From_Facility
From_Location
Segment_to_Facility
To_Location

H2S_Content
Substance_1
Substance_2
Substance_3

Pipe_Outside_Diameter
Pipe_Max_Operating_Pressure
Segment_Length

Bidirectional_Pipeline_Ind
```

🟡 第二优先级
```
H2S_Release_Volume
H2S_Release_Level
Pipeline_Specification_Id
Field_Centre
Pipeline_Class_Location
Pipeline_Environment
Pipe_Material
Pipe_Grade
```

🟢 第一阶段基本不用
```
Pipe_Wall_Thickness
Pipe_Type
Pipe_Stress_Level
Pipe_Joint_Method
Pipe_Internal_Protection
HDD_Bored_Ind
Liner_Grade
Liner_Type
Pipeline_External_Protection
Above_Ground_Pipeline
```
但是有一个非常关键的结论<br>

你现在这张 Pipelines.csv 可以很好地找到“低硫天然气管道”：
```
Segment_Status = Operating
AND
Substance_1 = Natural gas
AND
H2S_Content = low
```
但它不能告诉你“少水”。<br>

AER 这个 pipeline layout 里没有一个类似：
```
Water_Content
H2O
Dew_Point
```
的字段。官方字段定义里 H2S_Content 是明确的，但没有相应的 water-content 字段。<br>

所以你真正要做的是：
```

             AER Pipelines.csv
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
     Natural Gas             H2S低
          │                     │
          └──────────┬──────────┘
                     ↓
             候选 Pipeline
                     │
                     ↓
              From/To Facility
                     │
                     ↓
               Gas Plant
               / Meter
               / Compressor
                     │
                     ↓
          Gas Quality Data
        H2S / H2O / CO2 / N2
                     │
                     ↓
             Price / Tariff
                     │
                     ↓
             Calgary Delivered
                Gas Cost
```
这才是你真正需要的数据库。<br>

而且你提到的 unmapped pipelines.csv 也不要删掉：AER 明确说明它们没有 spatial reference，所以不进入 Shapefile，但里面的属性数据仍然可以用。
