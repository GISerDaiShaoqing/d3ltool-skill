# 产品目录速查 / Product catalog cheat sheet

**实测入口永远是 `list_products`**（`group` 子串筛选；单次输出上限 80 行，超出附 hint 提示
再筛）。任何 CMR short_name 都可以直接试——不限于本表。版本信息：MODIS/VIIRS 为 `6.1`/`2`，
MERRA-2 为 `5.12.4`，`search_granules` 的 `version` 参数可精确过滤。

## NASA（CMR 检索，用 tiles/bbox 定位）

| 组 | short_name | 内容 | 周期 / 分辨率 |
| --- | --- | --- | --- |
| MODIS Terra | MOD13Q1 | 植被指数 NDVI/EVI | 16 天 / 250m |
| MODIS Terra | MOD13A2 | 植被指数 NDVI/EVI | 16 天 / 1km |
| MODIS Terra | MOD11A2 | 地表温度/发射率 LST | 8 天 / 1km |
| MODIS Terra | MOD09GA | 地表反射率 | 日 / 1km |
| MODIS Terra | MOD09GQ | 地表反射率 | 日 / 250m |
| MODIS Terra | MOD10A1 | 积雪覆盖 | 日 / 500m |
| MODIS Terra | MOD17A2H | 总初级生产力 GPP | 8 天 / 500m |
| MODIS Terra | MOD17A3H | 净初级生产力 NPP | 年 / 500m |
| MODIS Aqua | MYD13Q1 | 植被指数 NDVI/EVI | 16 天 / 250m |
| MODIS Aqua | MYD11A2 | 地表温度/发射率 LST | 8 天 / 1km |
| MODIS Aqua | MYD09GA | 地表反射率 | 日 / 1km |
| MODIS Aqua | MYD10A1 | 积雪覆盖 | 日 / 500m |
| MODIS Combined | MCD43A4 | Nadir BRDF 反射率 NBAR | 日 / 500m |
| MODIS Combined | MCD43A3 | 反照率 Albedo | 日 / 500m |
| MODIS Combined | MCD12Q1 | 土地覆盖类型 | 年 / 500m |
| MODIS Combined | MCD64A1 | 火烧迹地 | 月 / 500m |
| VIIRS | VNP46A1 | 夜光 DNB 辐亮度（TOA） | 日 / 500m |
| VIIRS | VNP46A2 | 夜光 VPDN（去云） | 日 / 500m |
| VIIRS | VNP09GA | 地表反射率 | 日 / 1km/500m |
| VIIRS | VNP13A1 | 植被指数 NDVI | 16 天 / 500m |
| MERRA-2 (GES DISC) | M2T1NXSLV | 单层 2D 逐时气象要素（风/温/湿/压） | 逐时 |
| MERRA-2 (GES DISC) | M2T1NXRAD | 单层 2D 逐时辐射通量 | 逐时 |
| MERRA-2 (GES DISC) | M2T1NXAER | 单层 2D 逐时气溶胶 | 逐时 |
| MERRA-2 (GES DISC) | M2T1NXFLX | 单层 2D 逐时湍流/通量 | 逐时 |
| MERRA-2 (GES DISC) | M2I3NPASM | 3D 逐时同化态温压风湿 | 逐时 |
| MERRA-2 (GES DISC) | M2I3NPCLD | 3D 逐时云参数 | 逐时 |
| MERRA-2 (GES DISC) | M2C0NXLND | 常数 2D 陆地掩膜/网格 | 常数 |

**需求 → 产品换算**：NDVI/EVI → `MOD13Q1`（Terra）/`MYD13Q1`（Aqua）；夜光 → `VNP46A1`
（原始 TOA）/`VNP46A2`（去云）；GPP → `MOD17A2H`；反照率 → `MCD43A3`；无云反射率合成 →
`MCD43A4`；土地覆盖 → `MCD12Q1`；火灾 → `MCD64A1`；近地面气温逐时 → `M2T1NXSLV`（T2M，
开尔文）。瓦片定位用 `point_to_tile`（经纬度 → h,v）。

## GLASS（glass.hku.hk 公开档案，全程免登录）

- **变体 id** 形如 `LAI_MODIS_250M`：`{产品}_{传感器/来源}_{分辨率}`。目录由作者 R 包
  glass.sqdaitools 的核验登记表生成——133 个变体、131 个已核验（live number，以
  `list_products(group='GLASS')` 实测为准）。常见产品族：LAI、FPAR、Albedo、BBE、ET、DSR、
  FVC、EVI、FAPAR、AGB、Crops、AWGF LST、FFC、EDD，以及 HiGLASS 20m
  （`LAI_HR_LAI_V01`、`FAPAR_HR_FAPAR_V01`）。
- **归档布局**：v2.6 起四种全覆盖——合成 DOY 目录 / 静态文件集 / 年目录 / `YYYYDDD` 日目录；
  v2.8.0 起 HiGLASS 20m 两个变体按 **1° 带目录**检索（~1120 个带，带名 = 单元格西南角，
  如 `n018e108`），bbox/瓦片自动求交、逐带列表带进度；MCP/CLI/GUI 的 `tiles`
  参数**均可直接填带名**（如 `tiles=["n045e120"]`，与 bbox 求交取并集，只填
  带名时无需 bbox）。注意这两个 id 在登记表里 layout 仍标 flat（"无时间维"），但运行时经
  `BAND_VARIANTS` 特判走带检索——**必须给 bbox/瓦片/带名**，否则报"需先定位"。
- **静态集**（描述含"无时间维（固定层）"，如 `AGB_0.05D`、`Crops_PADDYCALENDAR`、
  `Albedo_ABI_500M`）：检索**免日期**，`start`/`end` 不用传（HiGLASS 两个带变体虽也标
  "无时间维"，但检索仍需定位，见上条）。
- **全球 CMG 变体**（文件名无 `h##v##`，如 `*_0_05D`、`*_CMG_*`）：`tiles`/`bbox` **无效**，
  传了也不会过滤；瓦片型变体则按正弦网格与 bbox 精确求交。
- **特例备注**（目录描述里已标注）：Albedo 多波段 BSA/WSA，选子数据集的后处理在作者 R 包中做；
  BBE 归档 scale_factor 标注方向反（×10000），修正在 R 包中做。
- GLASS **不支持** `point_timeseries`（OPeNDAP 直采只覆盖 MODIS/VIIRS/MERRA-2）；要 GLASS
  时间维就按流程 A 检索下载文件。
