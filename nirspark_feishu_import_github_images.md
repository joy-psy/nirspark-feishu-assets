本指南依据“2_Nirspark数据分析课程”中的 22 门 Nirspark/fNIRS 课程整理，核心目的不是复述课程逐字稿，而是把 Nirspark 近红外数据分析流程整理成一份可直接照着执行的操作步骤。推荐使用顺序是：先完成实验设计，再完成数据导入、marker/ROI、质控和预处理，随后根据研究问题选择脑激活、脑网络、小波幅值或超扫描分析，最后导出指标、统计检验和绘图。文中保留必要英文专有名词，如 fNIRS、Nirspark、marker、ROI、HbO、HbR、block average、GLM、functional connectivity、wavelet amplitude、wavelet coherence 等。

## 课程与分析路线

课程 1-4 主要服务于实验设计。学习目标是判断 fNIRS 是否适合你的课题，明确任务范式、目标脑区、通道布局、marker 规则、分析路线和统计模型。实验设计决定后续能不能分析，不能等数据采完后再临时决定做脑激活还是脑网络。

课程 5-8 主要服务于预处理。学习目标是把原始 `.hcx` 数据导入 Nirspark，检查 marker 和数据长度，完成数据质量查看、质控、ROI 定义、运动伪迹校正、滤波和血氧转换。预处理是所有后续分析的共同基础。

课程 9-16 主要服务于脑激活。学习目标是用 block average 或 GLM 提取任务诱发的 HbO/HbR 变化，完成个体分析、批处理、群组统计、激活地形图、波形图、方差分析、正态性和方差齐性检查、自定义绘图。

课程 17-19 主要服务于脑网络。学习目标是构建 channel/ROI 之间的 functional connectivity 矩阵，完成个体连接分析、批处理、群组统计和图论分析。

课程 20 主要服务于小波幅值。学习目标是提取不同时间-频率尺度上的 wavelet amplitude 指标，用于任务、组别或临床量表比较。

课程 21-22 主要服务于超扫描。学习目标是完成多人同步数据的拆分、SD 变换、预处理、时间对齐和 wavelet coherence 分析，解释人际脑同步。

![Nirspark 软件主界面与功能入口](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/05_02_14-15.jpg)

*图：Nirspark 主界面中常用入口，包括数据导入、marker/ROI、预处理、脑激活、脑网络和结果工具。*

## 实验设计

实验设计的目的，是在采集前把“研究问题、实验任务、通道布局、marker、分析方法、统计方法”全部对齐。fNIRS 不是万能设备，它适合浅层皮层、自然互动、运动/临床/儿童/多人交互等场景，但不适合深部脑区问题，也不适合完全无法控制头动或探头接触的任务。

![fNIRS 使用场景与适用条件](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/01_01_06-46.jpg)

*图：fNIRS 的典型使用场景与适用条件。*

实验设计的第一步是写清楚研究问题和假设。不要写成“我想看看大脑有没有变化”，而要写成可分析的形式，例如“抓握任务相对于休息是否引起对侧运动皮层 HbO 升高”，“患者组与健康组在 VFT 任务中的前额叶 HbO 激活是否不同”，“亲子合作任务中前额叶 inter-brain synchrony 是否高于非互动条件”。这一步的输出应是：研究问题、主要假设、主要因变量、主要自变量和预期方向。

第二步是判断应选择哪一种分析路线。如果问题是“任务是否激活某脑区”，选择脑激活分析；如果问题是“脑区之间是否协同变化”，选择脑网络分析；如果问题是“某个频段的信号幅值是否变化”，选择小波幅值分析；如果问题是“两个人之间是否同步”，选择超扫描分析。注意：分析路线一旦不同，任务设计、数据长度、通道布局和统计表结构都会不同。

第三步是选择任务范式。block design 适合稳定重复任务，例如运动、VFT、情绪观看、认知负荷等；event-related design 适合单个事件刺激，但要有足够间隔；静息态适合脑网络；自然情景或连续任务适合生态效度研究；双人互动适合超扫描。选择范式时要同时写出任务段、休息段、重复次数、条件顺序、随机化方式和总时长。

![block 与 event-related 设计思路](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/03_01_06-11.jpg)

*图：课程中关于根据分析方法反推实验范式的说明。*

