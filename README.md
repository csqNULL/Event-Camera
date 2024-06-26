# Event-Camera

# 事件相机相关模块、软件使用说明

## 该文档用途：
1. 说明基于事件相机进行机器视觉标记识别与位姿估计代码模块使用方法
  - 1.1 基于Aruco码主动LED标记（Acitve LED Marker）
  - 1.2 基于频率特征点检测
2. 说明基于频率特征点检测信息发端的上位机使用说明

## 1 基于事件相机进行机器视觉标记识别与位姿估计代码模块使用方法
### 1.1 基于Aruco码主动LED标记（Acitve LED Marker）
### 1.1.1 项目简介
  该代码块用于进行基于事件相机和Aruco码的机器视觉标记识别项目，是该项目的核心算法，其用途是通过输入事件相机的事件数据进行自动标记识别与位姿估计。  <br>
### 1.1.2 功能
  这段代码是机器视觉标记识别项目的核心算法，其功能包括通过事件相机的事件数据自动识别标记并估计其位姿。具体而言，它能够分析事件数据，识别出场景中的Aruco码标记，并推断标记在空间中的位置和方向，实现了标记识别和位姿估计的自动化处理。由于采用了事件相机作为输入数据源，代码具有很强的适应性，能够在各种复杂环境下稳健地运行，为机器视觉领域的应用提供了强大支持。  <br>
### 1.1.3 运行环境（建议配置）
  操作系统：  Windows11 64-bit  <br>
  处理器：    Intel Core 17-13700H @2.4GHz or AMD Ryzen 7 7840HS 8-core @3.8GHz  <br>
  内存：      16GB  <br>
  显卡：      无要求  <br>
  磁盘：      256GB  <br>
  python环境：python3.8.10  <br>
  必要库：    numpy1.24.4、opencv-python4.5.5.64、metavision_sdk_core、metavision_core.event_io、metavision_sdk_cv、metavision_hal   <br>
注：最低配置尚未实测，仅展示测试平台环境  <br>
### 1.1.4 使用说明
该代码模块处于完整的图像标记识别与检测的中间流程，前级事件相机将流数据输入数据处理模块，将流数据转化为数据帧，再将帧数据传给本模块。
```mermaid
graph TD;
    Dynamic_Visual_Sensor(Event Camera)-->Data_Stream_Processing;
    Data_Stream_Processing-->This_Module
    This_Module-->Automatic_Control;
```
#### 输入输出
  - 输入数据：
    - 数据帧  <br> 数据格式：三维矩阵
        当sequence_filename置为空时，流数据采集自事件相机，否则将根据文件路径读取已录制好的视频文件 <br>
  ```
    mv_it = EventsIterator(sequence_filename, start_ts=1000000, mode="mixed", delta_t=200, n_events=5000000)
    height, width = mv_it.get_size()
    ...
    event_frame_gen = PeriodicFrameGenerationAlgorithm(sensor_width=width, sensor_height=height,
                                                       accumulation_time_us=5000, fps=300.0, palette=ColorPalette.Dark)
  ```
  - 频率区间  <br>  数据格式：int型
  ```
          frequency_filter = metavision_sdk_cv.FrequencyAlgorithm(width=width, height=height, min_freq=1., max_freq=2000.)
          frequency_clustering_filter = metavision_sdk_cv.FrequencyClusteringAlgorithm(width=width, height=height,min_cluster_size=1,max_time_diff=1500)
  ```
  - 输出数据：三维坐标点
  - 数据格式：三元坐标矩阵

#### 后续数据使用
  将输出的三维坐标数据输入自动控制终端即可实现其自主导航。  <br>
### 1.1.5 性能
  最高可以50us的时间间隔处理10kHz的频闪数据，每个三维坐标点输出时间间隔不大于300us。<br> 
  为保证较高的定位精度和准确率，建议闪烁频率不要高于5kHz。  <br> 
### 1.1.6 非常规过程
  暂无  <br>

### 1.2 基于频率特征点检测
### 1.2.1 项目简介
  该代码块用于进行基于事件相机和频率特征点的机器视觉标记识别项目，是该项目的核心算法，其用途是通过输入事件相机的事件数据进行自动标记识别与位姿估计。
