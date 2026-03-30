## 捡鸡蛋
一筐鸡蛋：1个1个拿，正好拿完。2个2个拿，还剩1个。3个3个拿，正好拿完。4个4
个拿，还剩1个。5个5个拿，还差1个。6个6个拿，还剩3个。7个7个拿，正好拿完。8个8个拿，还剩1个。9个9个拿，正好拿完。问筐里有多少鸡蛋？
```
#include <stdio.h>
void takeegg(int N)
{
  int Num = 0;
  if(N%2 == 1) Num++;
  if(N%3 == 0) Num++;
  if(N%4 == 1) Num++;
  if(N%5 == 1) Num++;
  if(N%6 == 3) Num++;
  if(N%7 == 0) Num++;
  if(N%8 == 1) Num++;
  if(N%9 == 0) Num++;
  if(Num == 8) printf("N=%d\n",N);
}
int egg()
{
  int N = 1;
  int i = 0; 
  for(i = 1;i < 20000;i++)
  {
    N = i*9;
    takeegg(N);
  }
  return 0;
}
```
- 在输入运行时，要运行egg()也就是主函数，这样就会自动调用takeegg()函数
## 高斯分布直方图
### 不能改参数
```
#include "TH1F.h"
#include "TPad.h"

void DrawGaussian()
{
  TH1F *h = (TH1F*)gROOT->FindObject("h_gaus");
  if(h){h -> Reset();}
  else{h = new TH1F("h_gaus","Une gaussienne",100,0.,100.);}
  h->SetFillColor(17);
  h->Draw("BAR");
  h->SetLineColor(2);
  h->SetLineWidth(2);
  h->Draw("HIST");
  
  for (double x=0; x<100; x++)
  {
    double f = 20. * exp(-pow(x-50.,2)/2./pow(10,2));
    h->Fill(x,f);
    gPad->Modified();
    gPad->Update();
  }
}
```
### 能改参数
```
#include "TH1F.h"
#include "TPad.h"

void DrawGaussian1(double amp = 20, double moy = 50, double large = 10)
{
  TH1F *h = (TH1F*)gROOT->FindObject("h_gaus");
  if(h){h -> Reset();}
  else{h = new TH1F("h_gaus","Une gaussienne",100,0.,100.);}
  h->SetFillColor(17);
  h->Draw("BAR");
  h->SetLineColor(2);
  h->SetLineWidth(2);
  h->Draw("HIST");
  
  for (double x=0; x<100; x++)
  {
    double f = amp * exp(-pow(x-moy,2)/2./pow(large,2));
    h->Fill(x,f);
    gPad->Modified();
    gPad->Update();
  }
}
```
### 可以在脚本外调整画板
```
#include "TH1F.h"
#include "TPad.h"

double Gaussian(double a, double b, double c, double d)
{
  double e = b * exp(-pow(a-c,2)/2./pow(d,2));
  return e;
}

TH1F* DrawGaussian2(double amp = 20, double moy = 50, double large = 10)
{
  TH1F *h = (TH1F*)gROOT->FindObject("h_gaus");
  if(h){h -> Reset();}
  else{h = new TH1F("h_gaus","Une gaussienne",100,0.,100.);}
  
  for (double x=0; x<100; x++)
  {
    double e = Gaussian(x, amp, moy, large);
    h->Fill(x,e);
  }
  return h;
}
```
- 不用全局变量而是分别定义局部变量的原因
	- 局部变量（函数内）：函数跑完，变量自动销毁，内存自动释放
	- 全局变量（函数外）：只要 ROOT 不关闭，这块内存就一直被占用，里面的旧数据也一直留着。下次运行如果忘了清零，旧数据会污染新计算。
- 运行后的输入内容
	- ```
	  root[0] .L fillhisto_2.C
	  root[1] TH1F *h12 = DrawGaussian2();//";"可加可不加，加上没有结果那一行
	  root[2] h12->Draw();
	  ```
	- 要重新定义一个指针，这样可以在脚本外绘图
	- 可以用`SetLineColor`或者`SetFillColor`修改直方图的轮廓和填充色
	- 轮廓`Draw(“HIST”)`  柱状图`Draw("BAR")` 
## 绘制光的干涉图样![[Pasted image 20251229114358.png]]
- `double single(double *x, double *par) {return pow(sin(pi*par[0]*x[0])/(pi*par[0]*x[0]),2);}`
	- x和par两个指针，调用的时候要写[n]调用第几个数据
	- x是变量，par是参数
	- `TF1 *Fnslit = new TF1("Fnslit",nslit,-5.001,5.,2);`
		- `TF1`决定了只有一个变量x
		- `2`决定了有两个参数`par[0]`和`par[1]`
	- `double`是返回什么类型的数值，是输出，括号内是输入，主函数不用返回数值的话就是`void`，后边的函数也只是控制台而已不用输入，所以括号内是空的
	- 写-5.001是为了避开x=0的点防止程序崩溃
- 首先分别用两个函数计算单缝衍射因子和多缝干涉因子，再在nslit函数中合并
	- 主函数调用上边已经定义的函数
		- 先输入两个参数par[0]和par[1]，但是这里要赋值给另外两个变量，因为没有定义可画图的一维函数fnslit
		- 定义fnslit时，只有一个变量x，通过设定取值范围来给数值，后边的2代表有两个不确定参数
		- 再把输入的参数值赋值给不确定参数
		- 绘图

## 把.dat文件转换为.root文件
```
#include "Riostream.h" // 引入 ROOT 的输入输出库（类似于 C++ 的 iostream）
#include "TFile.h"     // 引入处理 ROOT 文件的类
#include "TH1.h"       // 引入一维直方图的类
#include "TNtuple.h"   // 引入 Ntuple（简单数据树）的类

// 定义函数 basic，参数有两个：
// 1. fdata: 输入的文本文件名（默认叫 "basic.dat"）
// 2. froot: 输出的 ROOT 文件名（默认叫 "basic.root"）
void basic(const Char_t* fdata="basic.dat", const Char_t* froot="basic.root")
{
   // 创建一个输入文件流对象，名字叫 in
   ifstream in; 
   
   // 使用 open 函数打开文本文件。
   // ios::in 表示以“读取模式”打开。
   in.open(fdata, ios::in);
   
   // 声明三个浮点数变量 x, y, z。
   // 它们是“搬运工”，用来暂时存放从文件里读出来的每一行数据。
   Float_t x, y, z;
   
   // 声明一个整数变量 nlines，初始化为 0。
   // 用来统计我们到底读了多少行数据。
   Int_t nlines = 0;

   // 创建一个新的 TFile 对象（ROOT文件），指针叫 f
   // "RECREATE" 选项很重要：
   // 如果文件不存在，就创建；如果文件已经存在，就覆盖它（彻底重写）。
   TFile *f = new TFile(froot, "RECREATE");

   // 创建一个一维直方图，指针叫 h1。
   // "h1": 内部名字。
   // "x distribution": 图片标题。
   // 100: 把横轴切成 100 个格子。
   // -4, 4: 横轴范围从 -4 到 4。
   TH1F *h1 = new TH1F("h1", "x distribution", 100, -4, 4);

   // 创建一个 Ntuple（类似于一张 Excel 表），指针叫 ntuple。
   // "ntuple": 内部名字。
   // "data from...": 标题。
   // "x:y:z": 定义了这张表有三列，列名分别叫 x, y, z。
   TNtuple *ntuple = new TNtuple("ntuple", "data from ascii file", "x:y:z");
   
   while (in >> x >> y >> z) // 核心动作：从文件里提取三个数，依次放进变量 x, y, z 中。
   {
	   if (nlines < 5)
	   {
		   cout << "X = " << x << ", Y = " << y;
		   cout << ", Z = " << z << endl;
	   }
	   
	   // 【关键步骤 1】把 x 的值填入直方图 h1。
	   h1->Fill(x);
	   
	   // 【关键步骤 2】把 x, y, z 这一组数据填入表格 ntuple。
	   ntuple->Fill(x, y, z);
	   
	   // 计数器加 1。
	   nlines++;
	}
   }// 循环结束后，在屏幕上打印总共读了多少行。
   cout << "On a trouve " << nlines << " lignes." << endl;

   // 关闭输入文本文件（好习惯，释放资源）。
   in.close();

   // 【最重要的一步】
   // 之前的数据都还在内存里飘着，这一行命令会把内存里的直方图和 Ntuple 
   // 真正地写入到硬盘上的 .root 文件里。
   f->Write();

   // 删除文件指针 f。
   // 这不仅释放内存，实际上也起到了 "Close" 关闭文件的作用。
   delete f;
}
```
- `Int_t`和`int`没啥区别，都行，只不过前者适合Root，后者适合C++
- 记为basic.C文件，在共享文件夹中可以找到
- .root文件中创建了一个名叫h1的一维直方图文件和一个名叫ntuple的表格文件
- 要记得把文本文件跟代码文件放在同一个文件夹中，然后直接root basic.C打开，输出文件也会在同一个文件夹
- 运行结果![[Pasted image 20251230105844.png]]

## 拟合
### 基础拟合操作
```
gStyle->SetOptFit(kTRUE)
TH1F* h = new TH1F("hg","Un example de fit",100,-2,2)
h->FillRandom("gaus",10000)
h->Fit("gaus", "V", "E1", -1, 1.5)
h->Draw("E1")
```
- `SetOptFit(mode)` 的参数 `mode` 就像一个**开关组合**。 `kTRUE` 就是1的意思，但它实际上可以是一个由 **4 位数字** 组成的整数，每一位数字控制着统计框里的一项内容的显示。
	- 这个 4 位数字的格式是：**p c e v**

| **位置** | **代号** | **含义**                 | **设置说明**                                         |
| ------ | ------ | ---------------------- | ------------------------------------------------ |
| **千位** | **p**  | **P**robability (拟合概率) | `1` = 显示<br>`0` = 不显示                            |
| **百位** | **c**  | **C**hisquare (卡方/自由度) | `1` = 显示 Chi2/NDF<br>`0` = 不显示                   |
| **十位** | **e**  | **E**rrors (误差)        | `1` = 显示参数误差<br>`0` = 不显示                        |
| **个位** | **v**  | **V**alues (参数值)       | `1` = 显示参数值<br>`2` = 显示所有参数(包括被固定的)<br>`0` = 不显示 |
> **注意**：如果想显示误差 (`e=1`)，通常个位必须也是 `1`（因为没有参数值，光有误差没意义）。

- 常用组合示例
	- **`gStyle->SetOptFit(1111)`**：**全开模式**。显示概率、卡方、误差、参数值。
	- **`gStyle->SetOptFit(0111)`**：**标准模式**（也就是默认的 `1` 或 `kTRUE`）。显示卡方、误差、参数值，但不显示概率。
	- **`gStyle->SetOptFit(0001)`**：**极简模式**。只显示参数的值，不显示误差，也不显示卡方统计量。
	- **`gStyle->SetOptFit(1011)`**：**概率模式**。显示概率、误差、参数，但不显示卡方。
- `gStyle->SetOptFit(11);` 只显示参数值和误差
- `FillRandom("gaus",10000)` 随机生成一万个高斯分布的数据点
- `Fit(function, option, graphics_option, xmin, xmax)` 
	- h->Fit("gaus", "V", "E1", -1, 1.5)
		- `"V"` 是**拟合选项**（控制怎么算）
		- `"E1"` 是**绘图选项**（控制怎么画）。
	- 拟合选项 (Fit Options)
		- **A. 控制输出信息 (最常用)**
			- **`"Q"` (Quiet)**：**静默模式**。屏幕上不打印任何拟合结果（只有最后画图），和`"V"` (Verbose/详细模式) 刚好相反
				- 如果在循环里拟合 1000 次，一定要用这个，否则屏幕会被刷爆。
			- **`"V"` (Verbose)**：**详细模式**。打印每一步迭代的过程、参数变化等。
				- 拟合失败或者结果很奇怪时，用来调试。
	    - **B. 控制计算方法 (物理分析必看)**
		    - **`"L"` (Log Likelihood)**：使用**对数似然法**（Log Likelihood）而不是默认的卡方（Chi-square）最小化。
			    - 当直方图中某些 bin 的计数很少（比如 < 10）时，高斯近似（卡方）不再准确，必须用 `"L"` 才能得到正确结果。
			- **`"W"` (Weights)**：忽略直方图的误差，将所有 bin 的权重设为 1。
			    - 只想看数据的“形状”趋势，不关心统计误差时用
			- **`"M"` (Minos)**：运行 Minos 算法来改善误差分析。
			    - 它可以计算非对称误差（比如参数是 $5^{+0.2}_{-0.5}$），比默认的误差计算更精确，但速度慢
		-  **C. 控制绘图与范围**
			- **`"N"` (No draw)**：**只计算，不画图**。
			    - 只需要获取拟合参数（比如均值），不需要把图画出来给别人看。
			- **`"0"`**：同上，不画拟合曲线。
			- **`"+"`**：**保留上一次的拟合线**。
			    - 默认情况下，新的 Fit 会把旧的 Fit 线删掉。如果您想在一张图上比较两个拟合结果（比如一个高斯 vs 一个多项式），就要加这个。
			- **`"R"` (Range)**：使用函数自身定义的范围，而不是直方图的范围。
	- 绘图选项 (Graphics Options)
		- **`"SAME"`**：**最常用**。把拟合结果叠加在当前画布上，而不是擦除重画。
		- **`""` (空字符串)**：默认值。画出直方图以及拟合曲线。
		- **`"E"` 及其变体**（如您用的 `"E1"`）：主要是控制直方图误差棒的画法
		    - `"E"`：画误差棒。
		    - `"E1"`：画这种带垂直线和小横杠的误差棒（工字型）。
		    - `"E2"`：用矩形填充误差区域。
		- **`"C"`**：平滑曲线（Curve）。让拟合线看起来更圆滑。

### 获取拟合参数
```
TF1* gausfit = (TF1*)h->GetFunction("gaus")
gausfit->GetParameter(0)
gausfit->GetParameter(1)
gausfit->GetParaError(0)
double par[3]
gausfit->Getparameter(par)
```
- `GetParameter()` 括号内要是有具体的参数就拿出来，要是有空数组就填进去
### 拟合测试
```
TF1* ft = new TF1("f3","[0]*sqrt([1]*(sin([2]*x)+2.))/([3]+pow(x,2.))",0,10)
ft->SetParameters(1,1,3,5)
ft->Draw()

TH1F* hd = new TH1F("h2","Un example",100,0,10)
hd->FillRandom("f3",100000)
ft->SetParameter(hd->GetMaximum(),1,2.8,6.)
hd->Fit("f3")
```
- 上边半段是写了一个公式，自己设置参数，然后画出来
- 下边半段是用FillRandom按照自己设置的公式生成100000个数字，画出一个分布直方图
- 然后然后设置一个开始拟合的参考值
- 最后拟合，可以得到一个跟上半部分画出来的图一样的图

## Root TTree 初阶
### 生成tree.root文件
```
// tree.cc
void tree()
{ 
//常量声明
  const Double_t D = 500.;//cm, distance between target and the scin.(Center)
  const Double_t L = 100.;//cm, half length of the scin.
  const Double_t dD = 5.;//cm, thickness of the scin.
  const Double_t tRes = 1.;//ns, time resolution(FWHM) of the scintillator.
  const Double_t Lambda = 380.;//cm, attenuation lenght of the scin.
  const Double_t qRes = 0.1;//relative energy resolution(FWHM) of the scin. 
  const Double_t Vsc = 7.5;//ns/cm, speed of light in the scin.
  const Double_t En0 = 100;//MeV, average neutron energy
  const Double_t EnFWHM = 50.;//MeV, energy spread of neutron(FWHM)
  const Double_t Eg0 = 1;//MeV, gamma energy  
  const Double_t RatioGamma = 0.3;//ratio of gamma,ratio of neutron 1-Rg 

  //1. 声明tree中Branch的变量
  Double_t x;//入射位置
  Double_t e;//能量
  int pid;    //粒子种类，n:pid=1,g:pid=0
  Double_t tof, ctof;//TOF:粒子实际飞行时间，cTOF：计算得到的TOF
  Double_t tu, td;
  Double_t qu, qd;

  Double_t tuOff = 5.5;//time offset，
  Double_t tdOff = 20.4;//time offset 

 //2. 定义新ROOT文件，声明新的Tree 
  TFile *opf = new TFile("tree.root", "recreate");//新文件tree.root，指针 *opf
  TTree *opt = new TTree("tree0", "tree structure");//前边给电脑看的名字，后边给人看的名字

  //3. 将变量地址添加到tree结构中
  //创建一个名为"x"的列，读取x的值存入，数据类型Double
  opt->Branch("x", &x, "x/D");
  opt->Branch("e", &e, "e/D");
  opt->Branch("tof", &tof, "tof/D");
  opt->Branch("ctof",&ctof,"ctof/D");
  opt->Branch("pid", &pid, "pid/I");
  opt->Branch("tu", &tu, "tu/D");
  opt->Branch("td", &td, "td/D");
  opt->Branch("qu", &qu, "qu/D"); 
  opt->Branch("qd", &qd, "qd/D");
  
  // histogram，ROOT文件中除了TTree结构外，还可存储histogram，graph等
  TH1D *hctof = new TH1D("hctof", "neutron time of flight", 1000, 0, 100);
  TRandom3 *gr = new TRandom3(0);//声明随机数

  //4. 循环，计算变量的值，逐事件往tree结构添加变量值。
  for(int i = 0; i < 100000; i++){
    x = gr->Uniform(-L, L);//粒子入射位置，在-L,L范围内随机抽样（概率均相同）
    Double_t Dr = D + gr->Uniform(-0.5, 0.5) * dD;//粒子在探测器厚度范围内随机产生光信号
    Double_t d = TMath::Sqrt(Dr * Dr + x * x);//粒子实际飞行距离
    if(gr->Uniform() < RatioGamma) { //判断为gamma入射
       pid = 0;
       e = Eg0;
       tof = 3.333 * (d * 0.01);
    }
    else {  //neutron
        pid = 1;
        e = gr->Gaus(En0, EnFWHM/2.35); // energy of neutrons
        tof = 72.29824/TMath::Sqrt(e) * (d * 0.01);//ns
    }
    if(isnan(tof)) continue; //check ZeroDivisionError

    tu = tof + (L - x)/Vsc + tuOff;
    tu = gr->Gaus(tu, tRes/2.35);
    td = tof + (L + x) / Vsc + tdOff;
    td = gr->Gaus(td, tRes/2.35);
    ctof = (tu + td)/2.;//simplified calculation.
    hctof->Fill(ctof);
//neutron：energy of recoil proton in plas. q0=0-En； gamma：q0=0-Egamma，compton plateau
    Double_t Q0 = e * gr->Uniform(); //light response of Qee = f(Ep) is not considered
    qu = Q0 *TMath::Exp(-(L - x)/Lambda);
    qu = gr->Gaus(qu, qu * qRes/2.35);//QRes, relative energy resolution
    qd = Q0 * TMath::Exp(-(L + x)/Lambda);
    qd = gr->Gaus(qd, qd * qRes/2.35);

    //5.将计算好的变量值填到Tree中
    opt->Fill();

    if(i%1000 == 0) cout << ".";
  }
  cout << endl;

  // 6.将数据写入root文件中
  hctof->Write(); // 将预定义的histogram写到文件
  opt->Write();   // 将TTree到文件
  opf->Close();   // 关闭文件
}
```
- 注意：主函数名字不要跟tree的内部名字一样，这个代码运行完前边那个临时的名字opt就没有了，必须得用这个内部名在系统里调用
- `Gaus(En0, EnFWHM/2.35)` 
- `TFile *_file0 = TFile::Open("tree.root")` 打开tree.root文件
- `tree0->Print()` 看一下tree长啥样![[Pasted image 20251230194254.png]]
- `tree0->Show(0)`![[Pasted image 20251230194405.png]]
- `tree0->Draw("td-tu>>htx(500,-20,50)")`![[Pasted image 20251230194443.png]]
	- `>>` 的意思是 **“输出并存储到...”**

