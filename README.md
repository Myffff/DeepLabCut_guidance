# 如何使用DeepLabCut
## 1. DeepLabCut 安装
### 1.1 下载安装文件
进入Github安装地址：

https://github.com/DeepLabCut/DeepLabCut/blob/master/docs/installation.md

选择 step2 中的下载链接，下载DEEPLABCUT.yaml 文件。

### 1.2 创建环境并安装
#### 1.2.1 确保电脑上有 Anaconda，如果没有要先下载 Anaconda
#### 1.2.2 打开Anaconda Navigator
从任务栏搜索即可.依次点击 Environment--> Import-->选择DEEPLABCUT.yaml文件的下载目录。
如果需要更改下载的地址，先把DEEPLABCUT.yaml文件移到自己想要下载的文件目录下。然后import就可以了，如果想要更改new environment name 也可以。

## 2. 模型训练
### 2.1 训练前准备
+ 选择安装好的DEEPLABCUT环境，在播放键处选择Openwith Ipython
在命令行分两次输入：
- `import deeplabcut`
- `deeplabcut.launch_dlc()`

然后就会出现deeplabcut的主页面，我们在左上方选择manage project页面。
+ 新的文件夹将以  project name + experimenter name 命名。
+ 选中 Copy the videos.如果电脑中有代码编辑器（如VScode）的话可以直接点击edit config file，进入 config.yaml。如果没有的话，先点OK，然后到文件资源管理器中使用记事本打开config.yaml。

#### 抽取关键帧
由于输入的是连续的视频，此处需要定义抽取关键帧的方法，用于后续的标注。 这里可以选择手动或自动提取（一般自动就可以 手动需要一帧帧挑），并可以选择自动抽帧的方法（kmeans和uniform）。
设置完成后点击OK ，需要一点时间来读入视频和提取关键帧。

在label data页面中，选择label frames等待打开GUI界面，开始正式标记.选择Load frames，在弹出的任务管理器界面中选择：以  project name + experimenter name 命名的文件-->labeled-data-->下一级文件夹
然后就可以开始标注了，如果前面导入了多段视频的话，每个视频会生成一个文件夹，需要对每个文件夹分别标注。
鼠标右键点击标注，页面右侧选择器会自动下滚。

在标注完成后，回到主页面，并选择Check Labels，就可以看到在任务文件夹下的labled-data文件中找到一个带有-labeled的文件，点进去后可以看到标注的点和骨架已经被绘制在了图中。（没有看到也没关系，不影响下一步进行）

然后就可以点击Creaete Training Dataset进入到创建训练集的部分（如果用服务器训练的可以直接跳过这部分）

此处可以选择特征提取网络，主要影响后续的运算速度和精度，配置好的可以选择resnet家族的，配置一般的可以选择mobilenet.
全部选完后点OK。
在Ipython命令行中可以看见创建成功的提示。

### 2.2 开始训练
我租了服务器练，如果硬件条件好（指GPU）就在本机上练。
如果选择租用服务器，需要选择预装CUDA tensorflow cuDNN的镜像。
+ 点击打开jupyterLab。
+ 需要下载deeplabcut，并向网盘中导入先前project name + experimenter name 命名的文件夹。
+ 新建.ipynb文件用以书写代码
+ 如果使用自己的GPU也需要安装相应的python包，但版本不宜过高。（可以到网上寻找python包的安装教程）

#### 代码书写
##### 配置环境
`ProjectFolderName = 'mouse-adm1-2022-08-22'`
`VideoType = 'avi' `
`videofile_path = ['/mnt/mouse-adm1-2022-08-22/videos/behavCam3.avi'] #Enter the list of videos or folder to analyze.根据自己情况更改`
`videofile_path`


`import os`
`os.environ["DLClight"]="True"`
`import deeplabcut #设置config.yaml的位置,根据自己情况更改`
`path_config_file = '/mnt/mouse-adm1-2022-08-22/config.yaml'`

##### 载入标签数据
`deeplabcut.load_demo_data(path_config_file)`
`#Perhaps plot the labels to see how the frames were annotated:`
`deeplabcut.check_labels(path_config_file)`

##### 训练特征检测器
在这个部分中，训练是不会主动停止的，需要人为停止：在训练到自己期望的次数后，点击此代码块，输入II（imput的首字母）回车。此后该代码块会报错，但不必在意，直接进行下一个代码块即可。

`deeplabcut.train_network(path_config_file, shuffle=1, saveiters=300, displayiters=10)`
`#notice the variables "saveiters" and "dsiplayiters" that can be set in the function,you just need to run this until you get at least 1 snapshot, which is set by: "save_iters" ,(so in this case you could stop after 500!) How do I stop? Click the STOP button!To train until ~2,000 iterations on a CPU should be ~30 min`

##### 评价训练好的特征网络
`deeplabcut.evaluate_network(path_config_file,plotting=True)`

##### 分析视频
`deeplabcut.analyze_videos(path_config_file,videofile_path, videotype=VideoType)`

##### 创建带有标注的视频
`deeplabcut.create_labeled_video(path_config_file, videofile_path, draw_skeleton=True)`
`#在视频上绘制关键点`
`deeplabcut.plot_trajectories(path_config_file,videofile_path, videotype=VideoType)`

在云的videos文件夹中找到该mp4文件，右键download后可以观看.

### 其他操作
都可以在官网链接：[DeepLabCut/standardDeepLabCut_UserGuide.md at master · DeepLabCut/DeepLabCut · GitHub](https://github.com/DeepLabCut/DeepLabCut/blob/master/docs/standardDeepLabCut_UserGuide.md) 中获取