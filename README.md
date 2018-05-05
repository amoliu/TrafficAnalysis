##Traffic Analysis

####概要

数据集：宣城交通数据集（http://www.openits.cn/openData2/746.jhtml）

- 交通拓扑（路段、路口）、路段旅行时间、路口车流量。数据量仅为1天。

研究思路：

- 首先研究一条直线上的两个路口之间的因果影响情况，跑通实验，验证分析思路的正确性。
- 其次选取几个流量较小的路口作为可能的cause，选取一个流量很大的路口作为effect，探究其中的因果关系与权重。


主要方法：

- 主要参考论文：[A.Fire and S.-C. Zhu. "Learning perceptual causality from video." ACM Trans. Intell. Syst. Technol., 7(2):23:1–23:22, 2016.](http://amyfire.com/projects/learningcausality)
- 大体思路：逐步构建因果树模型，刻画交通路口间的因果关系。
- 构建模型的算法：
  - 选定因路口A、果路口B，并定义“因动作（$$A$$）”、“果状态（$$\Delta F$$）”及其0-1阈值。（可配图说明）
  - 将两个路口的交通数据看作一个大的视频，对其进行分片。片段的时长应大于从A到B的旅行时间（暂使用Dijkstra最短路算法计算）。
  - 对所有分片进行统计。真实数据中因果的共现概率分布为$$f$$，初始化当前模型共现概率分布$$h$$，$$p_0=p_A*p_{\Delta F}$$。
  - 模型构建的方法为：每轮迭代，增加一条信息增益（information gain）最大的因果边，使模型分布$$h$$以最大步长逼近真实分布$$f$$。
  - 停止条件：信息增益小于某个阈值。


已完成的尝试：

```
# TODO: 2. 考虑使用最新的VIP数据，如何更好地定义Action。 【OK】已和学长讨论
# TODO: 4. 让c_ssid支持多个“因”路口。 【OK】
# TODO: 14. 尝试一下把F变成多个方向的。【OK】效果好像并不好。。
# TODO: 15. 尝试把阈值的“平均值”改成白天的平均值或中位数，看怎么效果更好。（数据清洗）【OK】
# TODO: 18. 5min粒度粗，按照论文方法生成origin_data往往time_lag里的动作数为0，如何解决【OK】

【尝试得到的结论】
对于唯一成功的HK-173 -> HK-83，数据清洗+中位数，效果不如不清洗+平均数。
make_origin_data时，保留所有时刻的 time_tag|fluent|actions 数据，效果很好。（可能原因：解决了“TODO:18.”的问题）

create_examples非常关键，可以着重思考“到底应该如何分片更科学？”
```




####tally.py：初步数据统计

- ssid_volume()：统计各路口各方向一天内的总流量、平均流量
- roadid_traveltime()：统计各路段的平均旅行时间、平均路程
- direction_volume()：统计要研究的路口各方向的车流量，每5分钟为一个时间段（可能需要plot绘直方图，辅助确定F、A的阈值的选取，继而定义F、A）
- make_origin_data()（暂时是取平均值作为流量阈值参数，或直接人为设定，**后应改为更加科学、统一的自动阈值设定**）：生成每一行为 “time | fluent | actions” 格式的 origin_data.csv，保证相邻两行的各对应值不完全相同
- find_path_return_travel_time()：输入目标路口id和“因”路口id，输出两者间的旅行时间。暂时使用了Dijkstra算法找最短路的时间，**后应考虑如“最短的三条路径的加权平均”等**




####learning.py：分析数据，学习因果关系

- create_examples_with_prev_fluent()（后单独写在一个.py文件中）：
  - 从time | fluent | actions 的origin_data到“关键帧”： fluent | actions | prev_fluent
- pursuit()：通过迭代，逐渐构建因果边，返回构建的因果树结果。



存在问题：

- 数据记录实时性不足：每5分钟才统计一次车流量、旅行时间，而大多数相邻路口间的车程不足5分钟。




####中期汇报

- 算法框架√
- 实验数据√
- 实验结果
  - 好结果的图（已验证算法的正确性，说明可能在别的复杂路口亦有一定适用性）
  - 坏结果的图（口头分析原因，考虑多种可能）
  - 需解释图中含义：绿点表示因果边$$A \rightarrow \Delta F$$满足$$P(\Delta F\ |\ A)>P(\Delta F\ |\ \overline{A})$$
  - 需在图上标记best action和best fluent change，或者做一个表格
- 后续工作
  - 实验后续：考虑数据清洗，尝试更多的路口组合，并调整阈值参数
  - 算法改进：
    - 分片时长的确定：考虑多条路径
    - 共现概率的定义：考虑对不同远近路口的加权
    - 进一步思考和改进核心迭代算法



####待研究问题

- causal_effect值的含义？**（与学长讨论）**【OK】
- 如果A、B路口间有***多条路径***，或者在一天中的不同时间，**它们之间的time delay（平均行车时间）是变化的**：
  - 问题1：难以确定分片的标准（可以取max？取前三条最短路径再加权平均？）
  - 问题2：如果算法发现A对B有影响，怎么确定A对**多少分钟后**的路口B有影响？（重要，如果解决了，就可以做预测问题）
- ***多个路口***对一个路口的影响：
  - 代码拓展
  - 把RF的定义从平均变为**加权**，应该怎么写？
- 讨论




#### 实验记录：

见result_recorder.txt。







####论文要求

- 参考文献至少要有20来篇
- 图表要清晰
- 至少要50页
- ​








####图层的使用

- 打开ArcGIS官网的Free Trial（http://www.esri.com/arcgis/trial）
- 注册账号
- 打开ArcGIS for Developers界面，添加新的Layer，一顿瞎设置
- 在Dashboard / Layers 界面右边点击“Open in Mar Viewer”，进入“我的地图”界面
- 在“我的地图”界面左上角点击：添加→从文件添加图层，即可导入文件中的“含shape文件的zip包”