| **一个变量** + **(N, min, max)**                   | **1D 直方图**       | `"x>>h(100,0,10)"`           |
| ---------------------------------------------- | ---------------- | ---------------------------- |
| **两个变量 (y:x)**                                 | **2D 散点图** (默认)  | `"y:x"`                      |
| **两个变量 (y:x)** + **>>h(N,min,max, N,min,max)** | **2D 直方图** (色块图) | `"y:x>>h(50,0,10, 50,0,10)"` |
- `tree0->Draw("tof>>hh(1000,0,100)");
- `gPad->SetLogy()` 纵坐标改为对数坐标（如果是颜色映射图还可以`gPad->SetLgogz()`）![[Pasted image 20251230195859.png]]
	- 可以写`gPad`也可以写`c1`，就是画板上边那个名字
	- `SetLogy()`或`(1)`都是打开log，`(0)`是关闭log

### 打开ROOT文件并进行简单数据计算
#### 方法一（ROOT快捷写法）
```
TFile *_file0 = TFile::Open("tree.root");
tree0->Draw("td-tu>>htx(500,-20,50)");
```
#### 方法二（C++标准写法）
```
TFile *f = new TFile("tree.root");
TTree *t = (TTree*)f->Get("tree0"); 
t->Draw("td - tu >> h_diff(500, -20, 50)");
```

### 读取ROOT的tree数据，进行逐事件分析
```
TH1D *hTOF = new TH1D("hTOF", "Time of flight", 1000, 0, 100);
void readTree()
{
// 1.打开文件，得到TTree指针
  TFile *ipf = new TFile("tree.root");//打开ROOT文件
  if (ipf->IsZombie()) {//是僵尸吗（文件打开失败），是就返回true，然后报错
   cout << "Error opening file!" << endl;
   exit(-1);
  }
  ipf->cd();//将当前的内存工作目录切换到ipf这个文件里
  TTree *tree1=(TTree*)ipf->Get("tree0");

//2. 声明tree1的Branch变量

  Double_t x;
  Double_t e;
  int pid;
  Double_t tof, ctof;
  Double_t tu, td;
  Double_t qu, qd;

//3. 将变量指向对应Branch的地址
  tree1->SetBranchAddress("ctof", &ctof);//将ROOT文件内tree内名为"ctof"的branch的数据的指针指向ctof的变量。
  tree1->SetBranchAddress("tof", &tof);  
  tree1->SetBranchAddress("pid", &pid);
  tree1->SetBranchAddress("tu", &tu);   
  tree1->SetBranchAddress("td", &td);
  tree1->SetBranchAddress("qu", &qu);   
  tree1->SetBranchAddress("qd", &qd);

//4. 逐事件读取tree的branch数据
//以下三行为固定格式
  Long64_t nentries = tree->GetEntries();  //得到tree的事件总数
  for(Long64_t jentry = 0; jentry < nentries; jentry++) 
  {//对事件进行遍历
    tree->GetEntry(jentry);//将第jentry个事件数据填入步骤3.中指向的变量。
    cout <<"jentry: " << jentry << " tof: " << tof <<" pid: " << pid << endl;
  }   
   ipf->Close();  
}

```
- 为什么`TTree *tree1=(TTree*)ipf->Get("tree0")` 这里必须`Get("tree0")` 不能直接赋值给tree1呢？
	- `TTree* opt = new TTree("tree0","tree structure")` 这里不管是opt还是tree0都只是一个地址而已，就是一个存东西的地方，可以看成一个播放器里边装了很多东西，`Print()` 就是上边的按钮，摁一下就可以播放里边的内容
	- 如果要把里边的东西给另一个播放器，必须一个一个复制过去，不能只是复制一个地址，不然两个指针只会指向同一个地方，如果这个地方关掉了（旧内存被清空），数据就没有了
	- 其实只是为了保险一点
	- 这个新的tree1相当于之前的opt
- **`Branch`和`SetBranchAddress`的区别**
	- **`Branch`** 是 **写数据**（发货），**`SetBranchAddress`** 是 **读数据**（收货）
	- `opt->Branch("x", &x, "x/D");`  创建一个新文件时往里面存数据
		- 我要在 Tree 里新建一列（Branch），名字叫 `"x"`。以后每次我喊 `Fill()` 的时候，请你从内存地址 `&x` 那里拿数据，存到x列中，这个数据的类型是 `Double` (`/D`)。
		- **三个参数的作用**：
		    1. `"x"`：**起名字**。这是将来存到文件里的列名（Excel 的表头）。
		    2. `&x`：**数据源**。告诉 ROOT 去哪里拿数据存进去。
		    3. `"x/D"`：**定类型**。告诉 ROOT 这列数据是小数（Double）、整数（Int）还是别的。因为是新建的，ROOT 不知道类型，必须告诉它。  
		- 从&x拿到x列中去
	- `tree->SetBranchAddress("ctof", &ctof);` 读取已经存在的文件，把数据拿出来分析
		- 我知道文件里有一列叫 `"ctof"`。以后每次我喊 `GetEntry()` 的时候，请你把那一列的数据取出来，抄写到内存地址 `&ctof` 那个变量里去。
		- **两个参数的作用**：
		    1. `"ctof"`：**找名字**。去文件里找哪一列。
		    2. `&ctof`：**目的地**。数据取出来后放哪。
		- **为什么少了一个参数？**
		    - 因为文件已经存在了！ROOT 打开文件时一看就知道 `"ctof"` 是 Double 还是 Int，写 `"ctof/D"` 反而会报错。
		- 从cof列拿到&cof内存中去

| **特性**   | **Branch(...)**      | **SetBranchAddress(...)** |
| -------- | -------------------- | ------------------------- |
| **方向**   | **内存 $\to$ 文件** (存)  | **文件 $\to$ 内存** (读)       |
| **使用时机** | 写代码（模拟/生成数据）时        | 读代码（分析数据）时                |
| **类型定义** | **必须写** (如 `"x/D"`)  | **不需要写** (ROOT自己知道)       |
| **动作关联** | 配合 `tree->Fill()` 使用 | 配合 `tree->GetEntry()` 使用  |
- `GetEntries()`：返回总行数
- `Long64_t` ：防止数据量太大超过普通整数范围
- for循环部分还可以写
	- ```
	  //在循环外面，定义一个直方图 
	  TH1D *hDiff = new TH1D("hDiff", "td - tu distribution", 100, -20, 50);
	  Long64_t nentries = tree1->GetEntries(); 
	  for(Long64_t jentry = 0; jentry < nentries; jentry++) 
	  {
		  tree1->GetEntry(jentry); 
		  //在循环里面，计算并填充
		  hDiff->Fill(td - tu); 
		  //cout ... (原来的打印可以保留也可以注释掉，打印太多会慢) 
	   } 
	   //循环结束后，画图 
	   TCanvas *c1 = new TCanvas("c1", "Time Difference", 800, 600); 
	   hDiff->Draw(); 
	   // 注意：不要在这里马上 ipf->Close()，否则图可能会消失或者报错 
	   // 如果必须关闭，请先保存图片 c1->SaveAs("diff.png"); 
	  ```

### 逐事件读取已有文件的TTree变量，计算生成新变量，并将其存入新的root文件
```
void calibration() {
    // ==========================================
    // 1. 打开旧文件，获取原 Tree
    // ==========================================
    TFile *ipf = new TFile("tree.root"); // 打开包含原始数据的ROOT文件
    if (ipf->IsZombie()) {
        cout << "Error opening file!" << endl;
        exit(-1);
    }
    TTree *tree = (TTree*)ipf->Get("tree"); // 获取原Tree指针

    // ==========================================
    // 2. 声明原 Tree 的变量并挂载地址 (读取所需)
    // ==========================================
    Double_t tu, td; // 我们至少需要 tu 和 td 来计算位置和时间
    Double_t qu, qd; // 可能需要 qu 和 qd 来计算能量
    Double_t ctof;   // 原始的 ctof 用于对比

    tree->SetBranchAddress("tu", &tu);
    tree->SetBranchAddress("td", &td);
    tree->SetBranchAddress("qu", &qu);
    tree->SetBranchAddress("qd", &qd);
    tree->SetBranchAddress("ctof", &ctof);

    // ==========================================
    // 3. 定义刻度参数 (Calibration Parameters)
    // ==========================================
    // 这些参数通常来自实验刻度分析
    
    // 时间刻度参数 (用于计算 TOF)
    Double_t kt = 1.0;  // 斜率，通常接近 1.0 或 0.5 (取决于公式定义)
    Double_t bt = -5.0; // 截距，用于扣除线缆延迟等 offset

    // 位置刻度参数 (用于计算 x)
    // 之前的代码中 vsc = 7.5, x = (td-tu)*vsc/2. 所以 kx 大约是 3.75
    Double_t kx = 3.75; 
    Double_t bx = 0.0;  // 位置偏移量 correction

    // 能量参数 (几何平均)
    Double_t ke = 1.0; 

    // ==========================================
    // 4. 声明新 Tree 的变量 (写入所需)
    // ==========================================
    Double_t ttof; // Calibrated TOF
    Double_t tx;   // Calibrated Position
    Double_t qe;   // Calibrated Energy (Q)

    // ==========================================
    // 5. 声明新文件和新 Tree
    // ==========================================
    TFile *opf = new TFile("tree2.root", "recreate"); // 新文件
    TTree *opt = new TTree("tree", "Calibrated Tree"); // 新Tree

    // 为新 Tree 创建分支 (Branch)
    // 注意：这里是 "Branch" 不是 "SetBranchAddress" (那是读的时候用的)
    // 也不是 "SetBrach" (这是拼写错误)
    opt->Branch("ttof", &ttof, "ttof/D");
    opt->Branch("tx",   &tx,   "tx/D");
    opt->Branch("qe",   &qe,   "qe/D");
    
    // 如果你想把旧变量也存进去方便对比，可以加上：
    // opt->Branch("ctof_old", &ctof, "ctof_old/D");

    // ==========================================
    // 6. 逐事件循环：读取 -> 计算 -> 填充
    // ==========================================
    Long64_t nentries = tree->GetEntries(); 
    cout << "Total entries: " << nentries << endl;

    for (Long64_t jentry = 0; jentry < nentries; jentry++) {
        // 6.1 获取当前事件的原始数据 (tu, td, qu, qd...)
        tree->GetEntry(jentry);

        // 6.2 进行物理计算 (Calibration)
        
        // 计算位置 tx: 基于时间差
        // 公式原型: x = v * (td - tu) / 2 + offset
        tx = kx * (td - tu) + bx; 

        // 计算飞行时间 ttof: 基于时间均值
        // 公式原型: TOF = (tu + td) / 2 - L/v + offset
        // 这里简化为线性修正:
        ttof = kt * (tu + td) / 2.0 + bt;

        // 计算能量 qe: 基于几何平均 (去除位置依赖)
        // 公式原型: Q = sqrt(qu * qd)
        qe = ke * TMath::Sqrt(qu * qd);

        // 6.3 将计算结果填入新 Tree
        opt->Fill();

        if (jentry % 10000 == 0) 
            cout << "Processed " << jentry << " events..." << endl;
    }

    // ==========================================
    // 7. 保存并关闭
    // ==========================================
    opt->Write(); // 将新 Tree 写入文件
    opf->Close(); // 关闭新文件
    ipf->Close(); // 关闭旧文件
    
    cout << "Done! Calibrated data saved to tree2.root" << endl;
}
```

### 想在ROOT中调用函数中的变量
```
//定义全局变量
TH1D *tdiff = nullptr;
TH1D *dtd   = nullptr;
TF1  *f1    = nullptr;

void readtreeTH1D()
{
// 1.打开文件，得到TTree指针
  TFile *ipf = new TFile("tree.root");//打开ROOT文件
  if (ipf->IsZombie()) {//是僵尸吗（文件打开失败），是就返回true，然后报错
   cout << "Error opening file!" << endl;
   exit(-1);
  }
  ipf->cd();//将当前的内存工作目录切换到ipf这个文件里
  TTree *tree1=(TTree*)ipf->Get("tree0");
  if (!tree1) { cout << "No tree!" << endl; return; }

//2. 声明tree1的Branch变量
  Double_t x;
  Double_t e;
  int pid;
  Double_t tof, ctof;
  Double_t tu, td;
  Double_t qu, qd;

//3. 将变量指向对应Branch的地址
  tree1->SetBranchAddress("ctof", &ctof);//将ROOT文件内tree内名为"ctof"的branch的数据的指针指向ctof的变量。
  tree1->SetBranchAddress("tof", &tof);  
  tree1->SetBranchAddress("pid", &pid);
  tree1->SetBranchAddress("tu", &tu);   
  tree1->SetBranchAddress("td", &td);
  tree1->SetBranchAddress("qu", &qu);   
  tree1->SetBranchAddress("qd", &qd);

//4. 逐事件读取tree的branch数据
  // 错误写法: TH1D *tdiff = new ... (这会创建一个新的局部变量，覆盖全局变量)
  // 正确写法: 
  if(tdiff) delete tdiff; // 防止重复运行报错，先删掉旧的
  tdiff = new TH1D("tdiff", "td - tu", 200, -20, 80);
  
  // 【关键】必须加上这句，否则文件关闭时 tdiff 会被删掉，全局指针变成野指针
  tdiff->SetDirectory(0);
  
  Long64_t nentries = tree1->GetEntries(); 
  for(Long64_t jentry = 0; jentry < nentries; jentry++) 
  {
    tree1->GetEntry(jentry); 
    //在循环里面，计算并填充
    hDiff->Fill(td - tu); 
    //cout ... (原来的打印可以保留也可以注释掉，打印太多会慢) 
  } 
  ipf->Close(); // 关闭文件
  
  // 计算微分
  if(dtd) delete dtd; // 防止重复运行
  dtd = (TH1D*)tdiff->Clone("dtd");
  dtd->SetDirectory(0); // 同样要保命
  dtd->Reset();
  
  for(int i = 1; i < tdiff->GetNbinsX(); i++) 
  {
    Double_t y_diff = tdiff->GetBinContent(i+1) - tdiff->GetBinContent(i);
    dtd->SetBinContent(i, y_diff);
  }
  
  // 拟合
  if(f1) delete f1;
  f1 = new TF1("f1", "[0]*TMath::Exp(-0.5*((x-[1])/[2])^2)", 39.5, 45);
  f1->SetParameters(-350, 41.5, 0.5);
  
  dtd->Fit("f1", "R"); // 拟合结果会自动保存在 dtd 的 list 里
  
  // 画图
  TCanvas *c1 = new TCanvas("c1", "Result", 800, 600);
  dtd->Draw("HIST");
  f1->Draw("SAME");
}
```

## TSpectrum
/home/vip/Desktop/tutorials/legacy/spectrum里边有很多样本代码
### 生成本底谱
```
TSpectrum *s = new TSpectrum(500);//maximum number of peaks
TH1F *hb0 = (TH1F*)s->Background(h0, 15, "same");
h0->Draw();
hb0->SetLineColor(kRed);
hb0->Draw("same");
c1->Draw();
```
- `TH1 * TSpectrum::Background( const TH1 *h, Int_t niter = 20, Option_t  *option = "" )`
	- Parameters:
	    - h: input 1-d histogram
	    - niter: numberIterations, (default value = 20) Increasing numberIterations make the result smoother and lower.
	        - 通过观察本底谱的形状，选择合适的niter参数。减本底的重要原则是宁少勿多，避免减出负值！
### 减去本底
```
h0->Clone("hh");
hh->Add(hh, hb1, 1, -1);//hh = 1 * hh-1 * hb
hh->Draw();
c1->SetLogy(0);
c1->Draw();
```
### 寻峰
```
Double_t *xpeaks, *ypeaks;
Int_t nfound = s->Search(hh,2,"",0.005); //根据实际峰宽选择合适的sigma值！
cout<<nfound<<endl;
xpeaks = s->GetPositionX();
ypeaks = s->GetPositionY();
for(int i=0; i<nfound; i++)
    cout << i << ": " << int(xpeaks[i]) 
         << ", " << ypeaks[i] << endl;