第四步是设置任务和休息时间。fNIRS 测量的是血氧动力学反应，HbO/HbR 不会在刺激出现瞬间立刻达到峰值，一般会有数秒延迟。因此 block 不能太短，休息段也不能太短。课程中反复强调：如果上一个 block 的血氧反应尚未回到基线，下一个 block 就开始，后续提取出来的激活指标会混入前一次反应。

![血氧反应延迟与任务周期](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/03_03_25-39.jpg)

*图：血氧反应延迟对任务段和休息段设计的影响。*

第五步是设计 marker。marker 是后续 block average 和 GLM 的时间锚点。你需要在采集前明确：哪个 marker 表示任务开始，哪个 marker 表示任务结束，不同条件是否用不同 value，是否需要记录休息开始，是否存在练习段或无效段。例如一个“15 秒抓握 + 20 秒休息”的任务，可以每次抓握开始打 Mark 1；如果还需要标记任务结束，可以在抓握结束打 Mark 2。无论怎样设计，后续分析参数必须与 marker 规则一致。

第六步是确定目标脑区和通道布局。fNIRS 的关键不是探头本身，而是 source-detector 之间形成的 channel 覆盖了哪个脑区。设计时应先查文献明确目标脑区，再安排头帽孔位和通道组合。脑激活研究可以重点覆盖假设脑区；脑网络研究应覆盖多个相关脑区；超扫描研究还要考虑每个被试的探头数量、多人同步和对应通道。

第七步是决定 ROI。ROI 最好在采集前或正式统计前按文献、解剖位置和实验假设定义，不建议看完显著结果后再临时圈 ROI。ROI 的目的有两个：降低 channel 层面的多重比较压力；让结果更容易用脑区功能解释。

第八步是规划样本和统计。至少要明确被试分组、组内/组间设计、条件数量、每个条件重复次数、主要统计检验、是否需要协变量、是否需要多重比较校正。脑激活常见统计单位是被试 × 条件 × ROI/channel 的指标值；脑网络常见统计单位是被试 × 连接边或图论指标；超扫描常见统计单位是 dyad × 条件 × ROI-pair/frequency 的同步指标。

实验设计阶段的注意事项：不要只追求复杂任务，任务越复杂，越容易引入头动、理解差异和无效 trial；不要只覆盖一个你“觉得可能显著”的区域，要能用文献解释；不要忽略休息段，因为基线质量直接影响激活结果；不要在自然情景研究中完全放弃控制变量，至少要记录行为事件、时间线和可能的混杂因素。

实验设计阶段的最终输出应包括：研究问题与假设；任务流程图；每个条件的时长和重复次数；marker 规则表；通道布局图；ROI 方案；预处理计划；主要分析模块；统计计划；异常情况记录表。

## 预处理

预处理的目的，是把原始采集数据整理成可信的 HbO/HbR 时间序列，并为后续脑激活、脑网络、小波幅值和超扫描分析提供统一输入。预处理不是“点几个按钮”，而是一个质量控制过程：要确认数据是完整的、marker 是正确的、通道是可用的、噪声被合理处理、参数可以复现。

预处理的第一步是导入数据。打开 Nirspark 后，从主界面选择数据读取/导入功能，选择采集软件导出的 `.hcx` 文件。导入时如果软件提示已有项目或已有 `.spark` 文件，需要判断是否覆盖。选择覆盖相当于重新生成分析项目，适合你要从头处理；选择不覆盖相当于读取之前的处理结果，适合继续上次分析。

![导入数据与已有项目提示](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/05_04_26-59.jpg)

*图：导入 `.hcx` 数据与读取已有项目时的提示。*

第二步是核对导入信息。导入后立即查看文件名、采样率、数据长度和基本波形。数据长度要与实验设计一致。例如预期 5 个 block，每个 block 35 秒，加上 10 秒前基线和收尾时间，总长度就应接近设计时长。如果长度明显不符，先不要继续分析，应回到采集记录检查是否提前停止、导出错误或选错文件。

第三步是检查 marker。进入 Edit Marker，查看 marker list 中每类 marker 的 value、time 和 frame。value 表示 marker 类型，time 表示出现时间，frame 表示采样帧。核对方法是把 marker 时间与任务流程表逐一对应：第一个任务是否从预期时间开始，每个 block 间隔是否一致，是否有多余 marker，是否漏掉 marker。

