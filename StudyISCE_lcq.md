# ISCE 2.6.3 Study.

@ 邱梁才(liangcaiqiu),
@ Email: liangcaiqiu@whu.edu.cn
@ Note:
       Created: 2025-02-05
       Updated: 

## 软件源码结构
```
ISCE2/
├── applications/            # 预定义程序目录，用于执行常见的InSAR处理任务
│   ├── dem.py
│   ├── insarApp.py
│   ├── stripmapApp.py
│   ├── topsApp.py
│   └── ...
├── components/              # 核心功能模块
│   ├── isceobj/
│   ├── iscesys/
│   ├── mroipac/
│   ├── stdproc/
│   ├── zerodop/
│   └── ...
├── configuration/           # 配置文件目录
│   ├── buildHelper.py
│   ├── sconsConfigFile.py
│   └── ...
├── contrib/                 # 第三方贡献的工具和脚本
│   ├── alos2filter/
│   ├── alos2proc/
│   ├── alos2proc/
│   ├── mdx/
│   ├── Snaphu/
│   ├── stack/
│   ├── timeseries/
│   └── ...
├── defaults/                # 默认配置
│   ├── logging/
│   ├── plugins/
│   └── ...
├── docker/                  # docker配置
│   ├── build_env.sh
│   ├── isce_env.sh
│   ├── Dockerfile
│   ├── Dockerfile.cuda
│   └── ...
├── docs/                    # 文档/信息
│   ├── dev/
│   └── Doxyfile
├── examples/                # 示例
│   ├── applications/
│   └── input_files/
├── library/                 # 核心库文件
│   ├── isceLib/
│   └── ...
├── schema/                  # 格式定义文件(.xml/.xsd)
│   ├── test/
│   └── ...
├── scons_tools/             # SCons配置脚本
│   └── cuda.py
├── setup/                   # 安装配置文件
│   ├── install.sh
│   ├── setup_config.py
│   ├── setup.py
│   └── ...
├── test/                    # 测试用例
│   ├── components/
│   └── ...
├── __init__.py              # python标记/初始化变量
├── CMakeLists.txt           # CMAKE 配置文件/依赖管理
├── license.py               # 软件许可
├── SConstruct               # SCons 配置文件/依赖管理
├── README.md                # 软件解释文档
└── ...
```

# ISCE 2.6.3 主要架构分析

**ISCE (InSAR Scientific Computing Environment) 2.6.3的源码主要分为以下几个核心部分：**

## 1. applications/
这是ISCE的应用程序目录，包含了各种SAR/InSAR处理工具：
- `dem.py`: 用于DEM（数字高程模型）处理
- `insarApp.py`: InSAR处理的主应用程序
- `stripmapApp.py`: 条带模式SAR数据处理
- `topsApp.py`: Sentinel-1 TOPS模式数据处理

## 2. components/
这是ISCE的核心组件目录，包含了以下主要模块：

### 2.1 isceobj/
- 包含了基本的SAR数据对象定义
- 实现了图像处理、轨道处理等基础功能
- 提供了各种传感器（ALOS、Sentinel-1等）的处理模块

### 2.2 iscesys/
- 系统级功能实现
- 包含配置文件解析器
- 提供日志记录和异常处理功能

### 2.3 mroipac/
- 包含干涉测量处理算法
- 实现了相位解缠、地理编码等功能

## 3. defaults/
- 包含默认配置文件
- 存储各种处理参数的默认值

## 4. library/
- 包含C++和Fortran实现的底层库
- 提供高性能计算支持
- 实现了一些关键的信号处理算法

## 5. contrib/
- 包含第三方贡献的代码和工具
- 提供额外的功能扩展

## 主要功能概述

1. **数据预处理**
   - 原始SAR数据导入
   - 轨道数据处理
   - DEM准备和处理

2. **干涉处理**
   - 配准
   - 干涉图生成
   - 相位解缠
   - 地理编码

3. **时间序列分析**
   - 形变监测
   - 大气改正
   - 时间序列反演

4. **工具支持**
   - 数据格式转换
   - 可视化工具
   - 质量控制

ISCE的这种模块化架构设计使得：
- 各个功能模块相对独立
- 便于维护和扩展
- 支持不同的SAR传感器数据处理
- 可以灵活组合不同的处理流程