hh->Draw();
c1->Draw();
```
- `Int_t TSpectrum::Search(const TH1  *hin, Double_t sigma = 2, Option_t *option = "", Double_t threshold = 0.05 )`
	- Parameters:
	    - hin: pointer to the histogram of source spectrum
	    - sigma: sigma of searched peaks
	    - threshold: (default=0.05) peaks with amplitude less than threshold * highest_peak are discarded. 0<threshold<1
    - 建议通过Background功能选择合适的本底形状，减完本底后再寻峰！
### 给寻出来的峰排序
这个 `map` 的作用就是**把找到的乱序的峰，整理成有序的数据表**。map提供一对一的数据处理，key-value键值对，其类型可以自己定义，第一个称为关键字，第二个为关键字的值
- 什么是 Key 和 Value？
	- **Key (键)**：
	    - 唯一的索引、编号
	    - 在谱图里对应的是 `xpeaks`（峰的位置/道址）。
	    - 不能重复！！！一个道址上只能有一个主峰数据（其实就是x只能对应一个y，但y可以对应好几个x），且 `map` 会自动根据 Key 的大小进行**从小到大排序**。
	- **Value (值)**：
	    - 该索引下存放的具体内容。
	    - 在谱图里对应的是 `ypeaks`（峰的高度/计数）。
	    - 可以重复，不同的位置（Key）可能有相同的高度（Value）。
	- **Pair (键值对)**：
	    - `map` 里的每一个元素都是一个“一对儿”数据，写作 `<Key, Value>`。
- 将 `TSpectrum` 找到的 `xpeaks` 和 `ypeaks` 存入 map 中。
#### 定义map
```
map<Int_t, Double_t> m1;//第一个是键的类型，第二个是值的类型
multimap<Int_t, Double_t> mul1;
```
- 前边的`map<Int_t, Double_t>`就相当于是int、double了，后边m1才是变量名
- `map`严格去重，有重复的key就不存了，但`multimap`是记流水账，管它重不重复全都存
#### 赋值
```
for(int i=0; i<nfound; i++){
    int key = xpeaks[i];
    double value = ypeaks[i];
    m1.insert(make_pair(key, value));//插入key，value值
    m1.insert(make_pair(key, value));//插入key，value值
    mul1.insert(make_pair(key, value));
    mul1.insert(make_pair(key, value));//重复插入key，value值    
}
```
- `m1.insert(make_pair(key, value))`
	- `make_pair(key, value)` `map` 不收散装的东西，它不接受左手拿一个 `key`，右手拿一个 `value` 直接扔进去。要求必须把这两个东西捆绑成一个整体，这个整体叫 **`pair`（一对儿）**。
	    - `make_pair` 把 `key`（道址）和 `value`（计数）变成一个 `pair` 对象
	- `m1.insert(...)`
	    - 它的参数是刚才 `make_pair` 打包好的包裹
	    - 接过这个 `pair` 包裹，打开 `m1` 的大门，把它放进去，并且根据 `key` 的大小，自动把它摆到正确的位置上（排序）。
#### 查看大小
`cout<<m1.size()<<","<<mul1.size
看看存了多少组数据
#### 查找元素
```
int i = 762;
auto im1 = m1.find(i);//完整的写应该是map<Int_t,Double_t>::iterator im1=m1.find()
if(im1 != m1.end()) 
    cout << im1->first << ", " << im1->second << endl;
else 
    cout << "key=" << i << " is not found." << endl;
i = 100;
im1 = m1.find(i);
if(im1 != m1.end()) 
    cout << im1->first << ", " << im1->second << endl;
else 
    cout << "key=" << i << " is not found." << endl;
```
- iterator其实是迭代器的类型 命名了一个im1变量
- `im1->first`是m1中的key数据，`im1->second`是m1中的value数据
- `!=`是不等于，意思是im1不等于m1尽头（空）的话，就是没结束，可以输出数据，反之报错
#### 正向遍历
```
int num = 0;
cout<<"map elements:"<<endl;

for(auto im=m1.begin(); im!= m1.end(); im++) {
    if(num > 10) break;
    cout << "  " << im->first << ", " << im->second << endl;
    num++;
}
num = 0;
cout << "multimap elements:" << endl;
for(auto imul=mul1.begin(); imul!=mul1.end(); imul++){
    if(num > 10) break;
    cout << "  " << imul->first << ", " << imul->second << endl;
    num++;
}
```
#### 反向遍历
```
num = 0;
cout << "map elements:" << endl;
for(auto im=m1.rbegin(); im!=m1.rend(); im++) {
    if(num > 10) break;
    cout << "  " << im->first << ", " << im->second << endl;
    num++;
}
num = 0;
cout << "multimap elements:" <<endl;
for(auto imul=mul1.rbegin(); imul!=mul1.rend(); imul++){
    if(num > 10) break;
    cout << "  " << imul->first << ", " << imul->second << endl;
    num++;
}
```
### 实操
draw_with_range.C文件
```
void draw_with_range() {
    
    TFile *spectrum = new TFile("TSpectrum.root");
    if (spectrum->IsZombie()) { cout << "Error opening file" << endl; return; }

    // 获取直方图
    TH1F *histo = (TH1F*)spectrum->Get("back1");
    if (!histo) { cout << "Histogram not found" << endl; return; }

    // 创建画板 (可选，但推荐)
    TCanvas *c1 = new TCanvas("c1", "c1", 800, 600);

    // ==============================================
    // 关键步骤：设置 X 轴范围
    // ==============================================
    // 这告诉 ROOT："我只想看横坐标在 0 到 3000 之间的数据"
    histo->GetXaxis()->SetRangeUser(0, 3000);

    // 也可以顺便设置一下 Y 轴的范围，例如 0 到 1000 (如果需要的话)
    // histo->GetYaxis()->SetRangeUser(0, 1000);

    // 画图
    // 推荐加上 "HIST" 选项，这样画出来的是标准的直方图台阶线，而不是带误差棒的点
    histo->Draw("HIST");
}
```
运行时在终端输入`root draw_with_range.C`
```
root [1] c1->SetLogy()//y轴变为log坐标
root [2] TSpectrum* s = new TSpectrum(500);//最大寻峰数500
root [3] TH1F* hback1 = (TH1F*)s->Background(back1,15,"same");//找本底
root [4] TH1F* hback1 = (TH1F*)s->Background(back1,30,"same");//改个参数找本底
root [5] back1->Clone("bback1");//删本底第一步先克隆一份
root [6] bback1->Add(bback1,hback1,1,-1)//删本底
(bool) true
root [7] bback1->Draw();//画出来
root [8] c1->SetLogy(0)//y轴改为正常坐标
```
FindPeaks.C文件寻峰
```
void FindPeaks() {
    // =========================================================
    // 1. 去内存里“捞”你刚才做好的 bback1
    // =========================================================
    // gDirectory->Get() 可以在当前内存目录中查找对象
    TH1F *bback1 = (TH1F*)gDirectory->Get("bback1");

    if (bback1 == nullptr) {
        cout << "错误：在内存里没找到 bback1！" << endl;
        cout << "请确认你刚才在命令行里运行了：back1->Clone(\"bback1\")" << endl;
        return;
    }

    // =========================================================
    // 2. 重新创建一个寻峰工具 (这个不能省，但开销很小)
    // =========================================================
    TSpectrum *s = new TSpectrum(500); 

    // =========================================================
    // 3. 直接开始寻峰
    // =========================================================
    // 注意：这里是对 bback1 寻峰
    Int_t nfound = s->Search(bback1, 2, "", 0.05);

    cout << "找到了 " << nfound << " 个峰。" << endl;

    // =========================================================
    // 4. 打印和保存结果 (和之前一样)
    // =========================================================
    Double_t *xpeaks = s->GetPositionX();
    Double_t *ypeaks = s->GetPositionY();

    // 简单打印一下前 10 个看看
    for(int i=0; i<nfound && i<10; i++) {
        cout << "Peak " << i << ": x=" << xpeaks[i] << ", y=" << ypeaks[i] << endl;
    }
    
    // 画图确认
    // bback1->Draw("HIST"); // 如果你屏幕上已经画着了，这句也可以不写
}
```
运行时必须按这个在root里输入，.x会报错
```
root [10] .L FindPeaks.C
root [11] FindPeaks()
```
AnalyzePeaks.C文件存map并读取
```
#include <iostream>
#include <map>
#include <utility> 

// 引入 ROOT 头文件
#include "TH1F.h"
#include "TSpectrum.h"
#include "TROOT.h"
#include "TFile.h" 

using namespace std;

void AnalyzePeaks() {
    // 1. 捞回直方图
    TH1F *bback1 = (TH1F*)gDirectory->Get("bback1");
    if (!bback1) {
        cout << "错误：没有在内存中找到 bback1" << endl;
        return;
    }

    // 2. 重新寻峰
    TSpectrum *s = new TSpectrum(500);
    // 使用 "goff" 选项关闭画图，只计算
    Int_t nfound = s->Search(bback1, 2, "goff", 0.05); 
    
    Double_t *xpeaks = s->GetPositionX();
    Double_t *ypeaks = s->GetPositionY();

    // =========================================================
    // 核心修改部分：避开 make_pair 的兼容性问题
    // =========================================================
    
    // 显式定义 map，使用 Int_t 和 Double_t
    map<Int_t, Double_t> m1;
    multimap<Int_t, Double_t> mul1;

    cout << "Found " << nfound << " peaks. Filling maps..." << endl;

    for(int i=0; i<nfound; i++){
        Int_t key = (Int_t)(xpeaks[i] + 0.5); 
        Double_t value = ypeaks[i];

        // 【安全写法 1】对于 map，直接用下标赋值
        // 这避免了调用 insert 和 pair 的构造函数，最不容易报错
        m1[key] = value; 

        // 【安全写法 2】对于 multimap，显式构造完全匹配的 pair
        // map 内部存储的其实是 <const Key, Value>
        // 我们这里手动构造这个类型，不让编译器瞎猜
        typedef std::pair<const Int_t, Double_t> MyPair;
        
        mul1.insert(MyPair(key, value));
        mul1.insert(MyPair(key, value)); // 重复插入
    }

    // 3. 打印大小
    cout << "--------------------------------" << endl;
    cout << "Map Size:      " << m1.size() << endl;
    cout << "Multimap Size: " << mul1.size() << endl;

    // 4. 遍历输出 Map (使用迭代器最原始的写法)
    cout << "--------------------------------" << endl;
    cout << "--- Map Elements (Top 10) ---" << endl;
    
    int num = 0;
    // 这里的迭代器类型写全，防止 auto 推导错误
    map<Int_t, Double_t>::iterator im;
    for(im = m1.begin(); im != m1.end(); im++) {
        if(num >= 10) break;
        cout << " Key: " << im->first << ", Value: " << im->second << endl;
        num++;
    }

    // 5. 遍历输出 Multimap
    cout << "--------------------------------" << endl;
    cout << "--- Multimap Elements (Top 10) ---" << endl;
    
    num = 0;
    multimap<Int_t, Double_t>::iterator imul;
    for(imul = mul1.begin(); imul != mul1.end(); imul++){
        if(num >= 10) break;
        cout << " Key: " << imul->first << ", Value: " << imul->second << endl;
        num++;
    }
}
```
运行时输入
```
root [12] .L AnalyzePeaks.C
root [13] AnalyzePeaks()
```
## 能量刻度
1. 画图，并将ADC的谱进行连续化。
2. 寻峰得到的 α-峰位与已知 α 能量之间进行匹配，利用峰位估计值进行粗刻度。
3. 选取合理的拟合区间，对每个峰进行高斯拟合，得到最终刻度系数。
### 输出双面硅条和环上的数据作图
```
Int_t pe[48];//Pie 0-48
Int_t re[48];//Ring 0-48
```

```
TFile *f = new TFile("s4.root");
TTree *tree = (TTree*)f->Get("tree");
TCanvas *c1 = new TCanvas;
```

```
tree->Draw("pe[0]>>hx0(1000, 0, 2000)");//单个条
tree->Draw("pe[1]>>hx1(1000, 0, 2000)");//单个条
TH1I *hx0 = (TH1I*)gROOT->FindObject("hx0");
TH1I *hx1 = (TH1I*)gROOT->FindObject("hx1");//这一步是为了后边方便单独调整图像
hx0->SetLineColor(kGreen);
hx1->Draw();
hx0->Draw("same");
c1->Draw();
```
- 硅探测器有128个条（Pie）和128个环（Ring）。每个条接的电子学放大器增益（Gain）都不可能完全一样。
- 绿线 (`pe[0]`) 和蓝线 (`pe[1]`) 的峰位虽然很接近，但并没有完全重合
- 不能简单地把所有数据加在一起用，必须对每一条（pe[0], pe[1], ... pe[47]）单独进行刻度，求出它们各自的参数，把它们“拉齐”了才能合起来用。
```
tree->Draw("pe>>(1000,0,2000)");//所有条pe[0-48]的总和图
c1->Draw();
```
- 确认总计数（Entries）是否足够多。
- 确认主要的峰（$\alpha$ 峰）是否清晰可见。
```
tree->Draw("re[0]>>hy0(1000, 0, 2000)");//单个环
tree->Draw("re[1]>>hy1(1000, 0, 2000)");//单个环
TH1I *hy0 = (TH1I*)gROOT->FindObject("hy0");
TH1I *hy1 = (TH1I*)gROOT->FindObject("hy1");
gPad->SetLogy();
hy0->SetLineColor(kGreen);
hy1->Draw();
hy0->Draw("same");
c1->Draw();
```
### 平滑
从ROOT文件里取数据 $\to$ 转换格式 $\to$ 算法处理 $\to$ 放回数据 $\to$ 画图。
#### 读取文件与对象
```
TFile *f = new TFile(file.Data());   // 1. 打开存有数据的 .root 文件
auto smooth = (TH2F *)f->Get("smooth1"); // 2. 取出那个噪点很多的二维直方图，叫 "smooth1"
gStyle->SetOptStat(0);   // 3. 关掉右上角的统计框（为了画图好看，不遮挡视线）
auto *s = new TSpectrum2();   // 4. 创建一个“平滑专家”工具箱 (TSpectrum2 对象)
```
#### 数据搬运（从直方图到数组）
```
for (i = 0; i < nbinsx; i++) {
   for (j = 0; j < nbinsy; j++) {
      // 把 smooth 直方图里的数，一个一个复制到 C++ 原生数组 source 里
      source[i][j] = smooth->GetBinContent(i + 1, j + 1);
   }
}
```
- 为什么要搬运？
    - ROOT 的直方图对象 `TH2F` 是一个复杂的类，但是 `TSpectrum2::SmoothMarkov` 不接受 `TH2F` 对象，它只接受纯粹的数学数组 (`double`)。
    - 所以要把数据从对象里拆出来放到数组里。      
#### 核心算法（马尔可夫平滑）
```
s->SmoothMarkov(source, nbinsx, nbinsy, 3); // 5,7
```
- 真正干活的一行，直接修改了 `source` 数组里的数值。
- `SmoothMarkov(Double_t **source, Int_t ssx, Int_t ssy, Int_t aver_window)`这四个参数分别对应**数据源**、**数据尺寸**和**平滑强度**
	- `source` (第一个参数)
		- 原始数据源（二维数组指针）。
		- 类型：`Double_t **` (指向指针的指针)。
	    - 这是你要“磨皮”的那张图的数据，不是 `TH2` 直方图对象，而是一个纯二维数组。
	    - 在这个数组里，`source[i][j]`存储的是第`(i, j)`个坐标点上的计数。
	- `nbinsx` (第二个参数)
		- X 轴的长度（Bin 的数量）。
		- 类型：`Int_t` (整数)。
	    - 告诉算法二维数组在 X 方向上有多少列
	- `nbinsy` (第三个参数)
		- Y 轴的长度（Bin 的数量）。
		- 类型：`Int_t` (整数)。
	    - 告诉算法你的二维数组在 Y 方向上有多少行。
	- `3` (第四个参数) —— **最关键的参数**
		- **含义**：**平均窗口大小 (Averaging Window)**。
		- **类型**：`Int_t` (整数)。
		- 如果填 `3`：对周围 $3\times3$ 的区域算权重，平滑效果细腻，保留细节。
	    - 如果填 `7`：对周围 $7\times7$ 的区域算，平滑效果强，图片会变得很糊，像磨砂玻璃一样。
