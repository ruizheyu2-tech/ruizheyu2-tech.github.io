- df017_vecHEP.root 这个文件以及同名脚本可以画出高能粒子能量分布图（有筛选）

- input_histos_rf_lagrangianmorph.root这个文件里一堆TFolder文件

- stock.root 这个文件是诺贝尔经济学奖马科维茨模型，跟物理没关系，纯炫技来的

- mlpHiggs.root 这个文件是一个用于训练机器学习模型（神经网络）的物理模拟数据集，教计算机如何在复杂背景噪声中认出希格斯玻色子

- ntprndm.root + ProofEvent.C 并行计算，适用于多核CPU或计算机集群

- machine_learning这个文件夹中全是机器学习的内容

- gallery.root全是TASImage文件

- beta+.root 这个文件演示了beta+衰变，释放一个正电子和一个中微子，是质子转化为中子的过程，这里边第一个图是正电子动能谱，第二个图是中微子能谱，第三个是衰变总动能

	- $p \rightarrow n + e^+ + \nu_e$
	
	- 可以通过端点能量猜测是哪种同位素
		- 横轴 (X-axis)：正电子的动能，代表衰变出来的 **正电子 ($e^+$) 飞出来时的能量大小**。  
			- 动能的最大值是端点能量 
		- 纵轴 (Y-axis)：相对概率 / 强度 (%)，代表 **“发射出具有该特定能量粒子的概率（可能性）”**。

- beta-.root 这个文件演示了beta-衰变，这是原子核内的中子转化为质子的过程，同时释放出一个电子和一个反中微子。

	- $n \rightarrow p + e^- + \bar{\nu}_e$

- ecapt.root 这个文件是电子俘获的模拟数据

	- $p + e^- \rightarrow n + \nu_e$
	
	- 原子核向内吃掉一个电子，不会有正电子或者电子出来，所以没有电子动能谱
	
	- 在电子俘获中，因为是**二体衰变**（如果不考虑极小的原子核反冲），中微子会带走固定的能量，它是一个**尖锐的峰（单能谱 / Mono-energetic peak）**，而不是一个宽宽的分布。

- it1.root 这个文件是同质异能跃迁 (Isomeric Transition, 简称 IT)。通过发射一个 **$\gamma$ 光子（Gamma Ray）**，把多余的能量释放出来，让自己落回到基态或较低的能级。

	- 第一个图是$\gamma$ 射线的能谱。不同于 $\beta$ 衰变的连续谱（馒头形），$\gamma$ 跃迁的能量是量子化的。
	- 第二个图是衰变链的总寿命 / 衰变时间分布，是一个**指数衰减 (Exponential Decay)** 的分布图。通过拟合这张图，可以算出这个核的半衰期 ($T_{1/2}$)。

```
// 1. 定义拟合函数
// [0] 是常数 A (幅度)
// [1] 就是我们要找的半衰期 T_half
// 我们选择拟合范围为 4.0 到 15.0 (避开左边的上升区)
TF1 *f1 = new TF1("f1", "[0]*exp(-0.693147*x/[1])", 4.0, 15.0);

// 2. 设置参数的初始猜测值 (这一步很重要，否则拟合可能不收敛)
// 看着图猜一下：高度大概 5000，半衰期看着像 2-3 ps
f1->SetParameter(0, 5000); 
f1->SetParameter(1, 2.0);  

// 3. 执行拟合
// "R" 表示只在我们在 TF1 中定义的范围 (4.0 - 15.0) 内拟合
h2->Fit("f1", "R");

// 4. 在画布上显示统计信息框，方便看结果
gStyle->SetOptFit(1111);
```

- 拟合半衰期

```
root [15] TH1D* h2 = (TH1D*)file11->Get("8")
(TH1D *) 0x5e19696f2220
root [16] h2->Draw("HIST")
Info in <TCanvas::MakeDefCanvas>:  created default TCanvas with name c1
root [17] TF1 *f1 = new TF1("f1", "[0]*exp(-0.693147*x/[1])", 2.0, 15.0);
root [18] f1->SetParameter(0, 5000); 
root [19] f1->SetParameter(1, 2.0);
root [20] h2->Fit("f1", "R");
****************************************
Minimizer is Minuit2 / Migrad
Chi2                      =      498.069
NDf                       =           63
Edm                       =  5.44335e-07
NCalls                    =           74
p0                        =      17148.5   +/-   155.356     
p1                        =      1.28375   +/-   0.00493083  
root [21] h2->Draw("SAME")
root [22] gStyle->SetOptFit(1111);
```