------------------------------
## 脚本工具
- isce2geotiff.py
  
**脚本功能：** 使用GDAL将ISCE的结果转为geotiff格式，以便使用其他软件进行读取处理。
脚本必要参数有4个，分别是`-i`：输入的ISCE结果（带转换的数据）；`-o`：输出文件名；`-b`：需要转换的波段（解缠数据一般是设置1，缠绕相位一般是设置0；`-c`：色带范围）
```
(insar) user@dell-precision-3680:$ isce2geotiff.py --help

usage: isce2geotiff.py [-h] -i INFILE -o OUTFILE [-b BAND] -c CLIM CLIM
                       [-m CMAP] [-t TABLE] [-n NCOLORS]

Generate graphics from ISCE products using gdal

options:
  -h, --help    show this help message and exit
  -i INFILE     Input ISCE product file
  -o OUTFILE    Output GEOTIFF file
  -b BAND       Band number to use if input image is multiband. Default: 0
  -c CLIM CLIM  Color limits for the graphics
  -m CMAP       Matplotlib colormap to use
  -t TABLE      Color table to use
  -n NCOLORS    Number of colors
```
**样例：**
```
**filt_topophase.flat.geo**
isce2geotiff.py -i filt_topophase.unw.geo -o /mnt/NewDisk/data/InSAR/2016MeinongEarthquake/InSAR/ISCE2/A_0109_0214/merged/filt_topophase.unw.geo.tif -b 1 -c -50 50
Warning 1: Input dataset has no nodata value. Ignoring 'nv' entry in color palette
Input file size is 10246, 8055
0...10...20...30...40...50...60...70...80...90...100 - done.

**filt_topophase.flat.geo**
isce2geotiff.py -i filt_topophase.flat.geo -o /mnt/NewDisk/data/InSAR/2016MeinongEarthquake/InSAR/ISCE2/A_0109_0214/merged/filt_topophase.flat.geo.tif -b 0 -c -50 50
Warning 1: Input dataset has no nodata value. Ignoring 'nv' entry in color palette
Input file size is 10246, 8055
0...10...20...30...40...50...60...70...80...90...100 - done.
```

- isce2gis.py
**脚本功能：** 将ISCE的结果直接导出为VRT或者ENVI格式。

```
isce2gis.py --help
usage: isce2gis.py [-h] {vrt,envi} ...

Export ISCE products directly to ENVI / VRT formats

positional arguments:
  {vrt,envi}  Output format options
    vrt       Export with VRT file
    envi      Export with ENVI hdr file

options:
  -h, --help  show this help message and exit
```

- gdal2isce_xml.py
**脚本功能：** 通过GDAL制作ISCE的xml文件。
```
gdal2isce_xml.py --help
usage: gdal2isce_xml.py [-h] -i FNAME

Generate ISCE xml from gdal products

options:
  -h, --help            show this help message and exit
  -i FNAME, --input FNAME
                        Input filename (GDAL supported)
```

