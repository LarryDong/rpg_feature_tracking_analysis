# 事件相机特征跟踪分析工具

forked from **uzh-rpg/rpg_feature_tracking_analysis** [https://github.com/uzh-rpg/rpg_feature_tracking_analysis]。

**增加了中文说明，并在代码中加入注释，便于理解与使用。**



# 0、引用

如果用于学术文章，请引用如下论文。（尊重作者成果，这段儿我单独拎出来）

```
@Article{Gehrig19ijcv,
  author = {Daniel Gehrig and Henri Rebecq and Guillermo Gallego and Davide Scaramuzza},
  title = {EKLT: Asynchronous Photometric Feature Tracking using Events and Frames},
  booktitle = {Int. J. Comput. Vis. (IJCV)},
  year = {2019}
}
```



# 1、安装

```bash
git clone feature_tracking_anlaysis
cd feature_tracking_analysis
virtualenv venv					    # 与下一条可选
source venv/bin/activate		# 遇上一条可选
pip install -r requirements.txt
```

注：除此之外，需要安装latex相关依赖，否则会在运行时报错
> OSError: [Errno 2] No such file or directory: 'latex'
解决办法：
```bash
sudo apt install texlive-latex-base texlive-latex-extra texlive-fonts-recommended dvipng 
pip install tabulate
```


# 2、 运行示例

### 2.1 计算真值与误差

无论是绘制单一误差曲线，还是进行比较，都需要首先计算真值与误差。运行 **evaluate_tracks.py** 文件。基础指令如下：

```bash
python evaluate_tracks.py --tracker_type KLT --file rel/path/to/tracks.txt --dataset rel/path/to/dataset.yaml --root path/where/to/find.bag
```

运行 evaluate_tracks.py 文件，需要提供参数至少需要4个：

`tracker_type`：选择使用KLT跟踪，或者重投影方式（一般不用），参数值只能是：KLT 或 reprojection

`file`：需要评估的特征跟踪文件，文件名一定要为tracks.txt（具体格式见 **3.1 数据准备**）

`dataset`：配置文件，以 .yaml 或 yml 结尾的配置文件（配置方式见 **3.2 数据集配置文件**）

`root`：数据集文件，评估时根据这个数据集文件(.bag)进行计算KLT跟踪

以及部分可选参数：

`error_threshold`：计算误差时，超过阈值认为丢失。默认为10 pixels

`tracker_params`：跟踪参数，没有制定时默认采用 config 下的文件，制定了 KLT 的窗口大小、金字塔层数

## 2.2 绘制单一误差图

在2.1的基础上，运行时加上`--plot_3d` 或 `--plot_errors` 等参数，可以绘制好看的 pdf 图像。

使用`--plot_errors`这个参数时，需要安装latex相关依赖，eth没有给出，经过我测试，需要安装的内容写在了 **1、安装** 当中

另外还有可选参数 `video_preview`，生成带有 gt 的视频。

运行结果将在 `file` 文件所在文件中，创建 results 文件，进行保存。

## 2.3 方法比较

如果需要比较两种方法，需要利用 **compare_tracks.py** 这个文件，具体指令形式为：

```bash
python compare_tracks.py --error_threshold 10 --root /path/to/tracks/dir --config /path/to/compare_config.yaml --results_directory /path/to/save/results
```

需要提供的参数有：

`--root`：根文件，包含有track数据等文件的目录

`--config`：比较方法时的配置文件，具体格式件见 **3.4**

`--results_directory`：运行结果保存文件，包括误差、特征平均时长、曲线图等

**注：在运行时需要首先生成每种方法对应的真值与误差数据，然后才能运行，否则会提示找不到gt文件**

个人采用的文件目录：

```bash
root
	-dataset.yaml		# 计算gt时需要用到，记录dataset的相关信息，对应 evaluate_tracks 的--dataset 参数
	-config.yaml		# 进行比较时用到，记录方法比较的文件目录，对应 compare_tracks.py 的--config 参数
	/method1_dir		# 需要比较方法1的目录
		/dataset_1		# 第一个数据集运行结果目录
			-tracks.txt	# 运行结果
		/dataset_2		# 第二个数据集运行结果目录
			-tracks.txt
		...
	/method2_dir
	...
	/results			# 结果保存目录
```

运行指令：

```bash
python evaluate_tracks.py --tracker_type KLT --file root/method1_dir/dataset_1/tracks.txt --dataset root/dataset.yaml --root path/to/rosbag.bag		# 根据method1跟踪结果生成error和gt
python evaluate_tracks.py --tracker_type KLT --file root/method1_dir/dataset_2/tracks.txt --dataset root/dataset.yaml --root path/to/rosbag.bag		# 根据method2跟踪结果生成error和gt
python compare_tracks.py --error_threshold 10 --root root --config root/config.yaml --results_directory root/results			# 进行比较
```

**注意：dataset.yaml 中的数据集名称与实际给的要统一**

 



# 3、 文件配置

## 3.1 数据准备

评估的特征跟踪数据，需要按照如下形式进行存储：

```
# feature_id timestamp x y
25 1.403636580013555527e+09 125.827 15.615 
13 1.403636580015467890e+09 20.453 90.142 
...
```

其中每行依次为：特征id，时间戳，x、y坐标。

这个文件对应运行时的 `file` 参数

## 3.2 数据集配置文件

配置文件指 xxx.yaml，需要指定：

1. 数据集路径（很奇怪配置文件需要数据集路径，且 `root` 参数也需要路径。保证统一吧）
2. rosbag数据集中图像的topic名称
3. 其他参数（基于重投影方法的跟踪，一般用不到）

示例如下：

```yaml
type: bag
name: relative/path/to/ros.bag  # relative path of bag

# For KLT based tracking 
image_topic: /dvs/image_raw  

# For reprojection based tracking
depth_map_topic: /dvs/depthmap
pose_topic: /dvs/pose
camera_info_topic: /dvs/camera_info
```

## 3.3 KLT跟踪配置文件

`--tracker_params`参数后面可选用于跟踪时的配置文件，若不指定，则默认使用 config 路径下对应的配置文件。跟踪配置文件指定KLT的窗口大小与最大金字塔层数。格式参考 config/KLT_params.yaml。



## 3.4 compare的配置文件

用于比较多种方法时的配置文件，以.yml/yaml结尾。每种方法3行，分别是：方法名称、颜色RGB、数据路径（路径下需包含原始数据，以及evaluate时生成的gt和error文件）。注意2、3行开头是4个空格。

```
- Our_method:
    - [0, 255, 255]
    - tracks/our_method/
- Other_method:
    - [0, 255, 255]
    - tracks/other_method/
...
```

# 4、代码分析

## 4.1 代码流程

**1. 特征数据处理**

1. 读取数据文件的所有特征跟踪的数据
2. 确定第一个特征的时间戳，作为初始时刻
3. 统计初始时刻的所有特征id，记为有效id
4. 将所有有效的特征，根据id号，创建多个dict存储特征

**2. 根据图像进行初始化KLT**

1. 根据上一步获得的初始时刻，在rosbag中寻找时间戳最接近的图像frame
2. 由于时间戳可能不同步，对于每个feature，寻找初始图像frame前后两个的位置，进行线性插值获得在图像中的位置
3. 完成图像中的初始创建

**3. 跟踪**

1. rosbag数据的图像中，利用opencv的`calcOpticalFlowPyrLK` 函数进行特征跟踪。

**4. 计算error**

**5. 绘图等其他可选操作**



## 4.2 代码局限

代码中有一行提示：

```
WARNING: This package only supports evaluation of tracks which have been initialized at the same  time. All tracks except the first have been discarded.
```

这个当完成 **“特征数据处理(4.1.1)** 后，如果发现有效id的特征数量少于了总的行数，这意味着在初始化时刻的时间戳，并没有包含所有的特征id。而这个代码对后续新增的特征id不会进行跟踪。这也提示我们，在自己准备 data 数据时，一定要保证初始时刻的所有特征，要具有相同的时间戳！ 



# 5. 其它

若使用此工具，用于学术时，注意引用作者论文！

若您的某些工作，参考了我这forked后给出的注释与解释，希望可以注明出处；

关于事件相机的特征提取与跟踪，欢迎大家前来交流！我目前正在进行相关研究。