#### 搬运回填与绘图
数据处理完了，现在它们还在 `source` 数组里，无法直接画图（ROOT 画图需要 `TH1/TH2` 对象）。所以需要再搬回去。
```
// 1. 把处理干净的数据，填回 smooth 直方图对象里
for (i = 0; i < nbinsx; i++) {
   for (j = 0; j < nbinsy; j++)
      smooth->SetBinContent(i + 1, j + 1, source[i][j]);
}
// 2. 画出来
smooth->Draw("SURF2");
```
- **`SetBinContent`**：把原来的噪点数据覆盖掉，换成平滑后的数据。
- **`"SURF2"`**：这是一个画图选项。
    - 普通 `Draw()` 画出来是散点图或色块图。
    - `SURF2` 画出来的是**彩色的 3D 曲面图**（从鸟瞰角度），颜色代表高度（计数多少）。这种画法最适合展示平滑后的地形效果。
### 寻峰并标注(存在peakstr.C里了，有改动自己找)
```
TH1F *h=NULL,*hb=NULL;
Int_t nfound;
Double_t *xpeaks=NULL, *ypeaks=NULL;
TSpectrum *s=NULL;
```
- **定义全局指针**：
    - `h`: 指向要处理的那个信号直方图
    - `hb`: 用来指向计算出的本底（Background）直方图
    - `xpeaks`, `ypeaks`: 用来存储找到的峰的横坐标（位置）和纵坐标（高度）
    - `s`: 指向 `TSpectrum` 对象（寻峰工具箱）。
- 初始化为 `NULL` 是为了防止野指针
	- `TH1F *h;` 这种是野指针，指针可能会随机指向什么地方
```
void peakstr(TString hname, vector<Double_t> &pe, Double_t thres=0.05, int backsub=0)
```
- 参数：
    - `hname`: 字符串，你要找峰的直方图的名字（例如 "hx0"）。
    - `&pe`: 一个 `vector` 的引用。用来把找到的峰的位置存进去，带回给主程序。
    - `thres`: 阈值，默认 0.05。只有高度超过最高峰 5% 的峰才会被认出来。
    - `backsub`: 开关，0 表示不扣本底，1 表示扣除本底。
```
{
  pe.clear();                                //清空pe
  multimap<Int_t,Double_t> me;               //用于排序, key: ypeaks, value: xpeaks
  h = (TH1F*)gROOT->FindObject(hname);
  if(!s) s = new TSpectrum(500);
  if(backsub) {
    hb = (TH1F*)s->Background(h, 80, "same"); // 80:本底光滑程度
    h->Add(h, hb, 1, -1);   
  }
  nfound = s->Search(h, 2, "", thres);  //2:gaus峰的sigma，需要根据峰的实际情况调整 
  TPolyMarker *pm = (TPolyMarker *)
     h->GetListOfFunctions()->FindObject("TPolyMarker");//是一行，写不开分开了
  pm->SetMarkerStyle(32);
  pm->SetMarkerColor(kGreen);
  pm->SetMarkerSize(0.4);
```
- **美化标记**：`Search` 函数会在图上画出三角形标记（TPolyMarker）。这里把它们找出来，改成样式 32（倒三角），绿色，大小 0.4。
```
  xpeaks = s->GetPositionX();
  ypeaks = s->GetPositionY();
  for(int j=0; j<nfound; j++) {
    stringstream ss;
    ss << xpeaks[j];
    TString s1 = ss.str();
    TLatex *tex = new TLatex(xpeaks[j],ypeaks[j],s1);//准备标签：把峰的位置数值（Double）转成字符串，创建一个TLatex文本对象，准备画在(x, y)坐标上。
    me.insert(make_pair(int(ypeaks[j]),xpeaks[j]));
```
- 存入 Map 排序，把（高度, 位置）作为一对插入到 map 中，但make_pair可能用不了，要`typedef std::pair<const Int_t, Double_t> MyPair;me.insert(MyPair(key, value));`这样写
```
    tex->SetTextFont(13);
    tex->SetTextSize(14);
    tex->SetTextAlign(12);
    tex->SetTextAngle(90);
    tex->SetTextColor(kRed);
    tex->Draw();
  }
```
- **画字**：设置字体、大小、垂直旋转90度、红色，然后画在图上。这样图上每个峰顶上都会标出它的位置数值。
```
  for(auto ie = me.rbegin(); ie != me.rend(); ie++) {//反向遍历，先看高的峰
    cout << ie->second << " " << ie->first <<endl;//peak,count;
    pe.push_back(ie->second);//按照计数由大到小填入
  }
  me.clear();
}
```
- `push_back()`的意思是把括号里的数据塞在pe的最后一个数据的后边
#### 调用
```
gROOT->ProcessLine(".L peakstr.C");//可以写root[0] .L peakstr.C
vector<Double_t> pe;
Double_t e[4] = {5.5658, 6.1748, 6.6708, 8.6931};//alpha energy
peakstr("hx1",pe);//只有这两个参数未知，写这俩就够了
gPad->SetLogy();
c1-Draw();
```
### 将候选峰与已知alpha峰位进行匹配
#### 峰位匹配 (Peak Matching)
`TSpectrum` 可能会找到 6 个甚至更多的峰，但标准源（$^{232}U$）主要只有 4 个标准能量值。我们必须搞清楚哪一个“道址（Channel）”对应哪一个“能量（Energy）”。
- 策略说明（文字部分）
	- 通过观察能谱发现：虽然有多个峰，但我们需要匹配的是那 4 个主要的 $\alpha$ 峰。
	- **最终策略**：先找到前 6 个峰，把它们按**位置（道址）从小到大排序**。然后，作者通过人眼观察数据发现，最前面的 2 个峰可能不是我们想要的（可能是低能端的杂峰或并峰），所以决定**跳过前 2 个，取第 3、4、5、6 个峰**来对应我们的 4 个标准能量。
- 代码
```
sort(pe.begin(), pe.begin()+6); // 1. 排序
```
- `pe` 里面存的是 `TSpectrum` 找到的峰的位置（Channel）。
- `sort` 函数把前 6 个峰按**从小到大**（道址从低到高）重新排列。这一步非常关键，因为物理上的能量 `e` 数组也是从小到大排的，必须让它们顺序一致才能对应。
```
Double_t mpe[4], cpe[4];//peak,count
for(int i=0; i<4; i++) {
    mpe[i] = pe[2+i];
    cpe[i] = hx1->GetBinContent(hx1->FindBin(mpe[i]));
    cout << mpe[i] << " " << cpe[i] << endl;
}
```
- **`mpe[i] = pe[2+i]`**: 这一行体现了上面说的策略。
    - `pe[0]` 和 `pe[1]` 被抛弃了。
    - 取 `pe[2]` 对应第 1 个能量，`pe[3]` 对应第 2 个能量，以此类推。
- 这意味着在实验数据中，比 5.56 MeV 更低的位置还有两个被误判的峰或者干扰峰，需要手动修正匹配逻辑，从第 3 个峰开始算起。
#### 拟合确认 (Fitting Confirmation)
一旦完成了“配对”（知道了 X 轴是 `mpe`，Y 轴是 `e`），就可以画图并计算方程了。
```
TGraph *gr = new TGraph(4, mpe, e);
```
- **创建图表**：
    - **X轴 (`mpe`)**：探测器测到的道址（比如 993, 1079...）。
    - **Y轴 (`e`)**：真实的物理能量（5.5658, 6.1748...）。
```
gr->Draw("A*");
gr->Fit("pol1");
```
- **`Fit("pol1")`**：这是核心计算步骤。
    - `pol1` 代表 First-order Polynomial（一次多项式），即直线方程 $y = p_0 + p_1 x$。
    - ROOT 会自动调整 $p_0$（截距 `offset`）和 $p_1$（斜率 `slope`），使得这条直线尽可能完美地穿过那 4 个点
```
TF1 *f1 = gr->GetFunction("pol1");
c1->Draw();
```
- **获取结果**：把拟合好的函数存到 `f1` 里。之后用 `f1->GetParameter(0)` 和 `f1->GetParameter(1)` 来获取最终的校准参数（斜率和截距），打印出来就是`E=k*Ch+b`。
![[Pasted image 20260103125607.png]]
- `chi2/ndf` 是数据分析（特别是高能物理 ROOT 拟合）中用来**判断“拟合得好不好”**的最核心指标。
	- $\chi^2$ (Chi-square / 卡方)，代表偏差的总和
	- ndf (Number of Degrees of Freedom / 自由度)，代表有多少个多余数据点来验证理论。
		- $ndf = \text{数据点的个数} - \text{拟合参数的个数}$。
		    - 有 **4** 个数据点（4个峰）。
		    - 拟合直线 $y = kx + b$，需要求 **2** 个参数（$k$ 和 $b$）。
		    - 所以，$ndf = 4 - 2 = 2$。
	- 判定标准

| **数值范围**              | **含义**   |
| --------------------- | -------- |
| **接近 1** (比如 0.5 ~ 2) | **完美拟合** |
| **非常大** (比如 > 100)    | **拟合很烂** |
| **非常小** (比如 < 0.001)  | **好得离谱** |
### 确定拟合区间，进行高斯拟合（其实到上边一步就差不多了）
```
double par[4][3];//peak,sigma,chi2/ndf
double ge[4];
TF1 *fg[4];
TFitResultPtr fr;
for(int i=0; i<4; i++) {
    fg[i] = new TF1(Form("fg%d", i), "gaus");
    fg[i]->SetParameters(cpe[i], mpe[i], 4);//constant,mean, sigma
    fr = hx1->Fit(fg[i], "SQ+", "", mpe[i]-5, mpe[i]+8);//superimpose TF1 to hx1
    par[i][0] = fg[i]->GetParameter(1);
    par[i][1] = fg[i]->GetParameter(2);
    par[i][2] = fr->Chi2()/fr->Ndf();
    ge[i] = par[i][0];
    cout << Form("peaks = %4.1f,sigma = %.2f,chi2/ndf = %.2f",par[i][0],par[i][1], par[i][2]) << endl;
}
c1->Draw();
```
拟合某个峰（上边这个拟合了四个峰）
```
TGraph *gr1 = new TGraph(4,ge,e);
gr1->Draw("A*");
gr1->Fit("pol1");
TF1 *f2 = gr->GetFunction("pol1");
c1->Draw();
```
画散点图用一阶函数再拟合
## 记录粒子能量和数量直方图
/home/vip/Desktop/tutorials/analysis/dataframe/df017_vecOpsHEP.root
```
auto filename = gROOT->GetTutorialDir() + "/analysis/dataframe/df017_vecOpsHEP.root";
auto treename = "myDataset";

using namespace ROOT;


void WithTTreeReader()
{
   TFile f(filename);
   TTreeReader tr(treename, &f);
   TTreeReaderArray<double> px(tr, "px");
   TTreeReaderArray<double> py(tr, "py");
   TTreeReaderArray<double> E(tr, "E");

   TH1F h("pt", "pt", 16, 0, 4);

   while (tr.Next()) {
      for (auto i=0U;i < px.GetSize(); ++i) {
         if (E[i] > 100) h.Fill(sqrt(px[i]*px[i] + py[i]*py[i]));
      }
   }
   h.DrawCopy();
}
```
这是比较老派的写法（虽然比更老的 `SetBranchAddress` 好一点，但依然繁琐）。
- **特点**：需要显式地写两层循环。
    - 外层 `while (tr.Next())` 循环遍历每一个**事件 (Event)**。
    - 内层 `for` 循环遍历事件里的每一个**粒子**。
- **缺点**：代码长，容易写错索引，逻辑和循环混杂在一起。
```
void WithRDataFrame()
{
  RDataFrame f(treename, filename.Data());
   auto CalcPt = [](RVecD &px, RVecD &py, RVecD &E) {
      RVecD v;
      for (auto i=0U;i < px.size(); ++i) {
         if (E[i] > 100) {
            v.emplace_back(sqrt(px[i]*px[i] + py[i]*py[i]));
         }
      }
      return v;
   };
   f.Define("pt", CalcPt, {"px", "py", "E"})
    .Histo1D<RVecD>({"pt", "pt", 16, 0, 4}, "pt")->DrawCopy();
}
```
这里引入了 **RDataFrame**（ROOT 的现代声明式分析工具）。
- **特点**：不再需要写外层的事件循环，RDataFrame 帮你自动并行处理。
- **缺点**：但是在 `CalcPt` 函数内部，作者依然手动写了一个 `for` 循环来遍历粒子。这就像是买了辆法拉利，结果还在用脚蹬着走。
```
void WithRDataFrameVecOps()
{
   RDataFrame f(treename, filename.Data());
   auto CalcPt = [](RVecD &px, RVecD &py, RVecD &E) {
      auto pt = sqrt(px*px + py*py);
      return pt[E>100];
   };
   f.Define("good_pt", CalcPt, {"px", "py", "E"})
    .Histo1D<RVecD>({"pt", "pt", 16, 0, 4}, "good_pt")->DrawCopy();
}
```
这是本教程想要推荐的写法。
- **特点**：**消灭了循环！**
    - `px * px`：这不是两个数相乘，而是两个**数组**对应的元素相乘（SIMD 向量化操作）。
    - `pt[E > 100]`：这是**布尔掩码（Boolean Masking）**。意思是“从 `pt` 数组中取出对应 `E` 数组大于 100 的那些元素”。
- **优点**：代码极短，像 Python 的 NumPy 一样直观，而且运行速度极快。
```
void WithRDataFrameVecOpsJit()
{
   RDataFrame f(treename, filename.Data());
   f.Define("good_pt", "sqrt(px*px + py*py)[E>100]")
    .Histo1D({"pt", "pt", 16, 0, 4}, "good_pt")->DrawCopy();
}
```
这是最“偷懒”的写法。
- **特点**：甚至连 C++ 函数都不写了，直接把逻辑写成一个**字符串**。
    - `"sqrt(px*px + py*py)[E>100]"`
- **原理**：ROOT 包含一个 Cling 解释器，它会在运行时（Just-In-Time）编译这个字符串。
- **优点**：非常适合写配置文件或快速探索数据。
```
void df017_vecOpsHEP()
{
   // We plot four times the same quantity, the key is to look into the implementation
   // of the functions above.
   auto c = new TCanvas();
   c->Divide(2,2);
   c->cd(1);
   WithTTreeReader();
   c->cd(2);
   WithRDataFrame();
   c->cd(3);
   WithRDataFrameVecOps();
   c->cd(4);
   WithRDataFrameVecOpsJit();
}
```
- 思路
	- 输入：一堆乱七八糟的粒子（有高能的，有低能的）。
	- 处理：算出每个粒子的 $p_T$，然后把低能粒子剔除。
	- 输出：一张只包含高能粒子 $p_T$ 分布的直方图（什么能量有几个粒子）
### 检验粒子是否存在
我们要挑战一个“假想敌”，叫做 **空假设 (Null Hypothesis, $H_0$)**。
- **$H_0$（背景假设）**：假设这世界上**没有**这个新粒子（即信号强度 `SigXsecOverSM` = 0）。一切看到的波动都是纯背景噪声。
- 计算在 $H_0$ 成立的情况下，产生当前实验数据（`obsData`）的概率（**p-value**）。
	- 如果 p-value 极小（比如小于 $2.8 \times 10^{-7}$），说明“纯背景产生这种数据”简直是天方夜谭。
	- 于是**拒绝 $H_0$**，也就是**宣布“发现”了新粒子**（Discovery）。
	- 通常把 p-value 换算成 **显著性 (Significance, $Z$)**。物理学界的黄金标准是 **$Z > 5$ (5 sigma)**。
### 实战代码：计算发现显著性（代码保存在`significance.C`）
ROOT 的 `RooStats` 提供了一个神器叫 **`AsymptoticCalculator`（渐进计算器）**，它利用数学公式（基于威尔克斯定理）能瞬间算出结果，不需要跑几千次模拟。
```
using namespace RooStats;
using namespace RooFit;

void significance() {
    // 1. 打开文件并获取 Workspace
    TFile* file = new TFile("ref_6.16_example_UsingC_combined_meas_model.root");
    RooWorkspace* w = (RooWorkspace*)file->Get("combined");

    // 2. 获取 ModelConfig (模型的总管，包含了pdf、参数、数据定义的配置包)
    // 根据Print结果，它在 "generic objects" 里，名字通常叫 "ModelConfig"
    ModelConfig* mc = (ModelConfig*)w->obj("ModelConfig");
    
    // 3. 获取实际数据
    RooAbsData* data = w->data("obsData");

    // 4. 定义我们要挑战的“空假设” (H0: 纯背景，无信号)
    // 获取感兴趣的参数 (POI)，即 SigXsecOverSM
    RooRealVar* poi = (RooRealVar*)mc->GetParametersOfInterest()->first();
    poi->setVal(0); // 设置为 0，代表假设没有信号
    
    // 把这个参数状态保存为一个快照，命名为 "BkgOnly"
    ModelConfig* bkgModel = (ModelConfig*)mc->Clone();
    bkgModel->SetSnapshot( *poi ); 

    // 5. 设置计算器 (AsymptoticCalculator)
    // 输入：数据，H1模型(有信号)，H0模型(无信号)
    AsymptoticCalculator ac(*data, *bkgModel, *mc);
    
    // 设置为“单边计算”：因为我们只关心信号多出来的正波动（发现），不关心信号比背景还少的情况
    ac.SetOneSidedDiscovery(true); 

    // 6. 开始计算！
    // 问计算器：这个数据和 H0 (mu=0) 相比，差异有多大？
    HypothesisTestResult* result = ac.GetHypothesisTestResult(*data);

    // 7. 打印结果
    result->Print(); 
    
    double pvalue = result->NullPValue();
    double significance = result->Significance();

    cout << "------------------------------------------------" << endl;
    cout << "P-value      = " << pvalue << endl;
    cout << "Significance = " << significance << " sigma" << endl;
    cout << "------------------------------------------------" << endl;

    if (significance > 5.0) {
        cout << ">>> 恭喜！这是 5 sigma 发现！ <<<" << endl;
    } else if (significance > 3.0) {
        cout << ">>> 有趣！看到了 3 sigma 的迹象 (Evidence) <<<" << endl;
    } else {
        cout << ">>> 遗憾，这就是普通的背景波动。 <<<" << endl;
    }
}
```
- 代码思路
	1. 用 `RooRealVar` 定义变量。
	2. 用变量构建模型（PDF）和读取数据 (`RooAbsData`)。
	3. 把模型和数据导入 `RooWorkspace` 保存起来。
	4. (进阶) 创建一个 `ModelConfig`，从 Workspace 里挑出哪些是信号参数，哪些是干扰参数，准备进行统计计算。