### 1.2.2 功能
  这段代码是机器视觉标记识别项目的核心算法，其功能包括通过事件相机的事件数据自动识别标记并估计其位姿。具体而言，它能够分析事件数据，识别出场景中的频率编码标记，并推断标记在空间中的位置和方向，实现了标记识别和位姿估计的自动化处理。由于采用了事件相机作为输入数据源，代码具有很强的适应性，能够在各种复杂环境下稳健地运行，为机器视觉领域的应用提供了强大支持。
### 1.2.3 运行环境（建议配置）
  操作系统：  Windows11 64-bit  <br>
  处理器：    Intel Core 17-13700H @2.4GHz or AMD Ryzen 7 7840HS 8-core @3.8GHz  <br>
  内存：      16GB  <br>
  显卡：      无要求  <br>
  磁盘：      256GB  <br>
  python环境：python3.8.10  <br>
  必要库：    numpy1.24.4、opencv-python4.5.5.64、metavision_sdk_core、metavision_core.event_io、metavision_sdk_cv、metavision_hal  <br>
注：最低配置尚未实测，仅展示测试平台环境  <br>
### 1.2.4 使用说明
该代码模块处于完整的图像标记识别与检测的中间流程，不同于1.1的模块，前级事件相机将未经处理过的流数据直接输入该模块，该模块对流数据直接进行处理，而非以帧格式，这可以极大提高代码的时延。
```mermaid
graph TD;
    Dynamic_Visual_Sensor(Event Camera)-->This_Module;
    This_Module-->Automatic_Control;
```
#### 输入输出
  - 输入数据：
    - 异步事件流  <br> 数据格式：四元组
        当sequence_filename置为空时，流数据采集自事件相机，否则将根据文件路径读取已录制好的视频文件 <br>
  ```
    mv_it = EventsIterator(sequence_filename, start_ts=1000000, mode="mixed", delta_t=200, n_events=5000000)
    height, width = mv_it.get_size()
  ```
  - 频率区间  <br>  数据格式：int型
  ```
          frequency_filter = metavision_sdk_cv.FrequencyAlgorithm(width=width, height=height, min_freq=1., max_freq=2000.)
          frequency_clustering_filter = metavision_sdk_cv.FrequencyClusteringAlgorithm(width=width, height=height,min_cluster_size=1,max_time_diff=1500)
  ```
  - 输出数据：三维坐标点
  - 数据格式：三元坐标矩阵

#### 后续数据使用
  将输出的三维坐标数据输入自动控制终端即可实现其自主导航。  <br>
### 1.2.5 性能
  最高可以50us的时间间隔处理10kHz的频闪数据，每个三维坐标点输出时间间隔不大于300us。<br> 
  为保证较高的定位精度和准确率，建议闪烁频率不要高于5kHz。  <br> 
### 1.2.6 非常规过程
  暂无  <br>

## 2 基于频率特征点检测信息发端的上位机使用说明
### 2.1 项目简介
  这款上位机软件是专门为Windows平台而设计开发的，采用了QT Designer作为开发工具。它主要面向Windows系统用户，并且其设计理念围绕着与嵌入式系统之间的USART串口通信展开。通过利用图形化界面，该软件能够将底层算法逻辑转化为用户友好的交互式界面，使得用户能够轻松理解和操作。其目标是为了让新手能够快速上手，同时保持高交互性和可解释性。
### 2.2 功能
  基于频率特征点检测的信息发端是使用基于ARM内核MCU的嵌入式硬件平台，而这款上位机软件则是其必不可少的补充，其有助于进一步强化该平台的灵活性和可操作性，通过上位机控制该嵌入式系统做出相应响应，以满足不同环境下的LED标记布设计。<br>
  该上位机软件可设置上位机串口通信各项参数，将底层通信指令（自主设计通信协议格式）转化为可视化、图形化语言，发送给嵌入式系统。上位机会根据输入信息自行计算系统指标，如最大信息量（Number of Pattern）、基于频率进行信源编码和调制、编码信息发送。