![Edit Marker 界面](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/05_05_32-59.jpg)

*图：Edit Marker 中查看、添加、修改、删除 marker 的界面。*

第四步是编辑 marker。若需要新增 marker，点击 Add，输入 marker value 和 time，软件会自动计算 frame；若某个 marker 时间错了，选中该项后用 Modify 修改 time；若多打了 marker，用 Delete 删除。注意删除前要确认是删除某一条 marker 还是某一类 marker。编辑完成必须点击保存并退出，直接关闭窗口可能不会保存。

第五步是导出 marker 表进行核对。课程中提到可以用保存/导出功能把 marker 情况导成表格。建议每个被试都保留一份 marker 检查记录，尤其是 block 设计或多条件任务。导出的表格可用于确认每个任务段是否按计划出现，也方便后续排查异常数据。

第六步是查看数据质量。进入数据质量查看模块，观察原始光强和血氧曲线。重点看是否存在光强过低、饱和、明显跳变、长时间漂移、突发尖峰、周期性异常、某些通道完全无信号、任务期间严重头动等。课程强调，采集软件中实时看到的曲线可能已经经过处理，不能替代 Nirspark 中对原始数据的质量检查。

![数据质量查看示例](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/06_03_18-39.jpg)

*图：数据质量查看中对原始信号、波形和通道状态的检查。*

第七步是进行质控判断。质控的目的不是让所有数据都“看起来好看”，而是决定哪些通道可信、哪些通道需要剔除、哪些被试可能不适合进入统计。质控结果应记录：剔除通道编号、剔除原因、是否剔除被试、保留通道比例、使用的阈值或判断标准。

第八步是定义 ROI。进入 ROI 定义模块，根据通道布局和目标脑区选择相应 channel，命名 ROI，例如 left M1、right M1、left PFC、right PFC 等。ROI 命名要清楚，最好与论文中的脑区名称一致。定义完成后保存 ROI 文件，后续个体分析和批处理都可以复用。

![ROI 定义示例](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/07_02_08-40.jpg)

*图：ROI 定义与通道选择界面。*

第九步是执行预处理。常见顺序是时间段提取、运动伪迹识别与校正、滤波、血氧转换。时间段提取用于去除采集前后无关片段，但不能破坏 marker 与任务的相对位置；运动伪迹校正用于处理头动引起的突变；滤波用于去除低频漂移和高频生理噪声；血氧转换用于把光强变化转换为 HbO/HbR 浓度变化。

![预处理步骤与参数界面](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/08_04_25-13.jpg)

*图：预处理流程中运动伪迹校正、滤波与血氧转换相关界面。*

第十步是检查预处理结果。预处理完成后不要立刻进入统计，先查看 HbO/HbR 波形是否合理。合理不等于“完全平滑”，而是漂移、尖峰和明显伪迹得到改善，同时任务相关变化仍然保留。若滤波后波形过度平滑或任务变化被削弱，说明参数可能过强。

预处理的重要注意事项：所有被试尽量使用同一套参数；不能为了显著性反复调整滤波和伪迹参数；如果某个被试使用特殊处理，要记录原因；如果删除某段数据，必须确认 marker 和 block 提取不会错位；如果任务段内有严重伪迹，单纯截掉中间一段可能会破坏 block 长度，宁可剔除该 block 或被试。

预处理阶段的最终输出应包括：导入后的项目文件；marker 检查表；质控记录；ROI 文件；预处理参数文件；预处理后的 HbO/HbR 数据；异常处理说明。

## 脑激活

脑激活分析的目的，是估计任务或条件相对于基线引起的 HbO/HbR 变化，并比较这种变化在组别、条件、通道或 ROI 之间是否不同。脑激活分析的关键是 marker、baseline、任务时间窗和指标提取。课程中主要使用 block average 和 GLM 两条路线。

脑激活分析第一步是选择方法。若任务是规则 block，例如“任务 15 秒 + 休息 20 秒”重复多次，优先使用 block average；若任务是事件相关、条件较多、刺激间隔不完全一致或需要用模型估计每个条件效应，可使用 GLM。两者都需要先完成预处理和 marker 核对。