- **Null P-Value**: 比如 `0.0013`. 这意味着如果你重复做一万次只有背景的实验，有 13 次会偶然出现这么大的信号。
- **Significance**: 比如 `3.0123`. 这是把 p-value 换算成正态分布的“标准差”。
    - **Z > 5**: **发现 (Discovery)** —— 我们可以开香槟发论文了。
    - **Z > 3**: **迹象 (Evidence)** —— 有点意思，但还需要更多数据。
    - **Z < 2**: **没戏** —— 这基本就是背景的统计涨落。

## data_C1_0520_wave.root数据处理初尝试
在文件夹空白位置右键然后Open in terminal就可以直接打开一个位于这个地址的终端
```
vip@vip:/mnt/hgfs/Root$ root -l data_C1_0520_wave.root
root [0] 
Attaching file data_C1_0520_wave.root as _file0...
(TFile *) 0x633131c5f740
root [1] .ls
TFile**		data_C1_0520_wave.root	
 TFile*		data_C1_0520_wave.root	
  KEY: TTree	tree;1	GDDAQ Multi-Crate sort Data
root [2] tree->Print()
******************************************************************************
*Tree    :tree      : GDDAQ Multi-Crate sort Data                            *
*Entries :  1278233 : Total =       120535521 bytes  File  Size =   10949063 *
*        :          : Tree compression factor =  11.03                       *
******************************************************************************
*Br    0 :sr        : sr/S                                                   *
*Entries :  1278233 : Total  Size=    2563964 bytes  File Size  =     361719 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression=   7.08     *
*............................................................................*
*Br    1 :pileup    : pileup/O                                               *
*Entries :  1278233 : Total  Size=    1282349 bytes  File Size  =      10163 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 126.07     *
*............................................................................*
*Br    2 :outofr    : outofr/O                                               *
*Entries :  1278233 : Total  Size=    1282349 bytes  File Size  =       9904 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 129.36     *
*............................................................................*
*Br    3 :cid       : cid/S                                                  *
*Entries :  1278233 : Total  Size=    2564049 bytes  File Size  =      19471 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression= 131.59     *
*............................................................................*
*Br    4 :sid       : sid/S                                                  *
*Entries :  1278233 : Total  Size=    2564049 bytes  File Size  =     616789 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression=   4.15     *
*............................................................................*
*Br    5 :ch        : ch/S                                                   *
*Entries :  1278233 : Total  Size=    2563964 bytes  File Size  =     906630 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression=   2.83     *
*............................................................................*
*Br    6 :evte      : evte/s                                                 *
*Entries :  1278233 : Total  Size=    2564134 bytes  File Size  =    2113479 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression=   1.21     *
*............................................................................*
*Br    7 :ts        : ts/L                                                   *
*Entries :  1278233 : Total  Size=   10254734 bytes  File Size  =    6121227 *
*Baskets :      321 : Basket Size=      32000 bytes  Compression=   1.67     *
*............................................................................*
*Br    8 :cfd       : cfd/S                                                  *
*Entries :  1278233 : Total  Size=    2564049 bytes  File Size  =      19471 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression= 131.59     *
*............................................................................*
*Br    9 :cfdft     : cfdft/O                                                *
*Entries :  1278233 : Total  Size=    1282304 bytes  File Size  =       9863 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 129.90     *
*............................................................................*
*Br   10 :cfds      : cfds/S                                                 *
*Entries :  1278233 : Total  Size=    2564134 bytes  File Size  =      19552 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression= 131.05     *
*............................................................................*
*Br   11 :esumf     : esumf/O                                                *
*Entries :  1278233 : Total  Size=    1282304 bytes  File Size  =       9863 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 129.90     *
*............................................................................*
*Br   12 :trae      : trae/i                                                 *
*Entries :  1278233 : Total  Size=    5127884 bytes  File Size  =      39003 *
*Baskets :      161 : Basket Size=      32000 bytes  Compression= 131.38     *
*............................................................................*
*Br   13 :leae      : leae/i                                                 *
*Entries :  1278233 : Total  Size=    5127884 bytes  File Size  =      39003 *
*Baskets :      161 : Basket Size=      32000 bytes  Compression= 131.38     *
*............................................................................*
*Br   14 :gape      : gape/i                                                 *
*Entries :  1278233 : Total  Size=    5127884 bytes  File Size  =      39003 *
*Baskets :      161 : Basket Size=      32000 bytes  Compression= 131.38     *
*............................................................................*
*Br   15 :base      : base/i                                                 *
*Entries :  1278233 : Total  Size=    5127884 bytes  File Size  =      39003 *
*Baskets :      161 : Basket Size=      32000 bytes  Compression= 131.38     *
*............................................................................*
*Br   16 :qsumf     : qsumf/O                                                *
*Entries :  1278233 : Total  Size=    1282304 bytes  File Size  =       9863 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 129.90     *
*............................................................................*
*Br   17 :qs        : qs[8]/i                                                *
*Entries :  1278233 : Total  Size=   41017942 bytes  File Size  =     309074 *
*Baskets :     1283 : Basket Size=      32000 bytes  Compression= 132.63     *
*............................................................................*
*Br   18 :etsf      : etsf/O                                                 *
*Entries :  1278233 : Total  Size=    1282259 bytes  File Size  =       9822 *
*Baskets :       41 : Basket Size=      32000 bytes  Compression= 130.44     *
*............................................................................*
*Br   19 :ets       : ets/L                                                  *
*Entries :  1278233 : Total  Size=   10255059 bytes  File Size  =      77581 *
*Baskets :      321 : Basket Size=      32000 bytes  Compression= 132.10     *
*............................................................................*
*Br   20 :ltra      : ltra/s                                                 *
*Entries :  1278233 : Total  Size=    2564134 bytes  File Size  =      19552 *
*Baskets :       81 : Basket Size=      32000 bytes  Compression= 131.05     *
*............................................................................*
*Br   21 :data      : data[ltra]/s                                           *
*Entries :  1278233 : Total  Size=    5145084 bytes  File Size  =      60288 *
*Baskets :      321 : Basket Size=      32000 bytes  Compression=  85.23     *
*............................................................................*
*Br   22 :dt        : dt[ltra]/s                                             *
*Entries :  1278233 : Total  Size=    5144434 bytes  File Size  =      59646 *
*Baskets :      321 : Basket Size=      32000 bytes  Compression=  86.14     *
*............................................................................*
root [4] tree->Draw("evte","sid==2&&ch==10")
(long long) 16798
root [5] tree->Draw("evte>>(5000,0,5000)","sid==2&&ch==10")
(long long) 16798
root [6] tree->Draw("sid*16+ch")
root [7] tree->Draw("sid*16+ch>>(128,0,128)")
```
- tree里边有一大堆分支，“evte”是能谱，可以单独调出某个ch的能谱，横坐标是道址，纵坐标是计数，这个道址和能量是有一个线性关系的，所以要先刻度
- 硬件地址与标识
	- **`cid` (Crate ID)**: **机箱号**。实验中通常有多个机箱（Crate），每个机箱插有多块采集卡。
	- **`sid` (Slot ID)**: **插槽号**。采集卡插在机箱的第几个插槽位置（通常对应具体的模块 ID）。
	- **`ch` (Channel)**: **通道号**。具体的信号输入通道（例如 Pixie-16 模块上的 0-15 号通道）。
		- 这里可能有很多sid，所以画图的时候要写明是哪一个sid的第几个ch
- 状态标志，用于判断该事件的数据质量
	- **`pileup`**: **堆积标志 (Pile-up Flag)**。
	    - `1 (True)`: 表示发生了信号堆积（两个粒子来得太近，波形叠在一起了），能量计算可能不准。
	    - `0 (False)`: 信号是干净的。
	- **`outofr`**: **超量程标志 (Out of Range)**。
	    - 表示信号幅度超过了 ADC 的动态范围（饱和/溢出）或低于底噪（下溢）。
	- **`sr`**: **状态寄存器 (Status Register)** 或 **采样率 (Sampling Rate)**。
	    - 通常包含更底层的硬件状态位（如触发类型、PLL 锁定状态等）。也可能是指该通道当前的采样频率索引。
- 时间信息，用于确定事件发生的时间，这是做符合测量（Coincidence）的基础。
	- **`ts` (Timestamp)**: **时间戳**。
	    - 最重要的时间信息，通常是 64 位整数，单位取决于数字化仪的时钟周期（例如 10ns 或 4ns）。
	- **`cfd` (Constant Fraction Discriminator)**: **恒比定时值 (Fine Time)**。
	    - 为了提高时间分辨率而计算的“精细时间”修正值，用于在粗略的时间戳（ts）之间进行插值，能实现亚纳秒级的时间精度。
	- **`cfdft`**: **CFD 强制触发标志 (CFD Force Trigger)**。
	    - 可能表示 CFD 算法是否成功找到了过零点，或者是否是因为超时而强制触发的。
	- **`cfds`**: **CFD 状态或源 (CFD Source/Status)**。
	    - 可能涉及 CFD 计算的具体参数或状态。
- 能量与波形参数 (Energy & DSP)，基于梯形滤波算法（Trapezoidal Filter）计算出的能量值。
	- **`trae` (Trapezoidal Energy)**: **梯形能量**。
	    - 这是通过 DSP 算法计算出的高精度能量值，通常是做能谱分析主要使用的变量。
	- **`evte` (Event Energy)**: **事件能量**。
	    - 这通常是板载 FPGA 快速计算出的能量值（可能是 16 位），精度可能略低于 `trae`，用于快速在线监视。
	- **`base` (Baseline)**: **基线值**。
	    - 脉冲到来之前的信号平均电平。计算能量时需要减去这个值。
	- **`leae` / `gape`**:
	    - 这两个比较特殊，推测是 **Leading Edge Energy (前沿能量)** 或 **Gap Energy (间隙能量)**。这可能是在不同的滤波参数（如 Peaking time 和 Gap time）下计算的中间能量值，用于做特殊的修正。
- 电荷积分与粒子鉴别 (QDC / PSD)，用于脉冲形状鉴别（PSD），常用于区分中子和伽马射线。
	- **`qs[8]` (Q-Sums)**: **电荷积分数组**。
	    - 这是一个包含 8 个元素的数组。数字化仪通常会在不同的时间窗口（如长门 Long Gate、短门 Short Gate、前背景 Pre-gate 等）内对波形进行积分。
	    - 通过比较 `qs` 中的不同元素（如长短门之比），可以进行粒子鉴别。
	- **`qsumf`**: **QDC 和标志 (Q-Sum Flag)**。
	    - 表示电荷积分计算是否有效。
	- **`esumf` / `etsf`**:
	    - 可能对应 **Energy Sum Flag** 和 **External Timestamp Flag**。
- 波形数据 (Trace / Waveform)，需要像示波器一样看原始信号时，就用这里。
	- **`ltra` (Length of Trace)**: **波形长度**。
	    - 表示 `data` 数组中有多少个采样点。
	- **`data[ltra]`**: **波形数据**。
	    - 这是原始的 ADC 采样值数组。画出来就是脉冲的形状。
	- **`dt[ltra]`**: **数字波形 (Digital Trace)**。
	    - 通常数字化仪在记录模拟信号（data）的同时，还会记录每一时刻的逻辑状态（如触发位、CFD 状态位等）。这个数组通常和 `data` 一一对应，用于调试触发逻辑。    
### 把所有通道能量谱以TH1F图的类型保存在一个新的root文件中
```
#include <TFile.h>
#include <TTree.h>
#include <TH1F.h>
#include <TString.h>
#include <iostream>
#include <map>

void BuildSpectra() {
    // ================= 配置区域 =================
    const char* inputFileName = "data_C1_0520_wave.root"; // 输入文件名
    const char* outputFileName = "Spectra_Output.root";   // 输出文件名
    int channelsPerSlot = 16;       // 假设每张卡16个通道(Pixie-16)
    int nBins = 32768;              // 能谱的道数 (通常是 32k 或者 65k)
    double minE = 0;                // 能谱最小值
    double maxE = 65536;            // 能谱最大值 (UShort_t 最大值)
    // ===========================================

    // 1. 打开原始文件
    TFile *fIn = new TFile(inputFileName, "READ");
    if (!fIn || fIn->IsZombie()) {
        std::cerr << "Error: Cannot open input file!" << std::endl;
        return;
    }

    TTree *tree = (TTree*)fIn->Get("tree");
    if (!tree) {
        std::cerr << "Error: Cannot find TTree 'tree'!" << std::endl;
        return;
    }

    // 2. 设置分支地址 (只读取我们需要的变量)
    Short_t sid, ch;
    UShort_t evte; // 注意: Tree Print中evte是s(UShort_t)类型

    tree->SetBranchAddress("sid", &sid);
    tree->SetBranchAddress("ch", &ch);
    tree->SetBranchAddress("evte", &evte);

    // 3. 创建输出文件
    TFile *fOut = new TFile(outputFileName, "RECREATE");

    // 4. 使用 map 动态管理直方图
    // Key = 全局通道ID (Global ID), Value = 直方图指针
    std::map<int, TH1F*> histMap;

    Long64_t nEntries = tree->GetEntries();
    std::cout << "Processing " << nEntries << " entries..." << std::endl;

    // 5. 循环处理每一条数据
    for (Long64_t i = 0; i < nEntries; i++) {
        tree->GetEntry(i);

        // 显示进度 (每10万条显示一次)
        if (i % 100000 == 0) std::cout << "Processed " << i << " events..." << std::flush << "\r";

        // 计算全局唯一ID
        // 逻辑: 如果 sid=2, ch=0 -> globalID = 32
        int globalID = sid * channelsPerSlot + ch;

        // 检查这个ID的直方图是否已经存在
        if (histMap.find(globalID) == histMap.end()) {
            // 如果不存在，创建一个新的 TH1F
            TString hName = Form("h_sid%d_ch%d_gid%d", sid, ch, globalID);
            TString hTitle = Form("Energy Spectrum - Slot %d Ch %d (Global %d);Energy (ADC);Counts", sid, ch, globalID);
            
            // 创建并存入 map (在 fOut 目录下创建)
            fOut->cd(); 
            TH1F *h = new TH1F(hName, hTitle, nBins, minE, maxE);
            histMap[globalID] = h;
        }

        // 填充能量
        histMap[globalID]->Fill(evte);
    }

    // 6. 保存所有直方图并关闭文件
    std::cout << "\nWriting histograms to " << outputFileName << "..." << std::endl;
    
    fOut->cd();
    // 遍历 map 将所有图写入文件
    for (auto const& [id, hist] : histMap) {
        // 可以在这里做一些简单的统计打印，例如：
        if (hist->GetEntries() > 0) {
            std::cout << "Saving: " << hist->GetName() << " (Entries: " << hist->GetEntries() << ")" << std::endl;
            hist->Write(0, TObject::kOverwrite);
```
这个地方不能写`hist->write()`不然每次运行都会产生新的重复文件在root里
```
        }
    }

    fOut->Close();
    fIn->Close();

    std::cout << "Done! All spectra saved." << std::endl;
}
```
可以读取然后画一个直方图出来，改一下x坐标范围，然后用之前那个peakstr.C代码寻峰，fitpannel拟合找中心值，这些每一个ch都要得到这几个峰位
### 遍历每个TH1F文件并进行寻峰拟合
```
#include <TFile.h>
#include <TH1F.h>
#include <TF1.h>
#include <TSpectrum.h>
#include <TKey.h>
#include <TList.h>
#include <iostream>
#include <fstream>
#include <algorithm>
#include <vector>
#include <set> 

// 定义结构体
struct PeakInfo {
    Double_t position;
    Double_t height;
};

// 排序规则 1：按位置 X 从小到大 (最后输出用)
bool comparePeaksByPosition(const PeakInfo &a, const PeakInfo &b) {
    return a.position < b.position;
}

// 【新增】排序规则 2：按高度 Y 从大到小 (筛选最强峰用)
bool comparePeaksByHeight(const PeakInfo &a, const PeakInfo &b) {
    return a.height > b.height; // 高的在前
}

void AutoFitPeaks() {
    // ================== 参数配置 ==================
    double globalSearchMin = 1000.0;   // 范围下限 
    double globalSearchMax = 8000.0;  // 范围上限
    double globalFitWindow = 5.0;    // 拟合窗口 (+/-) 改大一点点适应宽峰
    int    globalExpectedPeaks = 3;   // 找几个峰
    // =============================================

    TFile *fIn = new TFile("Spectra_Output.root", "READ");
    if (!fIn || fIn->IsZombie()) {
        std::cout << "Error: Cannot open input file!" << std::endl;
        return;
    }
    
    // 使用 RECREATE 模式，这会强制覆盖旧的 Spectra_Fitted.root
    TFile *fOut = new TFile("Spectra_Fitted.root", "RECREATE");
    
    std::ofstream outfile("FitResults.csv");
    outfile << "Histogram_Name,Peak1_Mean,Peak1_Sigma,Peak2_Mean,Peak2_Sigma,Peak3_Mean,Peak3_Sigma" << std::endl;
    
    // 用于去重 (防止处理多个 Cycle)
    std::set<std::string> processedNames; 
    
    // 遍历文件
    TIter next(fIn->GetListOfKeys());
    TKey *key;
    
    while ((key = (TKey*)next())) {
        std::string hname = key->GetName();

        if (processedNames.count(hname)) continue;
        processedNames.insert(hname);

        TObject *obj = key->ReadObj();
        
        if (obj->InheritsFrom("TH1F")) {
            TH1F *h = (TH1F*)obj;
            h->SetDirectory(0); 

            TList *funcList = h->GetListOfFunctions();
            if (funcList) funcList->Delete();

            std::cout << "Processing: " << hname << " ... ";

            // --- 步骤 A: 设定范围 ---
            h->GetXaxis()->UnZoom(); 
            h->GetXaxis()->SetRangeUser(globalSearchMin, globalSearchMax); 

            TSpectrum *s = new TSpectrum(50); 
            // 阈值设为 0.1 (10%)，可以过滤掉更多小毛刺
            int nFound = s->Search(h, 2, "nobackground", 0.10); 

            Double_t *xpeaks = s->GetPositionX();
            Double_t *ypeaks = s->GetPositionY();

            // --- 步骤 B: 【核心修改】筛选逻辑 ---
            std::vector<PeakInfo> allPeaks;
            
            // 1. 先把所有在范围内的峰收集起来
            for (int p = 0; p < nFound; p++) {
                if (xpeaks[p] >= globalSearchMin && xpeaks[p] <= globalSearchMax) {
                    allPeaks.push_back({xpeaks[p], ypeaks[p]});
                }
            }

            // 2. 关键一步：先按【高度】从大到小排序
            std::sort(allPeaks.begin(), allPeaks.end(), comparePeaksByHeight);

            // 3. 截取前 N 个最高的 (扔掉那些矮的噪声峰)
            std::vector<PeakInfo> finalPeaks;
            int nToSave = std::min((int)allPeaks.size(), globalExpectedPeaks);
            for(int i=0; i<nToSave; i++) {
                finalPeaks.push_back(allPeaks[i]);
            }

            // 4. 最后再按【位置】从小到大排序 (为了 CSV 输出顺序 Peak1(左)->Peak2(中)->Peak3(右))
            std::sort(finalPeaks.begin(), finalPeaks.end(), comparePeaksByPosition);

            // --- 步骤 C: 拟合 ---
            outfile << hname;
            
            for (int i = 0; i < globalExpectedPeaks; i++) {
                if (i < finalPeaks.size()) {
                    double peakX = finalPeaks[i].position;
                    
                    // 创建函数
                    TF1 *gfit = new TF1(Form("gfit_%s_%d", hname.c_str(), i), "gaus", peakX - globalFitWindow, peakX + globalFitWindow);
                    gfit->SetParameter(1, peakX); 
```
在简单的交互式操作中，可以直接用 `h->Fit("gaus")`，但**自动化、多峰、批量处理**的场景下，直接 `Fit("gaus")` 是**绝对不行**的。
- 它是 "单峰" 函数，但这里有 "三个峰"
	- 试图用**这一个**高斯曲线去拟合**整个直方图**（或者当前显示的范围）