- it2.root 这个文件描述的是内转换（Internal Conversion, IC）。

	- 当原子核处于激发态时，它想把能量释放出来，通常它会发射 Gamma 射线（像 it1 那样）。但在某些情况下，原子核会选择“直接把能量传给核外的电子（通常是 K 层或 L 层电子）”把这个电子一脚踢飞出去，这个被踢出去的电子就叫内转换电子 (Conversion Electron)
	
	- 这个过程和发射 Gamma 射线是**竞争关系**（Competing Process）。
		- **`TH1D 3 (Gamma)`**：记录了那些没被转换，直接作为光子飞出来的能量。
		- **`TH1D 1 (Electron)`**：记录了那些通过内转换飞出来的**单能电子**。
		- **`TH1D 8 (Life)`**：依然记录了这个激发态活了多久（半衰期）。

- it3.root 这个文件描述的是伴随有伽马（Gamma）辐射的 Alpha 衰变。

	- d第一个图**Alpha 衰变**是指原子核吐出一个 **氦原子核 ($\alpha$ 粒子, 即 2个质子+2个中子)** 的过程。
	
    - 反应式：$^A_Z X \rightarrow ^{A-4}_{Z-2} Y + \alpha$
    
	- 都是**离散谱（Discrete Spectrum）**，也就是**尖锐的峰**：
		- **Alpha 谱 (`TH1D 4`)**：你应该会看到一个或几个尖峰（通常在 4-6 MeV 范围）。这代表 $\alpha$ 粒子的能量是量子化的，不是乱飞的。
		- **Gamma 谱 (`TH1D 3`)**：同样是尖峰，对应子核能级之间的能量差。

- geo.root 这个文件是用来存储 **探测器几何结构（Detector Geometry）** 的。

	- 里边有一个`ExP02GeoTree` 文件，保存了实验装置的“**3D 蓝图**”。 它记录了探测器长什么样、用了什么材料、放在哪里、有多大。通常在物理模拟（如 Geant4 或 ROOT 模拟）中，我们不需要每次都重新写代码构建探测器，而是把构建好的探测器保存进这个 `.root` 文件，下次直接读取即可。

- ref_6.16_example_UsingC_combined_meas_model.root 这个文件是 **RooStats / HistFactory 统计建模** 的结果文件。

    - HistFactory 是 ROOT/RooStats 里专门用来把 **直方图 (Histograms)** 转换成 **RooFit 概率密度函数 (PDFs)** 的工具。
    
    - **RooWorkspace** 是 RooFit 的容器，它把所有的数学定义（公式、变量）、数据（Data）、参数（Parameters of Interest）都封装在了一起。
    
    - **“combined”** 这个名字通常暗示这是一个 **联合分析 (Combined Analysis)**，可能把多个信号区域（Signal Regions）或控制区域（Control Regions）结合在了一起。
    
	- `KEY: TDirectoryFile channel1_hists;1 channel1_hists`存放的是构建模型所需的**原始直方图**。
	    - HistFactory 会读取这里的直方图（比如 `h_signal`, `h_background`, `h_data`），考虑形状误差（Shape Systematics），然后生成 Workspace 里的数学函数。
	
	- 应用场景
		1. **上游（输入）**：物理学家跑完分析代码，画出了信号区和控制区的直方图。
		2. **中间（本文件）**：HistFactory 把这些直方图吃进去，加上系统误差（Systematics），打包生成了这个 `combined_meas_model.root`。
		3. **下游（输出）**：统计学家（或你自己）会读取这个文件里的 `RooWorkspace`，然后运行 RooStats 工具来回答物理问题：
		    - **假设检验**：我们要找的粒子存在吗？（计算 p-value / Significance）。
		    - **参数测量**：这个粒子的截面是多少？（Likelihood Fit）。
		    - **设置上限**：如果没找到，最多能排除多少质量范围？（CLs Limits）。
```
TFile* file12 = new TFile("文件名.root")
RooWorkspace* w = (RooWorkspace*)file2->Get("combined")
w->Print()
```