第二步是进入 block average 模块并加载数据。选择一个已经完成预处理的被试数据，先做单个被试分析，不要一开始就批处理。单个被试跑通后，保存 condition、feature 和 ROI 参数，再用于其他被试。

第三步是定义 condition。点击 condition 的 edit/add，输入 condition 名称，选择对应 marker，例如 Mark 1。设置 block range，例如 0-35 秒，代表从 marker 开始向后提取 35 秒。设置 baseline time window，例如 -2 到 0 秒，代表用任务开始前 2 秒作为基线。若有多个条件，例如左手、右手、想象、真实运动，需要分别定义多个 condition。

![block average 的 condition 设置](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/09_02_13-22.jpg)

*图：定义 condition、选择 marker、设置 block range 和 baseline。*

第四步是定义 feature。feature 是最终要导出并统计的指标。点击 feature 的 edit/add，输入指标名称，选择对应 condition，选择指标类型。常用指标是 mean，因为它代表某个时间窗内 HbO/HbR 的平均变化，解释最直观。设置 main time window，例如 0-15 秒或 5-15 秒；如果血氧反应有延迟，可以避免只取刺激刚开始的几秒。

![feature 指标设置](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/09_03_21-00.jpg)

*图：设置 feature 名称、指标类型、任务时间窗和 baseline 时间窗。*

第五步是运行计算。点击运行后，软件会把每个 block 按 marker 对齐并叠加平均，得到每个 channel/ROI 的平均波形和 feature 值。计算完成后先查看波形，再查看表格。理想情况下，任务开始后 HbO 延迟上升，任务结束后逐渐回落；HbR 可能下降或呈相反变化。

![block average 结果与波形](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/09_05_36-16.jpg)

*图：block average 后得到的任务相关波形与通道结果。*

第六步是查看结果表。结果表通常包含 channel、ROI、feature 值等信息。确认每个 channel/ROI 都有结果，确认指标单位和时间窗正确。若某些通道结果异常极端，回到质控检查该通道是否应剔除。

第七步是保存参数文件。保存 condition、feature、ROI 和分析参数。后续批处理时直接加载这些参数，保证所有被试使用同一分析逻辑。不要每个被试手动重新设置一遍，否则容易产生不一致。

第八步是批处理。进入 block average 批处理模块，选择所有被试数据，加载已经验证过的参数文件，设置输出目录并运行。批处理前至少确认 1-2 个被试单独跑通。批处理后检查是否每个被试都成功生成结果，失败被试要单独打开排查。

![block average 批处理界面](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/10_03_29-53.jpg)

*图：批处理时加载参数并对多个被试统一计算。*

第九步是 GLM 分析。进入 GLM 模块后，加载预处理数据，定义条件和 marker，设置模型参数，选择 HbO/HbR 和 channel/ROI，运行模型并导出 beta、t 值或相关指标。GLM 的优势是能够更系统地建模多个条件和事件，但模型设置必须与任务设计一致。

![GLM 分析参数与设计矩阵](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/11_03_31-22.jpg)

*图：GLM 分析中条件、模型和参数设置。*

第十步是群组统计。把每个被试导出的 feature 或 GLM 指标整理成统计表。若比较同一被试两个条件，用 paired t-test 或 repeated measures ANOVA；若比较两组被试，用 independent t-test 或 mixed ANOVA；若有两个因素，例如组别 × 条件，用 two-way ANOVA。统计前明确因变量是 channel-level 还是 ROI-level。

![脑激活群组统计示例](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/12_02_17-30.jpg)

*图：脑激活结果进入群组统计与可视化。*

第十一步是统计假设检查。做参数检验前，检查正态性、方差齐性和异常值。样本量较小时，不要机械地只看 p 值，应结合直方图、Q-Q 图、箱线图和原始散点。若假设明显不满足，可以考虑非参数检验、稳健统计或减少不必要的多重比较。

![正态性与方差齐性检查](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/16_04_29-42.jpg)

*图：统计假设检验和自定义绘图相关界面。*

第十二步是绘制结果图。常见图包括激活地形图、波形图、ROI 柱状图、散点图、组间比较图。地形图适合展示空间分布，波形图适合展示血氧反应是否符合任务时间线，柱状图/散点图适合展示统计比较。所有图都应标明 HbO/HbR、时间窗、条件、ROI/channel 和统计结果。