- 避免“拟合跑偏” (Initial Parameter Guessing)
	- ROOT 会自动猜测初始参数（高度、位置、宽度）
	    - 把旁边的一个高一点的噪声误判为峰顶，或者把两个挨得很近的峰当成一个宽峰。
- **手动定义 TF1 的优势**：
	- `gfit->SetParameter(1, peakX);`寻峰定下峰位置，把这个值强制喂给拟合函数作为**初始值**。这样拟合过程就非常稳定，几乎不会跑偏。
- 限定拟合范围 (The "R" Option)
	- `h->Fit(gfit, "RQ+");`这里的 **"R"** 代表 **Range**。如果不定义 TF1 并设定范围（`peakX - fitWindow, peakX + fitWindow`），ROOT 默认使用直方图的全范围。
```
new TF1( 参数1, 参数2, 参数3, 参数4 );
```
- 参数 1: `Form("gfit_%s_%d", hname.c_str(), i)`
	- **含义：函数的唯一名称 (Name)**
	- **`Form(...)`**：这是 ROOT 提供的一个类似于 C语言 `sprintf` 的格式化工具。
	    - `"gfit_%s_%d"`：这是模板。`%s` 代表字符串，`%d` 代表整数。
	    - `hname.c_str()`：填入当前直方图的名字（例如 `h_sid2_ch0_gid32`）。
	    - `i`：填入当前是第几个峰（0, 1, 或 2）。
	- 最终生成的函数名字会像这样：`"gfit_h_sid2_ch0_gid32_0"`。
	- 为什么要这么麻烦？
	    如果给所有拟合函数都起名叫 "myFit"，当你处理下一个峰时，ROOT 会把上一个叫 "myFit" 的函数覆盖掉/删掉。
- 参数 2: `"gaus"`
	- **含义：预定义的数学公式 (Formula)**
	- **`"gaus"`**：这是 ROOT 内置的一个关键字，代表标准高斯函数。
	- 数学表达式：$f(x) = p_0 \cdot \exp\left(-\frac{1}{2} \left(\frac{x - p_1}{p_2}\right)^2\right)$
    - ROOT 会自动建立 3 个参数：
	    - **Parameter [0] (Constant)**: 峰的高度（归一化系数）。
	    - **Parameter [1] (Mean)**: 峰的中心位置（均值）。
	    - **Parameter [2] (Sigma)**: 峰的宽度（标准差）。
- 后边两个参数是拟合的上下限
```
                    // 限制一下 Sigma，防止拟合失败变成一条直线
                    gfit->SetParLimits(2, 0.5, 50.0); 

                    h->Fit(gfit, "RQ+"); 

                    outfile << "," << gfit->GetParameter(1) << "," << gfit->GetParameter(2);
                } else {
                    outfile << ",0,0";
                }
            }
            outfile << std::endl;

            // --- 步骤 D: 保存 ---
            fOut->cd();    
            h->Write();    
            
            delete s;
            delete h; 
            
            std::cout << "Done." << std::endl;
        }
    }

    outfile.close();
    fOut->Close(); 
    fIn->Close();  
    
    std::cout << "Data saved to Spectra_Fitted.root" << std::endl;
}
```

## ADC,TDC信号处理
### 蒙卡模拟生成数据
```
void treeADC()
{
  const Double_t D = 500.;//cm, distance between target and the scin.(Center)
  const Double_t L = 100.;//cm, half length of the scin.
  const Double_t dD = 5.;//cm, thickness of the scin.
  const Double_t tRes = 1.;//ns, time resolution(FWHM) of the scintillator.
  const Double_t Lambda = 380.;//cm, attenuation lenght of the scin.
  const Double_t qRes = 0.1;//relative energy resolution(FWHM) of the scin. 
  const Double_t Vsc = 7.5;//ns/cm, speed of light in the scin.
 //neutron & gamma
  const Double_t En0 = 50;//MeV, average neutron energy
  const Double_t EnFWHM = 50.;//MeV, energy spread of neutron(FWHM)
  const Double_t Eg0 = 10;//MeV, gamma energy  

  const Double_t RatioNeutron = 0.75;//ratio of neutron 
  const Double_t RatioGamma = 0.05;//ratio of gamma; 

  //charged particle hited in Neutron Detector, 
  const Double_t RatioLP = 0.1;//ratio of LP
  const Double_t Elp0 = 50;//MeV
  const Double_t ElpFWHM = 50;//MeV energy spread of LP(FWHM)

  //ADC
  const Double_t ADCuGain = 40;//1MeV=40ch.
  const Double_t ADCdGain = 45;
  const Double_t ADCuPed = 140;//baseline of ADC for upper side
  const Double_t ADCdPed = 130;//                for bottom side
  const Double_t ADCnoise = 10;//sigma of noise
  const Int_t    ADCoverflow = 4095;

  //TDC
  const Double_t TriggerDelay = 15;//ns, trigger延迟,将感兴趣的时间信号放在TDC量程以内。
  const Double_t TDCuCh2ns = 25.;//1ns=25ch.
  const Double_t TDCdCh2ns = 28.;
  const Int_t TDCoverflow = 4095;

  TFile *opf = new TFile("treeADC.root","recreate");//新文件tree.root的指针 *opf
  TTree *opt = new TTree("tree","tree structure");//新tree的指针 *opt

  // 定义tree的branch变量
  Double_t x;
  Double_t e;
  int pid;    //0/1/2/3: gamma/neutron/LP/No_particle
  Double_t tof, ctof;
  Double_t tu, td;
  Double_t qu, qd;
  Int_t itu, itd;//TDC
  Int_t iqu, iqd;//ADC

  Double_t diff;

  Double_t tuoff = 5.5;//time offset
  Double_t tdoff = 20.4;//time offset

  // 将变量分支添加到tree结构中,第一个参数为变量名称，第二个为上面定义的变量地址，第三个为变量的类型说明，D表示Double_t。
  opt->Branch("x", &x, "x/D");//position
  opt->Branch("e", &e, "e/D");//energy
  opt->Branch("tof", &tof, "tof/D");//time of flight
  opt->Branch("ctof",&ctof,"ctof/D");//TOF from exp. data
  opt->Branch("pid", &pid, "pid/I");
  // raw time and energy
  opt->Branch("tu", &tu, "tu/D");
  opt->Branch("td", &td, "td/D");
  opt->Branch("qu", &qu, "qu/D");
  opt->Branch("qd", &qd, "qd/D");

  // energy in ADC, time in TDC 注意，以下Branch变量声明的类型为 Integer，
  opt->Branch("itu", &itu, "itu/I");
  opt->Branch("itd", &itd, "itd/I");
  opt->Branch("iqu", &iqu, "iqu/I");
  opt->Branch("iqd", &iqd, "iqd/I");

  opt->Branch("diff",&diff,"diff/D");

  TRandom3 *gr=new TRandom3(0);
  // 循环，逐事件往tree结构里添加对应分支信息。
  for(int i = 0 ; i < 1000000 ; i++){
    x = gr->Uniform(-L, L);
    Double_t Dr = D + gr->Uniform(-0.5, 0.5) * dD;
    Double_t d = TMath::Sqrt(Dr * Dr + x * x);//cm, flight path

    //pid
    pid = -1;

    Double_t ratio = gr->Uniform();
    if(ratio < RatioGamma) 
        pid = 0;
    if(ratio >= RatioGamma && ratio < RatioGamma + RatioNeutron) 
        pid = 1;
    if(ratio >= RatioGamma + RatioNeutron 
        && ratio < RatioGamma + RatioNeutron + RatioLP) 
        pid = 2;
    if(ratio >= RatioGamma + RatioNeutron + RatioLP) 
        pid = 3;

    // energy & tof    
    if(pid == 0) {//gamma
       e = Eg0;
       tof = 3.333 * (d * 0.01);
      }
    if(pid == 1) { //neutron
       e = gr->Gaus(En0, EnFWHM/2.35); // neutron
       tof = 72.29824/TMath::Sqrt(e) * (d * 0.01);//ns
    }
    if(pid == 2) {//LP
      e = gr->Gaus(Elp0, ElpFWHM/2.35); // LP
      tof = 72.29824/TMath::Sqrt(e) * (d * 0.01);
    }
    if(pid == 3) {
      e = -1;
      tof = -1;
    }

    if(pid < 0 || isnan(tof)) continue; //check ZeroDivisionError

    if(pid == 3) {
      tu = -1;
      td = -1;
      qu = -1;
      qd = -1;
    }
    else {
      //time
      tu = tof + (L - x)/Vsc + tuoff - TriggerDelay;
      td = tof + (L + x)/Vsc + tdoff - TriggerDelay;
      //resolution
      tu += gr->Gaus(0, tRes/2.35); 
      td += gr->Gaus(0, tRes/2.35); 

     //energy
      Double_t Q0;
      if(pid != 2) Q0 = e * gr->Uniform();//neutron & gamma,
      else Q0 = e ; //charged particle，full energy。

      //resolution
      Q0=gr->Gaus(Q0, Q0*qRes/2.35);      

      //light transmission within the plastic
      qu = Q0 * TMath::Exp(-(L-x)/Lambda);
      qd = Q0 * TMath::Exp(-(L+x)/Lambda); 
    }

    //TDC, ADC
    if(pid == 3) {//
      itu = TDCoverflow;
      itd = TDCoverflow;
      iqu = ADCuPed + gr->Gaus(0, ADCnoise);
      iqd = ADCdPed + gr->Gaus(0, ADCnoise);
    }
    else {
      itu = tu * TDCuCh2ns;
      itd = td * TDCdCh2ns;
      iqu = qu * ADCuGain + gr->Gaus(ADCuPed, ADCnoise);;
      iqd = qd * ADCdGain + gr->Gaus(ADCdPed, ADCnoise);;

      if(iqu < 0) iqu = 0;//no negative values
      if(iqd < 0) iqd = 0;
    }

    //overflow check
    if(itu > TDCoverflow) itu = TDCoverflow;
    if(itd > TDCoverflow) itd = TDCoverflow;
    if(iqu > ADCoverflow) iqu = ADCoverflow;
    if(iqd > ADCoverflow) iqd = ADCoverflow;

  //digitization
    itu = Int_t(itu);
    itd = Int_t(itd);
    iqu = Int_t(iqu);
    iqd = Int_t(iqd);

   //difference in amplitude before and after digitization.
    diff = tu - itu;

    opt -> Fill();

  }
  // 将数据写入root文件中
  opt->Write();
  opf->Close();

}
```
下边来拆解一下这个代码
#### 第一阶段：上帝掷骰子（事件生成）
在真实实验中，每一次核反应打出什么东西是随机的。代码利用 `TRandom3` 这个“上帝之手”来模拟这种这一枪打出了什么粒子（PID），能量是多少（Energy）
1. 什么粒子(PID)
    - 代码先扔一个骰子（0到1之间的小数）。
    - 如果数值在 0~0.05 之间，就假装这次打出的是**伽马射线**。
    - 如果在 0.05~0.80 之间，就是**中子**。
    - 还有一部分概率给**本底噪声**（也就是之前说的束流触发，探测器没打中东西，光在空跑）。
2. 能量多少(Energy)
    - 确定了是中子后，再扔一个骰子决定它的能量。通常中子能量服从一个高斯分布（有的快，有的慢，但在某个值附近最多）
设定概率和能量分布，定义规则
```
// 发生概率：75%中子，5%伽马，10%带电粒子(LP)，剩下的10%是本底
const Double_t RatioNeutron = 0.75; 
const Double_t RatioGamma = 0.05;
const Double_t RatioLP = 0.1;

// 能量设定：中子平均50MeV，伽马固定10MeV
const Double_t En0 = 50;      // 中子平均能量
const Double_t EnFWHM = 50.;  // 中子能量分散范围（高斯分布的宽窄）
const Double_t Eg0 = 10;      // 伽马能量
```
掷骰子决定命运（循环体内）生成事件
```
// 扔一个 0-1 的骰子
Double_t ratio = gr->Uniform(); 

// --- 决定身份 (PID) ---
// 如果骰子点数 < 0.05，就是伽马(pid=0)
if(ratio < RatioGamma) pid = 0;
// 如果点数在 0.05 到 0.80 之间，就是中子(pid=1)
if(ratio >= RatioGamma && ratio < RatioGamma + RatioNeutron) pid = 1;
// ... (后面判断 LP 和 本底 同理)

// --- 决定能量 (Energy) ---
// 如果是伽马，能量是固定的
if(pid == 0) e = Eg0; 
// 如果是中子，能量每次都不一样，围绕50MeV波动 (高斯分布)
if(pid == 1) e = gr->Gaus(En0, EnFWHM/2.35); 
```
#### 第二阶段：飞行过程
1. **击中哪里？(Position $x$)**
    - 探测器是一根长条（比如2米长）。粒子可能打在正中间，也可能打在边缘。代码随机生成一个位置 $x$，模拟粒子“随机”打在了探测器条的不同部位。
2. **跑了多远？(Distance $d$)**
    - 根据勾股定理，算出靶点到击中点 $x$ 的直线距离。
3. **跑了多久？(TOF)**
    - 这是最关键的物理量。
    - 如果是**伽马**，它以**光速**飞行，$t = d / c$。
    - 如果是**中子**，它比较重，飞得慢。根据它的动能 $E$，算出速度 $v$，再算出时间 $t = d / v$。
    - 代码里的那个 `72.298...` 就是把中子质量和光速常数乘在一起算出的系数。
定义部分
```
const Double_t D = 500.;   // 跑道长度：靶到探测器中心 5米 (500cm)
const Double_t L = 100.;   // 终点线宽度：探测器半长 1米
const Double_t dD = 5.;    // 终点线厚度：探测器厚 5cm
```
循环体内
```
// --- 决定击中位置 (x) ---
// 粒子不可能总打中心，在 -100cm 到 +100cm 之间随机落点
x = gr->Uniform(-L, L); 

// --- 计算实际路程 (d) ---
// 考虑探测器厚度(dD)带来的微小距离波动
Double_t Dr = D + gr->Uniform(-0.5, 0.5) * dD; 
// 勾股定理：实际飞行距离 d = sqrt(垂直距离^2 + 偏心距离^2)
Double_t d = TMath::Sqrt(Dr * Dr + x * x);

// --- 计算飞行时间 (TOF) ---
if(pid == 0) { // 伽马是光速
   // 3.333 是光跑1米需要的纳秒数。 d*0.01 是把cm换成m
   tof = 3.333 * (d * 0.01); 
}
if(pid == 1) { // 中子是慢跑
   // 物理公式 t = 距离 / 速度。
   // 72.29... 是常数，用于通过能量(e)算出中子速度
   tof = 72.29824/TMath::Sqrt(e) * (d * 0.01);
}
```
#### 第三阶段：发光与传输（探测器内部物理）
粒子一头撞进了塑料闪烁体，这一步是从“粒子物理”变成了“光学”。研究粒子撞击后，光在塑料条里怎么跑（时间延迟），怎么变暗（衰减）。
1. 闪光 (Scintillation)
    - 粒子把能量传递给塑料分子，产生闪光。能量越高，光越强 ($Q_0$)。