## 数据处理流程
### stackSentinel.py参数列表
| 参数 | 短选项 | 长选项 | 类型 | 是否必需 | 默认值 | 选项 | 含义 |
|--|--|--|--|--|--|--|--|
| display_help | -H | --hh | 无 | 否 | 无 | 无 | 显示详细帮助信息，调用 customArgparseAction 打印 helpstr 并退出程序 |
| slc_directory | -s | --slc_directory | str | 是 | 无 | 无 | 包含所有 Sentinel SLC 的目录 |
| orbit_directory | -o | --orbit_directory | str | 是 | 无 | 无 | 包含所有轨道数据的目录 |
| aux_directory | -a | --aux_directory | str | 是 | 无 | 无 | 包含所有辅助文件的目录 |
| working_directory | -w | --working_directory | str | 否 | ./ | 无 | 工作目录 |
| dem | -d | --dem | str | 是 | 无 | 无 | DEM 文件的路径 |
| polarization | -p | --polarization | str | 否 | vv | 无 | SAR 数据的极化方式 |
| workflow | -W | --workflow | str | 否 | interferogram | slc, correlation, interferogram, offset | InSAR 处理的工作流 |
| swath_num | -n | --swath_num | str | 否 | 1 2 3 | 无 | 要处理的条带列表 |
| bbox | -b | --bbox | str | 否 | 无 | 无 | 经纬度边界（SNWE），示例：'19 20 -99.5 -98.5' |
| exclude_dates | -x | --exclude_dates | str | 否 | 无 | 无 | 要排除处理的日期列表，示例：'20141007,20141031' |
| include_dates | -i | --include_dates | str | 否 | 无 | 无 | 要包含处理的日期列表，示例：'20141007,20141031' |
| start_date | --start_date | 无 | str | 否 | 无 | 无 | 堆栈处理的开始日期，格式应为 YYYY-MM-DD，如 2015-01-23 |
| stop_date | --stop_date | 无 | str | 否 | 无 | 无 | 堆栈处理的结束日期，格式应为 YYYY-MM-DD，如 2017-02-26 |
| coregistration | -C | --coregistration | str | 否 | NESD | geometry, NESD | 配准选项 |
| reference_date | -m | --reference_date | str | 否 | 无 | 无 | 参考采集的目录 |
| snr_misreg_threshold | --snr_misreg_threshold | 无 | str | 否 | 10 | 无 | 使用互相关估计距离失配准的 SNR 阈值 |
| esd_coherence_threshold | -e | --esd_coherence_threshold | str | 否 | 0.85 | 无 | 使用增强谱分集估计方位失配准的相干阈值 |
| num_overlap_connections | -O | --num_overlap_connections | str | 否 | 3 | 无 | 用于 NESD 计算（方位偏移失配准）的每个日期和后续日期之间重叠干涉图的数量 |
| num_connections | -c | --num_connections | str | 否 | 1 | 无 | 每个日期和后续日期之间干涉图的数量 |
| azimuth_looks | -z | --azimuth_looks | str | 否 | 3 | 无 | 干涉图多视处理时的方位视数 |
| range_looks | -r | --range_looks | str | 否 | 9 | 无 | 干涉图多视处理时的距离视数 |
| filter_strength | -f | --filter_strength | str | 否 | 0.5 | 无 | 干涉图滤波的强度 |
| unw_method | -u | --unw_method | str | 否 | snaphu | icu, snaphu | 解缠方法 |
| rmFilter | --rmFilter | 无 | bool | 否 | False | 无 | 是否生成一个额外的去除滤波效果的解缠文件 |
| param_ion | --param_ion | 无 | str | 否 | 无 | 无 | 电离层估计参数文件，若提供将进行电离层估计 |
| num_connections_ion | --num_connections_ion | 无 | str | 否 | 3 | 无 | 用于电离层估计的每个日期和后续日期之间干涉图的数量 |
| useGPU | -useGPU | --useGPU | bool | 否 | False | 无 | 当可用时允许程序使用 GPU |
| num_process | --num_proc | --num_process | int | 否 | 1 | 无 | 每个运行文件中并行运行的任务数量 |
| num_process4topo | --num_proc4topo | --num_process4topo | int | 否 | 1 | 无 | 并行进程的数量（仅针对地形） |
| text_cmd | -t | --text_cmd | str | 否 | 无 | 无 | 要添加到每个运行文件每行开头的文本命令 |
| virtual_merge | -V | --virtual_merge | str | 否 | 无 | True, False | 对合并的 SLC 和几何文件使用虚拟文件，不同工作流有不同默认值 |


### ISCE处理流程
**run_01_unpack_topo_reference：**

用于配置和执行解包堆栈参考SLC的操作。它会调用unpackStackReferenceSLC方法，从safe_dict中获取相关信息，对参考Sentinel - 1 SLC数据进行解包，将数据从原始的SAFE文件格式转换为适合后续处理的格式。

**run_02_unpack_secondary_slc：**

负责配置和执行解包辅助SLC的任务。通过调用unpackSecondarysSLC方法，使用stackReferenceDate、secondaryDates和safe_dict中的信息，对辅助日期对应的Sentinel - 1 SLC数据进行解包处理，同样将数据转换为可进一步处理的格式。

**run_03_average_baseline：**

用于配置和执行计算平均基线的操作。它调用averageBaseline方法，利用stackReferenceDate和secondaryDates相关数据，计算平均基线。基线信息在后续的干涉测量等处理中非常重要，它反映了不同SLC数据采集时卫星轨道的相对位置关系。