脑激活分析注意事项：baseline 必须来自真实休息或任务前稳定段；任务时间窗不能随意改到最显著的位置；不要只报告显著 channel，要报告选择 channel/ROI 的依据；不要忽视 HbR，至少说明为什么只分析 HbO；多通道统计要考虑多重比较；如果波形不符合血氧反应规律，显著结果也要谨慎解释。

脑激活分析最终输出应包括：每个被试的 feature/GLM 指标表；批处理日志；群组统计表；激活地形图；波形图；统计图；方法参数记录。

## 脑网络

脑网络分析的目的，是评估不同 channel 或 ROI 之间时间序列的协同变化，即 functional connectivity。它适合静息态、连续任务、自然观看、互动任务和较长时间窗数据。与脑激活不同，脑网络通常不关心某个 marker 后几秒是否上升，而关心一段时间内不同区域的相关关系。

脑网络分析第一步是确定数据类型和时间窗。选择已经预处理后的 HbO、HbR 或 HbT 数据。静息态应选择稳定静息段；任务态网络应选择明确任务阶段；自然情景应根据实验事件切分时间窗。时间窗太短会导致相关估计不稳定。

第二步是选择分析层级。channel-level 分析保留更多空间细节，但连接数量很多，多重比较压力大；ROI-level 分析更简洁，解释更稳定，但会损失部分局部信息。若课程或论文目标是快速理解网络差异，建议先做 ROI-level，再根据需要补充 channel-level。

第三步是进入 functional connectivity 模块，加载数据并设置参数。选择 correlation 或软件支持的连接方法，选择 HbO/HbR，选择 channel/ROI，设置时间窗。若使用 Pearson correlation，后续群组统计前建议进行 Fisher Z transformation。

![个体功能连接分析入口](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/17_02_15-34.jpg)

*图：个体 functional connectivity 分析的设置入口。*

第四步是生成个体连接矩阵。矩阵的每一行和每一列代表一个 channel 或 ROI，矩阵中的数值代表两者之间连接强度。检查矩阵是否对称，是否存在异常高或异常低的连接，是否有缺失通道。

![功能连接矩阵与网络图](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/17_03_24-28.jpg)

*图：功能连接结果可视化，包括矩阵和连接图。*

第五步是保存个体结果。保存连接矩阵、网络图和参数文件。个体结果用于后续批处理和群组统计，也可以用于检查某个被试是否异常。

第六步是批处理。选择所有被试数据，加载统一的连接分析参数，批量生成连接矩阵。批处理前要确认每个被试的 ROI 定义一致、有效通道足够、时间窗一致。若不同被试缺失通道不同，需要记录并决定如何处理。

第七步是群组统计。比较两组或两个条件的连接差异时，可以对每条连接边进行统计，也可以先提取某些理论相关连接或 ROI-pair。连接边数量很多时必须注意多重比较；不建议从全矩阵里只挑显著边事后解释。

![脑网络批处理与群组统计](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/18_04_46-37.jpg)

*图：脑网络批处理和群组统计结果表格。*

第八步是图论分析。图论分析基于连接矩阵计算网络拓扑指标，例如 global efficiency、local efficiency、clustering coefficient、path length、degree、centrality 等。进入图论模块前必须确定阈值策略、加权/二值网络、正负连接处理方式和节点定义。

![图论分析与统计设置](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/19_03_28-20.jpg)

*图：图论指标计算、阈值和统计设置示例。*

第九步是解释网络结果。功能连接代表统计相关或同步，不代表解剖连接，也不代表因果影响。解释时应结合任务背景、脑区功能和理论模型。例如“前额叶-运动区连接增强”应说明它可能反映认知控制与运动执行协同，而不是直接说两个脑区产生了因果连接。

脑网络分析注意事项：静息态要保证被试状态一致；时间窗不能过短；HbO 更敏感但也更容易受系统性生理噪声影响；连接矩阵统计要考虑多重比较；图论阈值会显著影响结果，必须报告；不同被试通道缺失会影响矩阵可比性。

脑网络分析最终输出应包括：个体连接矩阵；批处理矩阵；ROI/channel 对应表；群组统计表；网络图；图论指标表；阈值和统计参数记录。

## 小波幅值

