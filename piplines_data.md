```
Pipelines_SHP/
│
├── Pipelines.csv                         ← 属性数据
│
├── Unmapped Pipelines.csv                ← 没有空间坐标的管道
│
├── Pipelines_GCS_NAD83.shp              ← 管道空间线
├── Pipelines_GCS_NAD83.dbf
├── Pipelines_GCS_NAD83.shx
├── Pipelines_GCS_NAD83.prj
├── Pipelines_GCS_NAD83.cpg
│
├── Pipelines_NAD83_10TM_AEPForest.shp   ← 管道空间线
├── Pipelines_NAD83_10TM_AEPForest.dbf
├── Pipelines_NAD83_10TM_AEPForest.shx
├── Pipelines_NAD83_10TM_AEPForest.prj
├── Pipelines_NAD83_10TM_AEPForest.cpg
│
└── *.xml                                  ← metadata
```

Pipelines.csv 和 .shp 配合

```
Pipelines.csv
      │
      │ 属性
      ↓
Licence_Number
Company_Name
H2S_Content
Substance_1
Pressure
Diameter
...
      │
      │
      ↓
Pipelines_NAD83_10TM_AEPForest.shp
      │
      │ 空间 geometry
      ↓
LINESTRING
      │
      ↓
Calgary 距离
```
通常 Shapefile 的 .dbf 中也会带有属性字段，所以不一定需要自己把 CSV 再 join 一次。

```
Pipeline
│
├── Licence_Number
├── Company_Name
├── H2S_Content
├── Substance 1
├── Diameter
├── MAOP
├── From_Facility
└── To_Facility
```

```
Installation
│
├── Pipeline_Licence_Number
├── Pipeline_Installation_ID
├── Installation_Type
├── BA_Name
├── Prime_Source
├── Installation_Status
├── H2S_Content
└── Substance_1
```
结合：
```
                    PIPELINE LICENCE
                           │
                           │
              Pipeline_Licence_Number
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
         PIPELINE                  INSTALLATION
              │                         │
              │                         ├── MS
              │                         ├── MR
              │                         ├── RS
              │                         └── CS
              │
       H2S / Diameter / MAOP
```