**run_04_extract_burst_overlaps：**

由于inps.coregistration为NESD且updateStack为False，被生成并用于配置和执行提取突发重叠区域的操作。通过调用extractOverlaps方法，从解包后的SLC数据中确定不同突发之间的重叠区域，这些重叠区域的数据将用于后续的精确配准步骤。

**run_05_overlap_geo2rdr：**

负责配置和执行将重叠区域从地理坐标转换为距离向坐标并计算偏移量的操作。调用geo2rdr_offset方法，针对secondaryDates对应的SLC数据，在之前提取的重叠区域基础上，进行坐标转换和偏移量计算，为后续的重采样做准备。

**run_06_overlap_resample：**

用于配置和执行对重叠区域进行带载波重采样的操作。通过调用resample_with_carrier方法，以secondaryDates数据为基础，结合之前计算的偏移量，对重叠区域的数据进行重采样，使不同SLC数据在重叠区域的坐标和分辨率达到一致，便于后续处理。

**run_07_pairs_misreg：**

如果updateStack为True，配置和执行针对secondaryDates的成对配准误差处理；若updateStack为False，则针对acquisitionDates。通过调用pairs_misregistration方法，利用safe_dict中的数据信息，对每对SLC数据进行配准误差的计算和分析，以确定数据之间的精确差异，为后续的时间序列处理提供基础。 

**run_08_timeseries_misreg：**

用于配置和执行时间序列的配准误差处理。调用timeseries_misregistration方法，综合之前计算的成对配准误差信息，对整个时间序列的SLC数据进行配准误差的进一步分析和处理，以提高数据在时间维度上的一致性和准确性。

**run_09_fullBurst_geo2rdr：**

负责配置和执行对全突发数据进行从地理坐标到距离向坐标的偏移处理。调用geo2rdr_offset方法，并设置fullBurst='True'，对secondaryDates对应的全突发SLC数据进行坐标转换和偏移量计算，与之前重叠区域的处理类似，但针对的是整个突发数据。

**run_10_fullBurst_resample：**

用于配置和执行对全突发数据进行带载波的重采样操作。调用resample_with_carrier方法，同样设置fullBurst='True'，对经过地理到距离向坐标偏移处理后的全突发secondaryDates数据进行重采样，确保全突发数据在坐标和分辨率上的一致性，满足后续处理要求。 

**run_11_extract_stack_valid_region：**

负责配置和执行提取堆栈有效区域的操作。通过调用extractStackValidRegion方法，对经过前面一系列处理后的SLC堆栈数据进行分析，确定数据中的有效区域，排除无效或噪声数据区域，提高后续处理的数据质量。 

**run_12_merge_reference_secondary_slc：**

因为mergeSLC为True，用于配置和执行合并参考SLC和辅助SLC的操作。通过调用mergeReference方法合并参考日期的SLC数据，调用mergeSecondarySLC方法合并secondaryDates对应的辅助SLC数据，并设置virtual = 'False'，表示采用实际合并而非虚拟合并的方式，将多个SLC数据合并为一个更便于后续处理的数据集。 

**run_13_grid_baseline：**

负责配置和执行网格化基线的操作。调用gridBaseline方法，使用stackReferenceDate和secondaryDates相关数据，将之前计算的平均基线信息进行网格化处理，将基线数据转换为适合网格化分析和后续应用的格式。

**run_14_merge_burst_igram：**

配置了合并突发干涉图的操作，接着调用runObj.igram_mergeBurst(acquisitionDates, safe_dict, pairs)方法，将之前生成的突发干涉图进行合并，以得到更完整或适合后续处理的干涉图数据。

**run_15_filter_coherence：**

用于配置对干涉图相干性进行滤波的操作，然后调用runObj.filter_coherence(pairs)方法，对由pairs确定的干涉图数据进行相干性滤波处理，以提高干涉图的质量和可靠性。

**run_16_unwrap：**

配置了相位解缠的操作，随后调用runObj.unwrap(pairs)方法，对pairs对应的干涉图数据进行相位解缠处理，将缠绕的相位数据转换为连续的相位值，以便进行后续的变形分析等应用。 