小波幅值分析的目的，是从 fNIRS 时间序列中提取特定频段或时间-频率范围内的 amplitude 特征。它适合研究低频振荡、任务相关能量变化、临床组别差异或与行为/量表的相关关系。它不是 block average 的替代品，而是从频域或时频角度补充解释信号。

小波幅值分析第一步是明确频段。频段不能随意选，应结合文献和生理意义。fNIRS 信号中可能包含神经活动、Mayer wave、呼吸、心跳和系统性血流波动。若频段选得不合理，结果可能主要反映生理噪声。

第二步是准备输入数据。使用已经完成预处理和质控的数据。若数据中仍有明显运动伪迹，小波结果会出现强烈异常幅值。低频分析尤其需要足够长的数据，短任务段不适合解释很低频波动。

第三步是进入 wavelet amplitude 模块。选择数据类型 HbO/HbR，选择 channel 或 ROI，选择分析时间段，设置频段或尺度参数，选择输出指标和结果保存目录。

![小波幅值分析入口](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/20_02_15-59.jpg)

*图：小波幅值分析模块的参数入口。*

第四步是设置频段、时间窗和空间范围。空间范围可以是全通道，也可以是特定 ROI。时间窗应与研究问题一致，例如任务阶段、休息阶段或整段静息态。频段设置后要记录具体上下限，后续论文方法部分必须写清楚。

![小波幅值参数设置](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/20_03_25-08.jpg)

*图：小波幅值分析中的频段、通道/ROI 和时间窗设置。*

第五步是运行并查看结果。查看时间-频率图、幅值图或导出表格。先检查是否有极端异常区域，再看条件或组别差异。若某个被试所有通道都出现异常高幅值，应回到预处理和原始波形检查。

![小波幅值结果与统计](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/20_04_34-16.jpg)

*图：小波幅值结果查看与后续统计示例。*

第六步是导出指标并统计。导出的表格应包含被试编号、组别、条件、channel/ROI、频段、时间窗和 amplitude 指标。统计方法与脑激活类似，可进行组间比较、条件比较或与量表/行为指标相关。

小波幅值分析注意事项：不要对过短数据解释极低频；不要忽略运动伪迹对时频结果的放大效应；频段选择要有文献依据；多个频段和多个 ROI 同时比较时要考虑多重比较；结果解释应写成“某频段幅值变化”，不要直接等同于“脑激活增强”。

小波幅值分析最终输出应包括：频段设置记录；channel/ROI 设置；时间窗设置；幅值结果表；时频图；统计表；与行为或量表的相关分析结果。

## 超扫描

超扫描分析的目的，是研究两人或多人互动过程中脑活动的同步性。课程中的关键流程是：多人同步采集，数据拆分，SD 变换，分别预处理，时间轴对齐，选择配对 channel/ROI，计算 wavelet coherence 或其他同步指标，最后进行真实互动与控制条件比较。

超扫描第一步是设计互动任务。任务必须真的包含互动，例如合作、竞争、对话、亲子互动、医患沟通等。若两个人只是同时看同一个刺激，结果可能是共同刺激引起的同步，不一定是互动同步。因此应设置控制条件，例如非互动、随机配对、错位时间窗或独立任务。

第二步是规划同步采集。需要明确使用一台设备还是多台设备，多人 marker 是否同步，是否有同步装置，每个人的探头数和通道数是否足够，时间轴是否能对齐。采集时要记录每个互动阶段的开始和结束。

第三步是进行数据拆分。多人同步采集的数据需要拆成每个被试可单独识别的数据。拆分时要确认每个人的通道、source-detector 信息、marker 和时间轴没有错位。

![超扫描数据拆分与 SD 变换](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/21_03_21-58.jpg)

*图：超扫描中多人数据拆分和 SD 变换相关步骤。*

第四步是进行 SD 变换。SD 变换的目的，是让拆分后的数据具有正确的 source-detector/channel 结构，便于后续预处理和脑区定位。变换后要检查通道数、探头对应关系和头帽布局是否正确。

第五步是分别预处理每个被试。每个人的数据都要完成普通 fNIRS 预处理：marker 检查、数据质量查看、质控、运动伪迹校正、滤波和血氧转换。不要因为后续做同步分析就跳过个体质控；任何一个人的严重伪迹都会污染两人同步指标。