### 2.3 运行环境（建议配置）
  操作系统：  Windows11 64-bit  <br>
  处理器：    Intel Core 17-13700H @2.4GHz or AMD Ryzen 7 7840HS 8-core @3.8GHz  <br>
  内存：      16GB  <br>
  显卡：      无要求  <br>
  磁盘：      256GB  <br>
  QT版本：    4.11.1  <br>
  必要外设：  USB转串口模块  <br>
注：最低配置尚未实测，仅展示测试平台环境  <br>
### 2.4 使用说明
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/89d8522a-baf6-42e1-9ff0-cd4851b3d357" width="50%">
</div>
  上位机软件界面如图所示。  <br>
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/cfea3759-744b-44d5-9d02-e193ffbe9d99" width="20%">
</div>
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/c32c4852-26c4-449f-bb25-c32d8b0f1418" width="50%">
</div>
  
  左上角是**串口设置**，下拉窗口可分别选择`串口端口`、`波特率`、`数据位`、`停止位`和`校验位`。用户可自行设置串口各参数设置，并点击下方按钮检测当前可连接的串口，并点击`打开串口`来打开该串口，使其进入连接状态。  <br>
  
  下面是**接收设置**和**发送设置**，接收可设置`Hex接受`、`显示时间戳`和`自动换行`，便于用户查看接受命令，点击`清空接收`即可清空接收窗口内容；发送设置可设置为`Hex发送`和`自动发送`，其中自动发送需要输入发送时间间隔，该间隔不能过小。  <br>
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/5332a91c-ada6-4a48-8313-354b402cb672" width="20%">
</div>
  
  编码设置可配置频率编码位数和编码进制，默认为四位一进制编码，即四个顶点频率分别为：顶点1 = 1kHz、顶点2 = 2kHz、顶点3 = 3kHz、顶点4 = 4kHz。全部频率配置如下  <br>
| 编码位数   | 编码进制   |   顶点1    |  顶点2    |  顶点3   |   顶点4 |
| ---------- |---------- |---------- |---------- |---------- |---------- |
|      4     |     1     |    {1kHz}   |    {2kHz}   |    {3kHz}  |    {4kHz}    |
|      4     |     2     |    {1kHz,1.5kHz}   |    {2kHz,2.5kHz}   |    {3kHz,3.5kHz}  |    {4kHz,4.5kHz}    |
|      4     |     3     |    {1kHz,1.35kHz,1.7kHz}   |    {2kHz,2.35kHz,2.7kHz}   |    {3kHz,3.35kHz,3.7kHz}  |    {4kHz,4.35kHz,4.7kHz}    |
|      4     |     4     |{1kHz,1.25kHz,1.5kHz,1.75kHz}|{2kHz,2.25kHz,2.5kHz,2.75kHz}| {3kHz,3.25kHz,3.5kHz,3.75kHz} | {4kHz,4.25kHz,4.5kHz,4.75kHz}|

<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/fa780b99-cb7b-41cf-8991-af7aef51fd17" width="30%">
</div>

  中间模块是**编码设置**，可以设置`频率编码位数`、`频率编码进制`，进而自动计算其`信息容量`，即信息图样的个数。  <br>
  下面是**编码内容输入模块**，用户可以在`编码内容`内输入想要编码的码型，但不能超过信息容量，否则会提示错误。  <br>
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/d4ee792d-8ef5-4003-8d10-55b8737adaa4" width="20%">
</div>

  用户可以自行选择输入的进制模式，可选`二进制`、`十进制`、`十六进制`；同理，编码结果可选`二进制`和`十六进制`。  <br>
  右上角的表格展示了根据用户输入的编码内容进行频率编码后的频率与码元对应表，下图所示的一个示例，其编码内容为0b11。  <br>
<div align="center">
	<img src="https://github.com/csqNULL/project_EventCamera/assets/107977229/e11a0907-5b13-497a-9139-980b6375591e" width="30%">
</div>

  输入编码内容并成功进行编码后，即可点击`发送`来发送数据，此处可以通过勾选`发送新行`来使发送指令附带一个换行符，该指令对收端没有影响。
  ### 2.5 非常规过程
  暂无  <br>