2. 光的赛跑 (Light Transmission)
    - 这根塑料条两头各有一个“眼睛”（光电倍增管 PMT）。光在这个条子里向两头跑。
    - 时间差：假如粒子打在左边，光跑到左头就快，跑到右头就慢。代码通过 $x$ 计算了光分别到达两端的时间 ($t_u, t_d$)。
    - 亮度差（衰减）：塑料条不是完全透明的，像有雾一样。光跑得越远，亮度越暗。代码用了指数衰减公式 `Exp(-(L-x)/Lambda)`，模拟光在传输过程中慢慢变弱的现象。
设定介质属性（定义部分）
```
const Double_t Vsc = 7.5;     // 光在塑料里的速度 (7.5 ns/cm，比真空慢)
const Double_t Lambda = 380.; // 光的衰减长度 (跑3.8米衰减到原来的1/e)
```
循环体内（这一步计算的是物理真值，还没加误差）
```
// --- 计算光传输时间 (tu, td) ---
// tu (Up端时间) = 粒子飞来的时间(tof) + 光跑到上端的时间 + 线路固定延迟 - 触发延迟
// (L - x) 是击中点到上端的距离
tu = tof + (L - x)/Vsc + tuoff - TriggerDelay;
// (L + x) 是击中点到下端的距离
td = tof + (L + x)/Vsc + tdoff - TriggerDelay;

// --- 计算光的衰减 (qu, qd) ---
// Q0 是初始发光量。这里有个小细节：中子产生的初始光也要抖动一下(Uniform)
if(pid != 2) Q0 = e * gr->Uniform(); 

// 光跑得越远 (L-x 越大)，剩下的光越少。用指数衰减公式。
qu = Q0 * TMath::Exp(-(L-x)/Lambda);
qd = Q0 * TMath::Exp(-(L+x)/Lambda);
```
#### 第四阶段：电子学模拟（从物理量到数字）
这是为了让模拟数据看起来更像“真数据”。加上现实世界的“不完美”（误差、噪声、基线），最后变成整数（ADC/TDC数值）。
1. 模糊化 (Resolution/Smearing)
    - 真实世界里，测100次同样的东西，结果会像一个小山包一样波动。
    - 代码给计算出的精确时间 ($t$) 和能量 ($Q$) 加上了一个高斯噪声 (`gr->Gaus`)。这一步把完美的理论值，变成了带有误差的测量值。
2. 加底座 (Pedestal)
    - 就是理论基础ADC那一块提到的 Pedestal，人为给能量信号加上了一个基数（比如140道），模拟电子学的基线。
3. 加电子噪声 (Electronic Noise)
    - 除了物理测量的误差，电子元器件本身也有热噪声。代码在基线附近又加了一层随机抖动。
4. 变成格子 (Digitization)
    - 现实中的时间是连续的，但 TDC 只有 4096 个格子。
    - 代码最后把连续的浮点数强制变成了整数 (`Int_t`)。这一步模拟了数模转换过程中的精度丢失。
设定仪器缺陷和参数（定义部分）
```
// 误差参数
const Double_t tRes = 1.;     // 时间测不准，有 1ns 的误差
const Double_t ADCnoise = 10; // 能量测不准，有 10道 的噪声

// ADC参数 (能量尺子)
const Double_t ADCuGain = 40; // 增益：1MeV 变成 40道
const Double_t ADCuPed = 140; // 垫脚石：没信号时读数是 140

// TDC参数 (时间秒表)
const Double_t TDCuCh2ns = 25.; // 精度：1ns 分成 25个格子
const Int_t TDCoverflow = 4095; // 量程：最大只能记到 4095
```
模拟测量过程（循环体内）
```
// --- 步骤A：加上测量误差 (Resolution) ---
// 在完美的物理时间上，加一个高斯随机数，模拟测不准
tu += gr->Gaus(0, tRes/2.35); 
td += gr->Gaus(0, tRes/2.35);

// --- 步骤B：转换单位并加噪声 (Analog -> Digital Pre) ---
// TDC值 = 时间(ns) * 25 (ch/ns)
itu = tu * TDCuCh2ns; 
// ADC值 = 光强 * 增益 + 基线(Pedestal) + 电子噪声
iqu = qu * ADCuGain + gr->Gaus(ADCuPed, ADCnoise);

// --- 步骤C：检查量程 (Overflow) ---
// 如果爆表了，就强制记为最大值
if(itu > TDCoverflow) itu = TDCoverflow;
if(iqu > ADCoverflow) iqu = ADCoverflow;

// --- 步骤D：数字化取整 (Digitization) ---
// 电脑存的是整数，去掉小数点
itu = Int_t(itu);
iqu = Int_t(iqu);
```
### 调用并处理数据
## SortEvent
```
/************************************************************/
/*   ROOT Analysis Program for Digitizer Data Acquisition   */
/************************************************************/

#include <unistd.h>
#include <iostream>
#include <vector>
#include <TFile.h>
#include <TTree.h>
#include <TThread.h>
#include <TTreeCache.h>
#include <TTreePerfStats.h>
#include "setup.h"
#include "SortEvent.h"
#include "CAEN_DGTZ.h"
#include "DataShm.h"
#include "mapping.h"
#include "WaveProcess.h"

using namespace std;
extern DataShm* fp_shm;
```
-  `using namespace std;`
	- C++ 标准命名空间声明，意思是把C++ 的标准库函数（比如输入输出 `cout`、容器 `vector`、字符串 `string`）都放在一个叫 `std` 的“盒子里”。
    - 如果不写这行，每次用 `vector` 都得写成 `std::vector`。
-  `extern DataShm* fp_shm;`（多文件编程的概念）
	- **`DataShm*`**：这是一个**指针**，指向一个叫 `DataShm` 类型的对象。
	    - 结合名字看，`DataShm` 很可能是 **Data Shared Memory**（数据共享内存）的缩写。这是把处理好的数据“喂”给其他程序（比如在线监视器）的接口。
	- **`extern` (External)**：这是 C++ 的关键字，意思是 **“外部的”**。
	    - **翻译**：“编译器你好，有一个叫 `fp_shm` 的指针变量，它**不是在这个文件里定义的**（它可能定义在 `main.cc` 或者其他 `.cpp` 文件里），但我这个文件需要用到它。请你不要在这里创建一个新的，而是去连接到那个已经存在的变量上。”
	- **作用**：这是一个**全局变量的引用**。这就好比整个实验大厅只有一把“主钥匙”（定义在主文件里），而这个文件申请了一个“主钥匙的备份说明书”（extern），通过它也能操作那把钥匙。
```
Double_t Cal_Par[TOTAL_CH][2];
Double_t Cal_Par_FIT[TOTAL_CH][2];
Double_t Cal_Par_PHA[TOTAL_CH][2];
Double_t Cal_Par_SSD[TOTAL_CH][2];
```
- `Cal_Par[i][0]` 存放的是第 `i` 个通道的截距（Offset），`Cal_Par[i][1]` 存放的是第 `i` 个通道的斜率（Gain）
- `TOTAL_CH` 是一个宏定义（也就是一个常数，比如 64 或 128），代表整个实验系统中**探测器通道的总数**
- **`Cal_Par`**：通用的、或者默认算法使用的校准参数。
- **`Cal_Par_FIT`**：
    - **FIT** 指的是 **波形拟合 (Fitting)**。
    - 如果开启了宏定义 `DECODE_WAVE_PHA`，代码会用复杂的函数去拟合波形来算出能量。拟合算出来的“幅度”和硬件直接给出的“幅度”单位可能不一样，所以需要一套独立的校准参数。
- **`Cal_Par_PHA`**：
    - **PHA** 指的是 **Pulse Height Analysis**（板卡硬件直读）。
    - 这是给硬件直接输出的能量值用的校准参数。
- **`Cal_Par_SSD`**：
    - **SSD** 指的是 **Silicon Strip Detector**（硅微条探测器）。
    - 把硅探测器的参数单独分类存放。
```
int SortEvent(int filenum) 
{

	//read calibration parameters from files
	Read_Cal_Par();

	char filein[250], fileout[250];
	char TreeName[50], TreeTitle[50];

	DPP_PHA_Event_t DPPPHAEvent_Data;
	vector<DPP_PHA_Event_t> Vec_DPP_Data;
	vector<Waveform_t> Vec_DPP_Wave_Data;
```
- 输入runnum
- 读取校准参数`Read_Cal_Par()`
- 定义字符串
	- **`filein[250]`**: 用来存放**输入文件**的完整路径和文件名
	- **`fileout[250]`**: 用来存放**输出文件**的完整路径和文件名
	- **`TreeName[50]`**: 用来存放 ROOT Tree 的**名字**（Name）。ROOT 要求每个对象有个唯一的 ID，比如 `"DGTZ00"`（第0号板卡）。
	- **`TreeTitle[50]`**: 用来存放 ROOT Tree 的**标题**（Title）。这是一个给人看的描述，比如 `"Data for Board 00"`。
- 定义核心数据容器
	- **`DPP_PHA_Event_t`**: 这是一个自定义的结构体（Struct）。它里面就像一个表格的一行，包含了：能量、时间戳、通道号、板卡号等信息。
	- **`DPPPHAEvent_Data`**: 这是该结构体的一个实例（变量）。
	    - 循环时把当前正在处理的粒子的信息填进去，然后把这个变量写入 ROOT Tree。
	- **`vector`**: C++ 标准库里的“动态数组”（向量）。它可以根据需要自动伸缩大小。
	    - CAEN 的原始数据通常是以“Block”（块）或者是“Buffer”为单位存储的。一次读取操作（`GetDPPPHAEvent`）可能会读出来几百上千个事件。
	    - 我们不能用上面那个单个变量去接，必须用这个 `vector` 数组把这一批事件全部先接住，然后再一个一个处理。
	- **`Waveform_t`**: 也是一个自定义结构体，用来存波形数据的（比如 1024 个点的电压值）。
		- 用来接住波形数据。
	    - 即便是在 PHA 模式（只记能量和时间），很多时候我们也需要保留原始波形（用于调试、去堆积或者画图）。
	    - 这个向量与上面的 `Vec_DPP_Data` 是一一对应的：`Vec_DPP_Data[i]` 是第 i 个粒子的能量/时间，`Vec_DPP_Wave_Data[i]` 就是第 i 个粒子的波形图。