第六步是设置配对关系。确定谁和谁是一对，哪些 channel/ROI 之间要计算同步。可以选择同源脑区配对，例如两个人的 left PFC 对 left PFC；也可以根据理论选择跨脑区配对。配对方案应在分析前确定。

第七步是进入 wavelet coherence 分析模块。选择两个人的数据，选择 HbO/HbR，设置 channel/ROI pair，设置频段和时间窗，选择输出指标并运行。频段选择同样需要文献依据和生理解释。

![超扫描小波相干分析设置](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/22_03_18-58.jpg)

*图：wavelet coherence 分析前的配对、频段和时间窗设置。*

第八步是查看小波相干结果。结果通常显示不同时间和频率上的同步强度。重点不是“哪里颜色最强”，而是同步增强是否出现在理论预期的互动阶段、频段和脑区配对上。

![小波相干结果图](https://raw.githubusercontent.com/joy-psy/nirspark-feishu-assets/main/nirspark-assets/22_04_25-52.jpg)

*图：超扫描小波相干结果与同步区域显示。*

第九步是做统计比较。常见比较包括真实互动 vs 非互动，合作 vs 竞争，亲密关系 vs 陌生配对，真实 dyad vs pseudo dyad，任务阶段 vs 休息阶段。统计单位通常是 dyad，而不是单个个体。结果表应包含 dyad 编号、条件、ROI-pair、频段、时间窗和 coherence 指标。

第十步是解释结果。超扫描结果应解释为 inter-brain synchrony 或人际脑同步，不要直接说“两个大脑连接在一起”。同步可能来自真实互动，也可能来自共同刺激、共同动作、共同节律、呼吸/心率同步或设备噪声。必须通过控制条件和统计设计尽量排除这些混杂。

超扫描分析注意事项：采集阶段要保证同步；拆分后要检查每个人的数据结构；预处理参数要一致；配对关系要明确；频段和 ROI-pair 不要事后随意挑选；统计单位通常是 dyad；必须设置控制条件；结果解释要避免过度因果化。

超扫描分析最终输出应包括：互动任务时间线；同步方式记录；拆分后的个体数据；SD 变换记录；个体预处理结果；配对表；wavelet coherence 结果；真实配对和控制配对统计表；同步图和解释说明。

## 从数据到论文结果的整理方式

完成软件分析后，建议把所有导出结果整理成统一的项目表。每行表示一个被试、一个条件或一个 dyad；每列表示组别、条件、ROI/channel、指标值、时间窗、频段、统计变量和备注。不要直接把软件输出的多个表格零散保存，否则后期很难复现。

论文方法部分至少写清：设备与采样率；被试与分组；任务范式；通道布局和 ROI；marker 规则；预处理流程；质控标准；分析模块；指标定义；统计方法；多重比较策略。结果部分至少写清：HbO/HbR；脑区或连接；时间窗或频段；效应方向；统计量；p 值；图形对应关系。

最小可复现记录表应包括：被试编号；组别；任务条件；采集日期；数据时长；marker 情况；异常记录；剔除通道；剔除被试；预处理参数；ROI 文件；block/GLM/network/wavelet 参数文件；导出指标文件；统计脚本或统计软件设置；最终图表版本。

## 常见错误与检查清单

常见错误一：先采数据再决定分析。检查方法：采集前是否已经写明主要分析模块、主要指标和统计方法。

常见错误二：marker 没核对就做脑激活。检查方法：每个被试是否有 marker 表，marker 时间是否与任务流程一致。

常见错误三：只看显著性不看波形。检查方法：脑激活结果是否同时展示 block average 波形和统计图。

常见错误四：把通道当探头。检查方法：通道布局说明中是否写清 source-detector 形成的 channel 覆盖脑区。

常见错误五：每个被试参数不一致。检查方法：是否保存并复用同一套 ROI、预处理和分析参数。

常见错误六：脑网络结果解释成因果。检查方法：论文中是否把 functional connectivity 写成统计相关/同步，而不是解剖连接或因果连接。

常见错误七：超扫描没有控制条件。检查方法：是否设置非互动、随机配对、错位时间窗或其他控制分析。

常见错误八：图片漂亮但方法不透明。检查方法：每张图是否能追溯到数据类型、时间窗、ROI/channel、指标、统计方法和参数文件。
