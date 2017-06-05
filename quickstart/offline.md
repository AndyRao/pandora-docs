### 创建离线工作流

**操作流程：**

进入大数据工作流引擎，开始搭建离线工作流，点击大数据工作流引擎页面的**创建离线工作流**按钮，进入工作流编辑页面。

在工作流编辑器中，我们第一个看到的节点元素是`数据源`，其含义是**指定需要计算的数据所在地**；在这个节点中，我们首先要给这个数据源起一个名字，这个名字也是**工作流名称**。

!> 注意：每个公有云用户最多可以创建20个数据源。

!> 注意：填写完成数据源信息后，请点击**加载字段信息**按钮，系统会自动解析文件格式。

**数据源节点填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|名称|是|用来标识该数据源的唯一性|
|数据源类型|是|数据所在地的存储方式，目前仅支持对象存储|
|空间名称|是|七牛对象存储的Bucket名称|
|文件类型|是|使用哪一种方式来解析文件|
|文件前缀|否|以文件名称作为文件筛选条件|

**操作演示：**

![](_media/offline1.gif)


### 数据计算（SQL）

**操作流程：**

`数据源`节点中的数据，可以进行计算。

右键`数据源`节点，选择计算任务。

一个`数据源`可以有多个计算任务，每一个计算任务也可以派生子计算任务，以满足复杂的业务需求。

计算方式目前仅支持`SQL`。

`计算任务`是消耗物理资源的，所以每一个`计算任务`节点都需要为之分配**计算资源**（CPU & 内存）， 不同的`计算任务`之间的计算资源互相隔离，互不影响。

离线计算任务支持调度，目前支持3种调度方式：单次执行、定时执行、循环执行；

* 单次执行：每次需要执行时，需要点击工作流页面右上角的**计算任务列表**按钮，选择需要执行的计算任务名称，然后点击**运行**。
* 定时执行：输入一个标准的crontab表达式，工作流根据表达式的内容定时开始执行。
* 循环执行：输入类似 `5m`(5分钟)、`1h`（1小时）格式的内容，工作流根据时间自动循环执行；目前仅支持`m`（分钟）、`1h`（小时）两种单位。

离线计算任务支持魔法变量，并且提供6种默认变量：

```
$(year)=当前年份
$(mon) =当前月份
$(day) =当前日期
$(hour)=当前小时
$(min) =当前分钟
$(sec) =当前秒数
```

用户也可以自己定义魔法变量，并为其赋值，应用在`文件过滤条件`或`SQL语句`中；在设置调度方式为`单次运行`时，每次运行之前，可以改变魔法变量的值。

例如：
我们建立个魔法变量名称为 `createYear`，默认值为 `2017`；
那么编写sql时可以这么写：

```
select create_date,id from datesource where create_date = $(createYear)
```

实际解析后的SQL语句是：

```
select create_date,id from datesource where create_date = 2017
```


**计算任务节点填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|名称|是|用来标识该任务的唯一性|
|文件过滤条件|否|允许使用魔法变量以及`*`来动态指定数据源目录下需要加载的文件|
|数据表名称|是|为被加载的数据模拟一张表，并起一个别名|
|容器类型&数量|是|计算资源的数量及类型|
|调度方式|是|计算任务的调度方式以及运行规律|

**操作演示：**

![](_media/offline2.gif)

### 数据导出

**操作流程：**

任何一个`计算任务`节点中的数据，都可以导出。

目前仅支持将数据导出到`对象存储`服务当中。

鼠标右键点击`计算任务`节点，选择`导出任务`。

**导出任务节点填写参数说明：**

|参数|必填|说明|
|:---|:---|:---|
|名称|是|用来标识该任务的唯一性|
|空间名称|是|七牛对象存储的Bucket名称|
|文件前缀|否|文件名称的前缀|
|导出类型|是|导出文件的类型|
|文件压缩|是|以哪种方式压缩文件|
|文件保存天数|是|文件存储时限|
|文件数量|是|导出的文件数量|
|字段分区|否|选择一些字段进行分区，合理的分区可以节省存储空间以及提高查询性能|

**操作演示：**

![](_media/offline3.gif)