```
#ifdef DECODE_WAVE_PHA
	FIT_RESULT  Fit_Result;
#endif

	//read tree from rootfile and save into XXX_tmp.root

	if(INPUT_FILE_PATH[strlen(INPUT_FILE_PATH)-1] == '/')
		sprintf(filein,"%s%s%05d.root",INPUT_FILE_PATH,FILENAME_PREFIX,filenum);
	else
		sprintf(filein,"%s/%s%05d.root",INPUT_FILE_PATH,FILENAME_PREFIX,filenum);

		// FileIn
	TFile *FileIn = TFile::Open(filein,"READ");
	if(FileIn == NULL || FileIn->IsZombie()) {
		printf(" CAUTION: open the file %s error \n\n", filein);
		return -1;
	}
	FileIn->cd();

#ifdef FILE_TIME_SORT
	fp_shm->ClearShm();
	TThread* thread_map=new TThread("Mapping_Data",(TThread::VoidRtnFunc_t)Mapping_Data, (void*)(&filenum));
	thread_map->Run();
#endif

	// tmp file 
	if(OUTPUT_FILE_PATH[strlen(OUTPUT_FILE_PATH)-1] == '/')
		sprintf(fileout,"%s%s%05d_tmp.root",OUTPUT_FILE_PATH,FILENAME_PREFIX,filenum);
	else
		sprintf(fileout,"%s/%s%05d_tmp.root",OUTPUT_FILE_PATH,FILENAME_PREFIX,filenum);
	TFile *FileOut = new TFile(fileout,"RECREATE");
	if(FileOut->IsZombie()) {
		printf(" CAUTION: open the file %s error \n\n", fileout);
		delete FileOut;
		return -1;
	}
	FileOut->cd();

	TTree *DGTZ[MAXNB];
	for(int b=0; b<MAXNB; b++) {
		sprintf(TreeName,"DGTZ%02d",b);
		sprintf(TreeTitle,"A Tree for Board %02d",b);
		DGTZ[b] = new TTree(TreeName,TreeTitle);
		// src infor
		DGTZ[b] -> Branch("DGTZ_Type", &DPPPHAEvent_Data.DGTZ_Type, "DGTZ_Type/s");
		DGTZ[b] -> Branch("DGTZ_EntryNo", &DPPPHAEvent_Data.DGTZ_EntryNo, "DGTZ_EntryNo/l");
		DGTZ[b] -> Branch("DGTZ_BdAggregateNo", &DPPPHAEvent_Data.DGTZ_BdAggregateNo, "DGTZ_BdAggregateNo/s");
		DGTZ[b] -> Branch("DGTZ_ChAggregateNo", &DPPPHAEvent_Data.DGTZ_ChAggregateNo, "DGTZ_ChAggregateNo/s");
		//data info
		DGTZ[b] -> Branch("SysTime", &DPPPHAEvent_Data.SysTime, "SysTime/i");
		DGTZ[b] -> Branch("Ch", &DPPPHAEvent_Data.Ch, "Ch/s");
		DGTZ[b] -> Branch("Energy", &DPPPHAEvent_Data.Energy, "Energy/D");
		DGTZ[b] -> Branch("TriggerTimeTag", &DPPPHAEvent_Data.TriggerTimeTag, "TriggerTimeTag/l");
		DGTZ[b] -> Branch("TT", &DPPPHAEvent_Data.TT, "TT/s");
		DGTZ[b] -> Branch("PU", &DPPPHAEvent_Data.PU, "PU/s");
		DGTZ[b] -> Branch("Extras", &DPPPHAEvent_Data.Extras, "Extras/s");
		DGTZ[b] -> Branch("SampleNo", &DPPPHAEvent_Data.SampleNo, "SampleNo/s");
	}

	// FileIn and tree init
	FileIn->cd();
	TTree *Trigger = (TTree*) FileIn->Get("Trigger");
	if(Trigger == NULL)
	{
		printf("\n\n**** Get TTree in file %s error!****\n\n",filein);
		return -1;
	}
	Trigger->SetCacheSize(TREE_CACHE_SIZE);

	CAEN_DGTZ* ptr_data=new CAEN_DGTZ(Trigger);

	///// start proess
	Long64_t i = 0;
	int j = 0;
	time_t time_pre = time(NULL);
	time_t time_now = time(NULL);
	double test_value = 0.0;
	int    ret = 0;

	WaveProcess* ptr_wave_decode = new WaveProcess();

	Long64_t nEntry = Trigger->GetEntries();
	printf("****root2root:Start Process file %s***\n Total entries of Trigger Tree are %lld\n", filein, nEntry);

	Long64_t DGTZ_entry[MAXNB];
	memset(DGTZ_entry,0,sizeof(Long64_t)*MAXNB);

	for(i=0; i<nEntry; i++) 
	{
		if((nEntry/10 != 0) && i%(nEntry/10) == 0)
		{
			time_now = time(NULL);
			int time_remain = 0xFFFF;
			if(i >0)
			  	time_remain = 1.0*(time_now - time_pre)*(nEntry-i)/i;
			printf("process %3.0lf%% event file %s, estimated remaining time %d seconds ....\n", i*100.0/nEntry, filein, time_remain);
		}

		// read DPP data from src root file
		Vec_DPP_Data.clear();
		Vec_DPP_Wave_Data.clear();
		ptr_data->GetDPPPHAEvent(i, &Vec_DPP_Data, &Vec_DPP_Wave_Data);
		for(j = 0; j < Vec_DPP_Data.size(); j++)
		{
			//read event data and save into XXX_tmp.root
			memcpy(&DPPPHAEvent_Data, &Vec_DPP_Data[j], sizeof(DPP_PHA_Event_t));
			DPPPHAEvent_Data.BdEntryNo=DGTZ_entry[DPPPHAEvent_Data.BdNo];
			DGTZ[DPPPHAEvent_Data.BdNo]->Fill();
			DGTZ_entry[DPPPHAEvent_Data.BdNo]++;
			if((DPPPHAEvent_Data.Extras&0x8) == 0x8)
				continue;
			if(DPPPHAEvent_Data.BdNo==26 && DPPPHAEvent_Data.Ch==5)
				continue;
#ifdef FILE_TIME_SORT
			//E
			//PHAEvent
			//
			
			if(DPPPHAEvent_Data.BdNo < MAXNB_V1724)
				DPPPHAEvent_Data.Ch += DPPPHAEvent_Data.BdNo*8;
			else
				DPPPHAEvent_Data.Ch += DPPPHAEvent_Data.BdNo*16-MAXNB_V1724*8;

			//check the wave of silicon detector
			//DSSD,
		//	if(DPPPHAEvent_Data.Ch < SSD_CH_LOW)
		//	{
		//		if(DPPPHAEvent_Data.SampleNo > 0)
		//		{
		//			//using Digitizer results, or decode by itself?
		//			ptr_wave_decode->SetWaveTrace(Vec_DPP_Wave_Data[j].Probe1, (UInt_t)DPPPHAEvent_Data.SampleNo);
		//			if(ptr_wave_decode->CheckWave(&test_value) < 0)
		//			{
//		//				DPPPHAEvent_Data.PU = 100;
//		//				DPPPHAEvent_Data.Energy = test_value;
//		//				Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
		//				continue;
		//			}
		//		}
		//	}

			//check true saturation signal
			if((DPPPHAEvent_Data.Extras & 0x10) > 0 )
			{
#ifdef DEBUG_MSG_OFFLINE
				static int saturation_cnt=0;
				if(saturation_cnt < 10)
				{
					printf("entry:%ld, Ts:%lld,bd:%d, ch:%d,extras:%d,bdAggrateNo:%d, chagreNo:%d,saturation energy:%f\n",DPPPHAEvent_Data.DGTZ_EntryNo ,DPPPHAEvent_Data.TriggerTimeTag, DPPPHAEvent_Data.BdNo, DPPPHAEvent_Data.Ch, DPPPHAEvent_Data.Extras, DPPPHAEvent_Data.DGTZ_BdAggregateNo, DPPPHAEvent_Data.DGTZ_ChAggregateNo, DPPPHAEvent_Data.Energy);
					saturation_cnt++;
				}
#endif
				if(DPPPHAEvent_Data.Ch < DSSD_YH_CH_LOW && DPPPHAEvent_Data.SampleNo > 0)
				{
					if(Vec_DPP_Wave_Data[j].Probe1[E_SATURATION_CHECK_CH1] > 16000 ||  Vec_DPP_Wave_Data[j].Probe1[DPPPHAEvent_Data.SampleNo-500] > 16000)
						DPPPHAEvent_Data.Energy = 0x7FFF;
				//	else
				//		printf("entry:%ld, Ts:%lld,bd:%d, ch:%d,extras:%d,bdAggrateNo:%d, chagreNo:%d,saturation energy:%f\n",DPPPHAEvent_Data.DGTZ_EntryNo ,DPPPHAEvent_Data.TriggerTimeTag, DPPPHAEvent_Data.BdNo, DPPPHAEvent_Data.Ch, DPPPHAEvent_Data.Extras, DPPPHAEvent_Data.DGTZ_BdAggregateNo, DPPPHAEvent_Data.DGTZ_ChAggregateNo, DPPPHAEvent_Data.Energy);
				}
				else
					DPPPHAEvent_Data.Energy = 0x7FFF;
			}
			//
			else if( DPPPHAEvent_Data.Energy > E_SATURATION_CHECK && DPPPHAEvent_Data.SampleNo > 0 && DPPPHAEvent_Data.BdNo < 34)
			{
			//	if(Vec_DPP_Wave_Data[j].Probe1[E_SATURATION_CHECK_CH1] < E_SATURATION_CHECK_THRESHOLD && Vec_DPP_Wave_Data[j].Probe1[E_SATURATION_CHECK_CH2] < E_SATURATION_CHECK_THRESHOLD)
				if(ptr_wave_decode->CheckNoise(NSAMPLE_PRETRIGGER + 10) < 0)
				{
					DPPPHAEvent_Data.Energy = 0;
					continue;
				}
			}

			// 
			// corroct MWPC timestamp
			if(DPPPHAEvent_Data.Ch == MWPC0_CH || DPPPHAEvent_Data.Ch == MWPC1_CH )
			{
				if(DPPPHAEvent_Data.TriggerTimeTag > 500)
					DPPPHAEvent_Data.TriggerTimeTag -= MWPC_TS_CORR[DPPPHAEvent_Data.Ch - MWPC0_CH];
				else
					DPPPHAEvent_Data.TriggerTimeTag = 1;
			}
			//corroct veto ts
			else if(DPPPHAEvent_Data.Ch >= VETO0_CH && DPPPHAEvent_Data.Ch <= VETO2_CH )
			{
				if(DPPPHAEvent_Data.TriggerTimeTag > 500)
					DPPPHAEvent_Data.TriggerTimeTag -= VETO_TS_CORR[DPPPHAEvent_Data.Ch - VETO0_CH];
				else
					DPPPHAEvent_Data.TriggerTimeTag = 1;
			}
			// corroct SSD timestamp
			else if(DPPPHAEvent_Data.Ch >= SSD_CH_LOW &&  DPPPHAEvent_Data.Ch <= SSD_CH_HIGH )
			{
				if(DPPPHAEvent_Data.TriggerTimeTag > 500)
				{
					int bd_idx = (DPPPHAEvent_Data.Ch - SSD_CH_LOW)/8;
					DPPPHAEvent_Data.TriggerTimeTag -= SSD_TS_CORR[bd_idx];
				}
				else
					DPPPHAEvent_Data.TriggerTimeTag = 1;
			}
			//corr Ge timestamp
			else if(DPPPHAEvent_Data.Ch >= GE_LOW_CH && DPPPHAEvent_Data.Ch <= GE_HIGH_CH)
			{
				if(DPPPHAEvent_Data.TriggerTimeTag > 500)
					DPPPHAEvent_Data.TriggerTimeTag -= GE_TS_CORR[DPPPHAEvent_Data.Ch - GE_LOW_CH];
				else
					DPPPHAEvent_Data.TriggerTimeTag = 1;
			}
			//corr Beam on/off timestamp
			else if(DPPPHAEvent_Data.Ch == BEAM_PULSE1_CH)
				DPPPHAEvent_Data.TriggerTimeTag += TARGETOn_TS_CORR;
			else if(DPPPHAEvent_Data.Ch == BEAM_PULSE1_CH+1)
				DPPPHAEvent_Data.TriggerTimeTag += TARGETOff_TS_CORR;
			else if(DPPPHAEvent_Data.Ch == BEAM_PULSE2_CH-1 )
				DPPPHAEvent_Data.TriggerTimeTag += BEAMOn_TS_CORR;
			else if(DPPPHAEvent_Data.Ch == BEAM_PULSE2_CH)
				DPPPHAEvent_Data.TriggerTimeTag += BEAMOff_TS_CORR;

			//
			DPPPHAEvent_Data.Energy = Get_Calib_Energy(DPPPHAEvent_Data.Energy, DPPPHAEvent_Data.Ch, DPPPHAEvent_Data.PU);
			//
			//if(DPPPHAEvent_Data.Ch < SSD_BACK_CH_LOW && DPPPHAEvent_Data.Energy < 400 && DPPPHAEvent_Data.PU == 1)
			//	DPPPHAEvent_Data.Energy = DSSD_X_THRESHOLD;
			if(DPPPHAEvent_Data.Energy < DSSD_X_THRESHOLD && DPPPHAEvent_Data.PU != 1)
				continue;

#ifdef DECODE_WAVE_PHA
			if( (DPPPHAEvent_Data.Ch < DSSD_YH_CH_LOW && DPPPHAEvent_Data.Energy > DSSD_X_THRESHOLD ) || DPPPHAEvent_Data.PU ==1)
			{
				if(DPPPHAEvent_Data.SampleNo > 0)
				{
					//RCCR2
					if(DPPPHAEvent_Data.Ch < DSSD_Y_CH_LOW)
						ptr_wave_decode->SetFilterThreshold(FT_THRESHOLD);
					else if(DPPPHAEvent_Data.Ch < DSSD_YH_CH_LOW)
						ptr_wave_decode->SetFilterThreshold(FT_THRESHOLD/1.414);
					else
						ptr_wave_decode->SetFilterThreshold(FT_THRESHOLD/1.414/4);

					//RCCR2
					ret = ptr_wave_decode->Get_Trace_Trigger_RCCR2(NULL);
					if(ret < 2)
						Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
					else
					{
						//PHATs
						ptr_wave_decode->GetTriggerResult(&Fit_Result);
#ifdef DEBUG_MSG_OFFLINE
						static int pileup_count = 0;
						if(pileup_count < 10)
						{
							cout<<"filenum:"<<filenum<<",entry:"<<i<<", piluup num:"<<ret<<",ch:"<<DPPPHAEvent_Data.Ch<<",ts:"<<DPPPHAEvent_Data.TriggerTimeTag<<",E:"<<DPPPHAEvent_Data.Energy<<endl;;
							pileup_count++;
						}
#endif
						ret = ptr_wave_decode->GetPHAResult(&Fit_Result, NULL);
						if(ret > 1)
						{
							ULong64_t ts_tmp = DPPPHAEvent_Data.TriggerTimeTag;
							int ts_unit = 10;
							if(DPPPHAEvent_Data.DGTZ_Type == 1) //if board is  V1730
								ts_unit = 2;
							int short_pileup_cnt=0;
							for(int peak_idx = 0; peak_idx < ret; ++peak_idx)
							{
								if(peak_idx > 0 && (Fit_Result.Ts[peak_idx] - Fit_Result.Ts[peak_idx-1])*ts_unit < COIN_WINDOW)
								{
									short_pileup_cnt++;
									DPPPHAEvent_Data.PU = peak_idx + 2 +10*short_pileup_cnt;
									DPPPHAEvent_Data.TriggerTimeTag = ts_tmp + Fit_Result.Ts[peak_idx]*ts_unit - NSAMPLE_PRETRIGGER*ts_unit + short_pileup_cnt*COIN_WINDOW; // ???
								}
								else
								{
									DPPPHAEvent_Data.PU = peak_idx + 2;
									short_pileup_cnt = 0;
									DPPPHAEvent_Data.TriggerTimeTag = ts_tmp + Fit_Result.Ts[peak_idx]*ts_unit - NSAMPLE_PRETRIGGER*ts_unit; // ???
								}
								DPPPHAEvent_Data.Energy = Get_Calib_Energy_PHA(Fit_Result.E[peak_idx], DPPPHAEvent_Data.Ch, DPPPHAEvent_Data.PU);
								Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
							}
						}
						else
							Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
					} // end pileup
				}
				else
					Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
			} // end check ch 
			else
			{
				Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
			}
#else
			Fill_DPP_Data_Into_Shm(&DPPPHAEvent_Data);
#endif //end DECODE_WAVE_PHA
#endif // end FILE_TIME_SORT
		} // end loop vector
	}   // Entries cycle end

#ifdef FILE_TIME_SORT
	//
	DPPPHAEvent_Data.Ch=0xFFFF;
	while(fp_shm->BlockFillShm((char*)&DPPPHAEvent_Data,sizeof(DPP_PHA_Event_t)) <= 0)
		usleep(1000);
	thread_map->Join();
	//while(thread_map->GetState() != TThread::kCanceledState)
	//	usleep(1000);
	delete thread_map;
#endif
	Trigger->Delete();
	FileOut->cd();
	FileOut->Write();
	for(int b=0; b<MAXNB; b++) {
		DGTZ[b]->Delete();
	}
	FileOut->Close();
	FileIn->Close();
	delete FileIn;
	delete FileOut;

	return 0;
}

void Fill_DPP_Data_Into_Shm(DPP_PHA_Event_t* ptr_data)
{
	//if( ptr_data->Energy > 0)
	if(ptr_data->PU ==1 || (ptr_data->Ch < TOTAL_SI_CH && ptr_data->Energy > DSSD_X_THRESHOLD) || (ptr_data->Extras&1024) > 0 || (ptr_data->Ch >= TOTAL_SI_CH && ptr_data->Energy > 0))
		while(fp_shm->BlockFillShm((char*)ptr_data, sizeof(DPP_PHA_Event_t)) <= 0)
			usleep(1000);
}


Int_t Read_Par_From_File(char* filename, double (*cal_par)[2])
{
	//cal par
	Double_t par[2];
	char str_read[256];
	int ch;

	for(int ii = 0;ii < TOTAL_CH;++ii)
	{
		cal_par[ii][0] = 0.0;
		cal_par[ii][1] = 1.0;
	}

	FILE* fp=fopen(filename, "r");
	if(fp == NULL)
	{
		printf("==============================================\n");
		printf("open calibration file %s fail,  plz check!\n", filename);
		printf("==============================================\n");
		return -1;
	}
	while(fgets(str_read, 256, fp) != NULL)
	{
		sscanf(str_read, "%d\t%lf\t%lf\t", &ch, &par[0], &par[1]);
		if(ch < TOTAL_CH)
		{
			cal_par[ch][0] = par[0];
			cal_par[ch][1] = par[1];
		}
#ifdef DEBUG_MSG_OFFLINE
		if( ch < 3)
			printf("read file %s line:%s, decode cal par: %d %lf %lf\n", filename, str_read, ch,  cal_par[ch][0], cal_par[ch][1]);
#endif
	}//end the while loop
	fclose(fp);
	return 0;
}


Int_t Read_Cal_Par()
{
	Read_Par_From_File(CALIBRATION_FILENAME, Cal_Par);
//	Read_Par_From_File(SSD_CALIBRATION_FILENAME, Cal_Par_SSD);
#ifdef DECODE_WAVE_PHA
	Read_Par_From_File(PHA_CALIBRATION_FILENAME, Cal_Par_PHA);
#endif
	return 0;
}
```
- 宏定义 (Macro Definition)
	- C/C++ 中以 `#` 开头的指令，比如 `#define`, `#ifdef`。在编译程序之前，告诉编译器保留哪些代码，删除哪些代码。
    - `#ifdef DECODE_WAVE_PHA`：这句意思是“如果我定义了 `DECODE_WAVE_PHA` 这个开关，那么就编译下面这段波形分析的代码；否则直接跳过”。
    - 只想快速看能谱（关掉宏），想详细分析波形堆积（打开宏）。
```
inline Double_t Get_Calib_Energy(UInt_t ADC_values,  Int_t ch, UShort_t PU)
{
	if(ch < TOTAL_SI_CH && ADC_values == 0x7FFF)
	{
		if (ch < DSSD_YH_CH_LOW)
			return E_SATURATION_DSSDY;
		else if (ch < SSD_BACK_CH_LOW)
			return E_SATURATION_DSSDYH;
	}
	/// DSSD inter source corr
	Double_t ret = 0.0;
	if(ch < SSD_CH_LOW)
		ret = 1.0*IntSour_Corr_k*Cal_Par[ch][0]+IntSour_Corr_b+1.0*IntSour_Corr_k*Cal_Par[ch][1]*ADC_values;
	//SSD
	else if( ch < SSD_BACK_CH_LOW)
	{
		ret=1.0*Cal_Par[ch][0] + 1.0*Cal_Par[ch][1]*ADC_values;
	}
	else
		ret = 1.0*Cal_Par[ch][0]+1.0*Cal_Par[ch][1]*ADC_values;
	
	if(ret < 0)
		ret = 0;
	return ret;
}

inline Double_t Get_Calib_Energy_FIT(double ADC_values,  Int_t ch, UShort_t PU)
{
	if(ch < TOTAL_SI_CH && ADC_values == 0x7FFF)
	{
		if (ch < DSSD_YH_CH_LOW)
			return E_SATURATION_DSSDY;
		else if (ch < SSD_BACK_CH_LOW)
			return E_SATURATION_DSSDYH;
	}

	Double_t ret = 0.0;
	/// DSSD inter source corr
	if(ch < SSD_CH_LOW)
		 ret = 1.0*IntSour_Corr_k*Cal_Par_FIT[ch][0]+IntSour_Corr_b+1.0*IntSour_Corr_k*Cal_Par_FIT[ch][1]*ADC_values;
	//SSD
	else if( ch < SSD_BACK_CH_LOW)
	{
		// add SSD corr par, wait for update
		ret = 1.0*Cal_Par_FIT[ch][0]+1.0*Cal_Par_FIT[ch][1]*ADC_values;
	}
	else
		ret =  1.0*Cal_Par_FIT[ch][0]+1.0*Cal_Par_FIT[ch][1]*ADC_values;
	if(ret < 0)
		ret = 0;
	return ret ;
}

inline Double_t Get_Calib_Energy_PHA(double ADC_values,  Int_t ch, UShort_t PU)
{
	if(ADC_values == 0)
		return 0;
	if(ch < TOTAL_SI_CH && ADC_values == 0x7FFF)
	{
		if (ch < DSSD_YH_CH_LOW)
			return E_SATURATION_DSSDY;
		else if (ch < SSD_BACK_CH_LOW)
			return E_SATURATION_DSSDYH;
	}
	/// DSSD inter source corr
	Double_t ret = 0.0;
	if(ch < SSD_CH_LOW)
		ret =  1.0*IntSour_Corr_k*Cal_Par_PHA[ch][0]+IntSour_Corr_b+1.0*IntSour_Corr_k*Cal_Par_PHA[ch][1]*ADC_values;
	//SSD
	else if( ch < SSD_BACK_CH_LOW)
	{
		// add SSD corr par, wait for update
		ret = 1.0*Cal_Par_PHA[ch][0]+1.0*Cal_Par_PHA[ch][1]*ADC_values;
	}
	else
		ret = 1.0*Cal_Par_PHA[ch][0]+1.0*Cal_Par_PHA[ch][1]*ADC_values;
	if(ret < 0)
		ret =0;
	return ret;
}
```
- 流程
	1. **探测器**产生一个微弱电压脉冲。
	2. **CAEN 数字化仪 (Board)** 捕捉这个脉冲。
	3. **硬件 DPP** 内部用 **梯形滤波** 算出脉冲高度（能量），用 **CFD/CR-RC** 算出时间（时间戳）。
	4. 这个程序 (**SortEvent**) 读取这些数字。
	5. 它根据 **板卡/通道** 知道是哪个探测器响了。
	6. 它修正 **时间戳** 的延迟。
	7. 如果定义了 **宏 (DECODE_WAVE_PHA)**，它会再用软件里的 **CR-RC 算法** 检查一下是不是有两个离子同时打进来了（堆积）。
	8. 最后存入 ROOT 文件供你分析。