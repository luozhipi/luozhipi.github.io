---
layout: page
title: Resume
permalink: /cv_en/
---
# Zhiping Luo| Curriculum Vitae #

Male • 1986 • Single

English • Chinese

qq: 277542747 • Wechat: lzp138252 

[luozhipi@gmail.com](mailto:luozhipi@gmail.com) • [luozhipi.github.io](http://luozhipi.github.io) • [github.com/luozhipi](https://github.com/luozhipi)

C++ • C# • JAVA • Python • OpenCV • OpenGL • GLSL • Maya • JavaScript • CGAL • Matlab • Adobe after effects

Computer vision: object recognition and tracking • image search

Computer graphics: 3D character animation • shape modeling

## Education ##

----------

`Utrecht University`         										Netherlands

PhD in Computer Graphics,  						2012.02–2016.01 (supported by the [COMMIT] project)

Dissertation: Interactive Character Deformation Using Simplified Elastic Models [[preprint]]


`South China University of Technology`   Guangzhou, China

Master in Image Processing and Pattern Recognition,  						2007.09–2010.07

Thesis: Towards Landmark Recognition Via Large Scale Social Media Mining


`Nanchang University`           										Nanchang, China

Bachelor in Embedded System,  						2003.09–2007.07


## Publication ##

----------
Z. Luo, R.C. Veltkamp, A. Egges.: `Volumetric Space Deformations Based on Voxels`. submitted to Journal of Visual Computer.

Z. Luo, R.C. Veltkamp, A. Egges.: `Fast Multi-Domain Smooth Embedding for Character Deformation`. revision submitted to Journal of Computer Animation and Virtual Worlds.

Z. Luo, R.C. Veltkamp, A. Egges.: `As-Rigid-As-Possible Character Deformation Using Point Handles`. 11th International Symposium on Visual Computing 2015 (ISVC'15), oral.

Z. Luo, R.C. Veltkamp, A. Egges.: `Flexible Point Handles Metaphor for Character Deformation`. ACM Symposium on Virtual Reality Software and Technology (VRST) 2015, poster.

Z. Luo, R.C. Veltkamp, A. Egges.: `Volumetric Surface Deformation with Auxiliary Voxel Grids`. The 32nd Annual Conference on Computer Graphics International (CGI) 2015, short paper. [VoxelARAP code]

Z. Luo, R.C. Veltkamp, A. Egges.: `Space Deformation for Character Deformation using Multi-Domain Smooth Embedding`. The 28th Annual Conference on Computer Animation and Social Agents (CASA 2015), short paper.

Z. Luo, N. Pronost, A. Egges.: `Physics-based Human Neck Simulation`. Virtual Reality Interactions and Physical Simulations 2013: 10p. [FEMNeck code]

Z. Luo, H. Li, J. Tang, R. Hong, TS. Chua.: `ViewFocus: explore places of interests on Google maps using photos with view direction filtering`. Proceedings of the 17th ACM international conference on Multimedia 2009: 963-964.

Z. Luo, H. Li, J. Tang, R. Hong, TS. Chua.: `Estimating poses of world's photos with geographic metadata`. Advances in Multimedia Modeling 2010, 695-700.

Z. Luo, X. Zhang.: `A Semi-Supervised Learning Based Relevance Feedback Algorithm in Content-Based Image Retrieval`. Pattern Recognition, 2008. CCPR '08. Chinese Conference on , vol., no., pp.1,4, 22-24 Oct. 2008.

J. Lu, VA. Nguyen, Z. Niu, B. Singh, Z. Luo, MN. Do.: `CuteChat: a lightweight tele-immersive video chat system`. ACM Multimedia 2011: 1309-1312. [CuteChat]

TS. Chua, J. Tang, R. Hong, H. Li, Z. Luo, Y. Zheng.: `NUS-WIDE: a real-world web image database from National University of Singapore`. Proceedings of the ACM International Conference on Image and Video Retrieval 2009, 48. [NUS-WIDE]

R. Hong, J. Tang, ZJ. Zha, Z. Luo, TS. Chua.: `Mediapedia: Mining web knowledge to construct multimedia encyclopedia`. Advances in Multimedia Modeling 2010, 556-566.

Y Wu, M Wang, G Li, Z Luo, TS Chua, X Liu.: `VDictionary: automatically generate visual dictionary via wikimedias`. Advances in Multimedia Modeling, 796-798.

RH. Li; Z. Luo; G. Han.: `Pseudo-inverse Locality Preserving Projections`. Computational Intelligence and Security. 2009. CIS '09. International Conference on , vol.1, no., pp.363,367, 11-14 Dec. 2009.

## Presentation ##

----------
`As-Rigid-As-Possible Character Deformation Using Point Handles`, 11th International Symposium on Visual Computing 2015 (ISVC'15), Las Vegas, Nevada, USA, December 14, 2015.

`Flexible Point Handles Metaphor for Character Deformation`, ACM Symposium on Virtual Reality Software and Technology (VRST) 2015, Beijing, China, November 13th, 2015.

`Volumetric Surface Deformation with Auxiliary Voxel Grids`, Computer Graphics International 2015, Strasbourg, France, June 25, 2015.

`Space Deformation for Character Deformation using Multi-Domain Smooth Embedding`, 28th Annual Conference on Computer Animation and Social Agents, Singapore, May 13, 2015.

`Physics-based Human Neck Simulation`, 10th Workshop on Virtual Reality Interaction and Physical Simulation, INRIA Lille,  France, November 28, 2013.

`Real-time Human Neck Animation Using Simulated Blendshapes`, GRIS Kolloquium, TU Darmstadt, Germany, 10-Oct-2013.

`ViewFocus: explore places of interests on Google maps using photos with view direction filtering`, ACM Multimedia Conference 2009, Beijing, Oct, 2009.

## Research Experience ##

----------

###Ph.D student researcher, Utrecht University, Netherlands###

2012-02 ~ 2016-01

3D character deformation           **skinning, deformable body**

- `Facial animation`:  This project includes text-to-speech conversion based on the Microsoft Speech SDK, and speech-driven facial expression synthesis based on the MPEG-4 standard. A phonogram is mapped to a specific expression (a morph target), those of which are blended with weights that related to the tone. The front-end is implemented in Python, and the engine is based on Ogre and C++.
- `Human neck modeling`: 通过物理方法模拟人体的骨骼肌肉系统和皮肤变形来达到高保真的虚拟人变形动画. 方法包括使用maya处理几何曲面 (polygonal mesh) 如缺口 (hole), 从几何曲面自动生成体积曲面 (volumetric mesh), 基于有限元的肌肉建模和模拟, 基于多刚体系统动力学 (multi-body dynamics) 的骨骼运动模拟, 基于弹簧模型 (springs) 的骨骼-肌肉耦合机制, 以及基于质点-弹簧 (mass-spring) 模型的皮肤建模. 整个系统动力学的模拟用隐式时间积分法 (implicit time integration) 求解从牛顿第二定律推导出的常微分方程 (ODE).
- `基于弹性能量最小化变形`: 首先构建几何曲面的保刚性弹性能量 (as-rigid-as-possible elastic energy), 帧与帧之间单元的旋转矩阵由奇异值分解法SVD求解, 然后最小化其目标方程使得曲面的每个单元的变换为刚性变换 (rigid transformation). 因此，变形中曲面局部细节将最大可能不失真. 方法有助于高质量渲染有纹理渲染的曲面, 以及生成高质量的弹性变形动画. 通过体素化(voxelization)和修改目标方程所要求的权重方程, 将曲面弹性能量扩展到体积曲面以进一步最小化曲面物体的体积变化 (volume preservation).
- `空间变形 (space deformation)`: 非线性的弹性能量最小化以及基于有限元的软体动画 (soft body animation) 或可变形体 (deformable body) 的模拟一般而言其计算复杂度很高, 运行速度慢, 无法满足实时要求. 空间变形技术是一个有效的加速策略: 最小化求解或模拟在一个分辨率更低 (coarser) 的曲面上进行, 而后曲面变形定义为一个插值函数, 其输入为分辨率低曲面上的点位移. 实现了基于径向基函数 (radial basis functions) 的平滑插值以及常用的三线性插值 (trilinear interpolation) 和顶点混合技术 (vertex blending). 为满足角色变形动画 (character deformation) 的要求, 提出域分割法 (domain decomposition), 每个域构建一个RBF插值系统, 域与域之间的缝合缺陷 (seam artifacts) 通过Laplacian平滑来消除.
- `机器学习子空间变形模型 (learning linear blend skinning model)`: 当前3D扫描设备或运动捕捉设备在电影以及动画中得到了广泛的应用. 它们用来捕捉变形动画, 并生成融合变形 (blendshape) 数据库. 但因当前主流引擎仍只支持基于骨骼的动画模型, 或因blendshapes需很大存储空间, 或不方便艺术家来操作角色 (character rigging) 因操作骨骼更直观, 因而需把blendshapes转换成角色蒙皮最常用的模型Linear blend skinning (skeletal subspace deformation). 以blendshapes为学习样本,构建一个LBS模型与样本之间顶点位置最小化的最小二乘法方程 (least squares), 来求解最优的LBS骨骼动画以及蒙皮权重.
- `软体动画`: 基于Bullet引擎的连续碰撞检测 (continuous collision detection) 方法实现软体动画soft body animation. 其中变形用重心插值 (barycentric interpolation) 法实现.

###Research associate, gameLAB of Nanyang Technological University, Singapore###

2011-09 ~ 2012-01

计算机视觉 					  **运动捕捉**

- `基于标志物 (marker-based) 的人体运动捕捉`: 基于OpenCV的模板匹配跟踪腿部运动轨迹，并实验了不同的匹配方程. 捕捉的运动轨迹用OpenGL和vtk来可视化, 方便分析标志物在三维空间的位置以及他们之间的空间关系. 此类信息提供给新加坡运动员训练中心用以帮助提高运动效果.

###Software Engineer, Advanced Digital Sciences Center of University of Illinois Urbana-Champaign, Singapore###

2010-10 ~ 2011-09

计算机视觉 					  **目标跟踪与分割**

- `摄像机标定`: 基于OpenCV相机标定模块标定深度相机swissranger 4000 与普通彩色相机的内外部参数. 首先使用cvCalibrateCamera2(…)分别标定各个相机的内部参数 (拍摄的棋盘chessboard图像缩放到同一大小). 因深度相机直接提供了点云, 因此二维图像的特征点 (by corner detection) 与点云之间的变换矩阵也就容易通过cvFindExtrinsicCameraParams2(…)计算得到.
- `沉浸式会议聊天系统`: 作为团队成员, 共同开发出实时算法可以从视频中实时分割出人物对象. 独立负责: 通过OpenMP实时化系统, 使得系统可运行于带摄像头的普通笔记本; 根据关键字从互联网中多线程下载图片, 用以替换背景, 背景也可换成ppt, 视频等多媒体内容.
- `手势识别`: 基于kinect提供的深度信息，识别手势. 与聊天系统结合, 达到通过手势替换背景的人机交互.

###Research Intern, Lab for Media Search, National University of Singapore, Singapore###

2008-09 ~ 2009-12

**互联网图像检索**

- `视角过滤景点浏览`: 首先根据关键字和地理标签 (geo-tag) 从网络中检索出给定景点图片, 而后提取出图像的SIFT特征, 通过bundle adjustment匹配SIFT点云 (使用了华盛顿大学的Bundler) 估计相机姿态. 设定地图的三维坐标系统, 简单估计相机的拍摄角度. 用户选择景点的一个兴趣点, 系统将返回拍摄角度指向该点的图片.
- ` 图像标注`: 基于C#语言实现一个图像标注系统, 标注结果保存到mysql.
- `图像检索数据库构建`: 从flickr下载海量图片, 提取图像特征, 用支持向量机和k-NN方法训练图像对象识别分类器.

###Research Fellow, Singapore Management University, Singapore###
 
2009-12 ~ 2010-03

**社交网络挖掘**

- `产品垃圾评论自动检测模型`: 用JAVA和Mysql实现Amonzon产品垃圾评论检测模型. 主要基于向量空间模型 (tf-idf) 来判断评论的相似度(依据向量点积)作为一个启发式规则. 系统还提供用户标注功能, 从而得到学习样本, 为开发机器学习模型奠定基础.

###Software Developer, FairEX International Financial System, Singapore###
 
2010-04 ~ 2010-10

**外汇交易系统**

- 开发基于adobe flex的外汇交易系统前端, 以及基于JAVA和weborb的实时外汇行情数据流引擎. 引擎以web service形式接入整个系统, 从而适应了由不同开发语言开发的现有版本.

## Certifications ##

----------
2006~ Software Architect awarded by Ministry of Industry and Information Technology of
China

2008～ System Analyst awarded by Ministry of Industry and Information Technology of
China

## Teaching Assistant ##

----------
2014-09~2014-11 Game Programming, Universiteit Utrecht

2015-04~2015-07 Three-dimensional Modeling, Universiteit Utrecht

[VoxelARAP code]: https://github.com/luozhipi/VoxelARAP
[CuteChat]: https://sites.google.com/site/adscitem/recent-contributions-2
[NUS-WIDE]: http://lms.comp.nus.edu.sg/research/NUS-WIDE.htm
[FEMNeck code]: https://github.com/luozhipi/FEM-Neck
[preprint]:  /papers/theis_v2_low.pdf
[COMMIT]: http://www.commit-nl.nl/

