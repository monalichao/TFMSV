# 3.MySQL数据库

## 3.1 配置MySQL

- 使用 root 登录 MySQL

```
$ mysql -u root -p
```

- 为 M-SENA 创建数据库

```
mysql> CREATE DATABASE TFMSV;
```

1. **创建用户**

```
CREATE USER TFMSV IDENTIFIED BY '120455713p@P';
```

- 作用：在 MySQL 中创建一个用户名为 `TFMSV` 的新用户，登录密码为 `120455713p@P`。
- 之后可以用这个用户连接数据库，而不必用 `root`。

1. **授权访问数据库**

```
GRANT ALL PRIVILEGES ON TFMSV.* TO TFMSV@`%`;
```

- 作用：授予 `TFMSV` 用户对 **数据库 TFMSV 下的所有表** (`TFMSV.*`) 的**全部权限**（CREATE、SELECT、INSERT、UPDATE 等）。
- `TFMSV@`%`` 中的 `%` 表示允许该用户从**任何主机**连接 MySQL。如果你只想在本机连接，可以改成 `TFMSV@localhost`。

1. **刷新权限**

```
FLUSH PRIVILEGES;
```

- 作用：让 MySQL 立即应用刚才的用户创建和授权操作。MySQL 会重新加载权限表。

或者直接在 Windows 命令行执行一次性创建

在 **PowerShell 或 CMD** 中运行：

```
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS TFMSV; CREATE USER 'TFMSV'@'%' IDENTIFIED BY '120455713p@P'; GRANT ALL PRIVILEGES ON TFMSV.* TO 'TFMSV'@'%'; FLUSH PRIVILEGES;"
```

- 这会直接创建数据库、用户并授权。

------

查看是否成功

进入 MySQL 后执行：

```
-- 查看数据库是否存在
SHOW DATABASES;

-- 查看用户是否存在
SELECT user, host FROM mysql.user;

-- 查看 TFMSV 用户权限
SHOW GRANTS FOR 'TFMSV'@'%';
```

如果能看到 `TFMSV` 数据库、用户和授权信息，就表示创建成功。

![xin'j](TFMSV.assets/image-20251006150046119.png)

## **3.2 新建数据库并新建表 **

在 MySQL 里可以用下面命令：

```
USE tfmsv;           -- 切换到你的项目数据库
SHOW TABLES;          -- 查看这个数据库里的所有表
```

执行后会显示类似：

```
+----------------+
| Tables_in_tfmsv|
+----------------+
| User           |
| Result         |
| Dsample        |
+----------------+
```

## 一，先查看 `user` 表结构（确保字段名正确）

在 MySQL 命令行中执行：

```
DESC user;
DESC dsample;
```

这会显示类似下面的结果：

```
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| user_id   | int(11)      | NO   | PRI | NULL    | auto_increment |
| user_name | varchar(50)  | YES  |     | NULL    |                |
| password  | varchar(100) | YES  |     | NULL    |                |
| is_admin  | tinyint(1)   | YES  |     | 0       |                |
+-----------+--------------+------+-----+---------+----------------+
```

确认字段名称分别是：

- `user_id`（自增主键）
- `user_name`
- `password`
- `is_admin`（0：普通用户，1：管理员）

------

##  二、插入一个新用户（示例：管理员账号）

执行以下 SQL：

```sql
INSERT INTO user (user_name, password, is_admin) VALUES ('mnlc', '120455713p@P', 1);
INSERT INTO user (user_name, password, is_admin) VALUES ('quxinzhi', '123456', 1);
INSERT INTO user (user_name, password, is_admin) VALUES ('yangyaqi', '123456', 1);

SELECT * FROM user;
```

这会在 `user` 表中添加一个用户名为 `admin`、密码为 `123456` 的管理员账户。

------

##  三、验证是否插入成功

执行：

```
SELECT * FROM user;
```

如果看到类似结果：

```
+----------+-----------+----------+----------+
| user_id  | user_name | password | is_admin |
+----------+-----------+----------+----------+
|        1 | admin     | 123456   |        1 |
+----------+-----------+----------+----------+
```

说明插入成功 。

![image-20251030110730872](TFMSV.assets/image-20251030110730872.png)

------

## 四、然后前端登录测试

在前端登录页输入：

- 用户名：`admin`
- 密码：`123456`

点击登录后，后端的 `login()` 就会匹配成功并返回：

```
{
  "code": 0,
  "msg": "success",
  "token": "xxxxx..."
}
```



```
mysql> USE tfmsv;
Database changed
mysql> SHOW TABLES;
+-----------------+
| Tables_in_tfmsv |
+-----------------+
| annotation      |
| dsample         |
| eresult         |
| feature         |
| result          |
| sresults        |
| task            |
| user            |
+-----------------+
8 rows in set (0.058 sec)

mysql> SELECT * FROM user;
+---------+-----------+--------------+----------+
| user_id | user_name | password     | is_admin |
+---------+-----------+--------------+----------+
|       1 | mnlc      | 120455713p@P |        1 |
|       2 | quxinzhi  | 123456       |        1 |
|       3 | yangyaqi  | 123456       |        1 |
+---------+-----------+--------------+----------+
3 rows in set (0.044 sec)

mysql> SELECT * FROM feature;
+----+--------------+--------------+-----------+-----------+---------------------------------------------+------------------------------------------------------------------------------+-------------+
| id | dataset_name | feature_name | feature_T | feature_A | feature_V                                   | feature_path                                                                 | description |
+----+--------------+--------------+-----------+-----------+---------------------------------------------+------------------------------------------------------------------------------+-------------+
|  2 | SIMS         | SIMS_V       | None      | None      | openface&action_units&head_pose&landmark_2D | D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\SIMS\Processed\feature_2.pkl |             |
|  5 | SIMS         | SIMS_T       | bert      | None      | None                                        | D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\SIMS\Processed\feature_5.pkl |             |
|  7 | SIMS         | SIMS_V       | None      | None      | openface&action_units&head_pose&landmark_3D | D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\SIMS\Processed\feature_7.pkl |             |
+----+--------------+--------------+-----------+-----------+---------------------------------------------+------------------------------------------------------------------------------+-------------+
3 rows in set (0.055 sec)

mysql>
```



# 4.模型转化  (Flask后端)

## 4.1BERT预训练tensorflow模型转换为pytorch模型

- 从 [Google-Bert](https://github.com/google-research/bert) 下载 [Bert-Base，中文](https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip)。然后，使用 [transformers-cli](https://huggingface.co/transformers/converting_tensorflow_models.html) 将 Tensorflow 转换为 pytorch。将转换后的模型放在目录下。`MM-Codes/pretrained_model`

[BERT预训练tensorflow模型转换为pytorch模型 - 时光如你般美好 - 博客园](https://www.cnblogs.com/edkong/p/16032595.html)

```python
import transformers.models.bert.convert_bert_original_tf_checkpoint_to_pytorch as con

con.convert_tf_checkpoint_to_pytorch(
    r'.\bert_model.ckpt',
    r'.\bert_config.json',
    r'.\pytorch_bert.bin'
)
```

![image-20251006134844769](TFMSV.assets/image-20251006134844769.png)

![image-20251006134909648](TFMSV.assets/image-20251006134909648.png)



## 4.2 下载open space toolkit

![image-20251006135540007](TFMSV.assets/image-20251006135540007.png)

![image-20251013175412926](TFMSV.assets/image-20251013175412926.png)



## **4.3 configs 配置文件**

这一步是把你的项目路径和数据库信息配置好，以便程序能找到数据、模型和数据库。

1. **编辑 `constants.py`**
   文件中包含了很多路径常量，比如：

   ```
   DATASET_ROOT_DIR       # 数据集根目录
   DATASET_SERVER_IP      # 数据服务器 IP
   OPENFACE_FEATURE_PATH  # OpenFace 提取的特征存放路径
   MM_CODES_PATH           # 项目里的 MM-Codes 目录路径
   MODEL_TMP_SAVE          # 模型临时存放路径
   AL_CODES_PATH           # 项目里的 AL-Codes 目录路径
   LIVE_TMP_PATH           # 实时数据存放路径
   ```

   **操作：**

   - 根据你的电脑目录，把这些路径修改为实际存放位置。

   - 例如：

     ```
     DATASET_ROOT_DIR = "D:/TFMSV/M-SENA/Datasets"
     OPENFACE_FEATURE_PATH = "D:/TFMSV/M-SENA/OpenFace2/features"
     ```

2. **编辑 `config.sh`**
   文件中会有数据库连接字符串，比如：

   ```
   DATABASE_URL=mysql+pymysql://TFMSV:120455713p@P@localhost:3306/TFMSV
   ```

   **操作：**

   - 修改为你自己的数据库用户名、密码、端口和数据库名。
   - 这确保后续 Flask 服务能连接 MySQL 数据库。

------

## **4.4 Datasets 数据集准备**

1. **下载数据集并放到 `DATASET_ROOT_DIR` 下**

   - 数据集文件夹路径需要和 `constants.py` 中 `DATASET_ROOT_DIR` 一致。

   - 例如：

     ```
     D:/TFMSV/M-SENA/Datasets/MyDataset/
     ```

2. **注册数据集信息**

   - 找到 `DATASET_ROOT_DIR/config.json` 文件，添加你新数据集的信息。

   - 例如：

     ```
     {
       "MyDataset": {
         "language": "cn",
         "path": "D:/TFMSV/M-SENA/Datasets/MyDataset"
       }
     }
     ```

   这样程序才能识别新数据集。

3. **格式化数据集**

   - 使用项目里的 `MM-Codes/data/DataPre.py` 脚本把数据集整理成项目需要的格式。

   - 命令示例：

     ```
     python MM-Codes/data/DataPre.py --working_dir D:/TFMSV/M-SENA/Datasets/MyDataset --openface2Path D:/TFMSV/M-SENA/OpenFace2 --language cn
     ```

   参数说明：

   - `--working_dir`：你的数据集路径
   - `--openface2Path`：OpenFace2 的工具路径（用于提取人脸特征）
   - `--language`：数据集语言，`cn` 或 `en`

4. **如果数据集需要标注**

   - 配置文件位于 `AL-Codes` 目录中（Active Learning 相关代码），程序会读取标注信息。

![image-20251030100209735](TFMSV.assets/image-20251030100209735.png)

![image-20251030100246278](TFMSV.assets/image-20251030100246278.png)

------



## **4.5 Run 运行项目**

1. **加载配置文件**

   ```
   source config.sh
   ```

   - 在 Linux 或 Git Bash 中，这条命令会把 `config.sh` 中的环境变量（比如数据库连接字符串）加载到当前终端会话。

   - Windows PowerShell 下可以用：

     ```
     .\config.sh
     ```

2. **运行 Flask 服务**

   ```
   flask run --host=0.0.0.0
   ```

   - 启动项目的 Web 服务（通常提供界面或 API 接口）
   - `--host=0.0.0.0` 表示允许局域网内其他电脑访问这个服务。





# 5.创建Vue 前端

现在vue最新的是vue3，但是很多老项目都是vue2，vue3不兼容可以共存这里把项目设定为vue2，需要强制使用新的镜像源。

```
vue create tfmsv-frontend --registry=https://registry.npmmirror.com
```

![image-20251013192442173](TFMSV.assets/image-20251013192442173.png)

![image-20251013180727560](TFMSV.assets/image-20251013180727560.png)



# 6.部署后端Flask代码

## 6.1 更改后端Flask配置文件 constants.py

![image-20251027144458444](TFMSV.assets/image-20251027144458444.png)

## 6.2 报错：tensorflow旧版本接口报错解决

```yaml
(TFMSV) D:\TFMSV\M-SENA>D:/ANACONDA/envs/TFMSV/python.exe d:/TFMSV/M-SENA/tfmsv-backend/test_main.py
Testing basic imports...
OK: os, logging
OK: flask
OK: gevent

Testing local module imports...
OK: config.constants

[DEBUG IMPORT] Trying to import _tensorflow
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\huggingface_hub\serialization\__init__.py", line 18, in <module>
    from ._tensorflow import get_tf_storage_size, split_tf_state_dict_into_shards
  File "d:/TFMSV/M-SENA/tfmsv-backend/test_main.py", line 10, in debug_import
    traceback.print_stack(limit=3)

[DEBUG IMPORT] Trying to import tensorflow
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\transformers\image_transforms.py", line 50, in <module>
    import tensorflow as tf
  File "d:/TFMSV/M-SENA/tfmsv-backend/test_main.py", line 10, in debug_import
    traceback.print_stack(limit=3)

[DEBUG IMPORT] Trying to import tensorflow.tools.docs

(TFMSV) D:\TFMSV\M-SENA>
```

### 问题核心：**Hugging Face 版本与 TensorFlow 版本不兼容**

从 Transformers 4.41 起，它尝试调用 `tensorflow.tools.docs` 旧路径中的 `doc_controls`（早已废弃），
 这在 TensorFlow ≥ 2.10 中会直接报错。

### 关于 `transformers` 库的作用说明

`transformers` 是 HuggingFace 的自然语言处理库，作用是加载各种预训练模型（如 BERT、GPT、T5 等），用于：

- 文本情感分析（text sentiment analysis）
- 文本分类 / 生成 / 翻译 / 问答
- 多模态模型（CLIP、BLIP 等）





# 7.MMSA—FAT 特征提取工具包

MMSA-Feature Extraction Toolkit 为多模态情感分析数据集提取多模态特征。它集成了几种常用的视觉、听觉和文本模态工具。提取的特征与 [MMSA](https://github.com/thuiar/MMSA) 框架兼容，因此可以直接使用。该工具还可以提取单个视频的特征。

这项工作包含在 ACL-2022 演示论文中：[M-SENA：用于多模态情感分析的集成平台](https://github.com/thuiar/MMSA-FET/blob/master)。如果您发现我们的工作有用，请随时引用我们的论文。谢谢！

```
@article{mao2022m,
  title={M-SENA: An Integrated Platform for Multimodal Sentiment Analysis},
  author={Mao, Huisheng and Yuan, Ziqi and Xu, Hua and Yu, Wenmeng and Liu, Yihe and Gao, Kai},
  journal={arXiv preprint arXiv:2203.12441},
  year={2022}
}
```

- 为单个视频或数据集提取完全自定义的特征。
- 集成一些最常用的工具，包括 Librosa、OpenFace、Transformers 等。
- 支持当前发言人检测，以防视频中存在多张人脸。
- 易于使用，提供 Python API 和命令行工具。
- 提取的特征与 [MMSA 兼容，MMSA](https://github.com/thuiar/MMSA) 是多模态情感分析的统一训练和测试框架。

## 1. 安装

MMSA-Feature Extraction Toolkit 可从 PyPI 获得。由于 PyPi 上的包大小限制，大模型文件无法随包一起提供。用户需要运行安装后命令才能手动下载这些文件。如果您无法访问 Google Drive，请参阅[此页面](https://github.com/FlameSky-S/MMSA-FET/wiki/Dependency-Installation#2-post-installation-script)进行手动下载。

```
# Install package from PyPI
$ pip install MMSA-FET
# Download models & libraries from Google Drive. Use --proxy if needed.
$ python -m MSA_FET install
```

> **注意：**需要手动安装一些系统范围的依赖项。有关详细信息，请参阅[依赖项安装](https://github.com/FlameSky-S/MMSA-FET/wiki/Dependency-Installation)。

## 2. 快速入门

MMSA-FET 相当易于使用。您可以在 python 中调用 API 或使用命令行界面。下面是使用 python API 的基本示例。

> **注意：**要提取数据集的特征，需要将数据集组织成特定的文件结构，并且需要一个文件。有关详细信息，请参阅[数据集和结构](https://github.com/FlameSky-S/MMSA-FET/wiki/Dataset-and-Structure)。MOSI、MOSEI 和 CH-SIMS 的原始视频文件和标签文件可以从 [BaiduYunDisk](https://pan.baidu.com/s/1XmobKHUqnXciAm7hfnj2gg) 或 [Google Drive](https://drive.google.com/drive/folders/1A2S4pqCHryGmiqnNSPLv7rEg63WvjCSk?usp=sharing) 下载。`label.csv``code: mfet`

```
from MSA_FET import FeatureExtractionTool
from MSA_FET import run_dataset

# initialize with default librosa config which only extracts audio features
fet = FeatureExtractionTool("openface")

# alternatively initialize with a custom config file
fet = FeatureExtractionTool("custom_config.json")

# extract features for single video
feature1 = fet.run_single("input1.mp4")
print(feature1)
feature2 = fet.run_single("input2.mp4")

# extract for dataset & save features to file
run_dataset(
    config = "aligned",
    dataset_dir="~/MOSI", 
    out_file="output/feature.pkl",
    num_workers=4
)
```

是自定义配置文件的路径，其格式如下所述。`custom_config.json`

详细用法请阅读 [API](https://github.com/FlameSky-S/MMSA-FET/wiki/APIs) 和[命令行参数。](https://github.com/FlameSky-S/MMSA-FET/wiki/Command-Line-Arguments)

## 3. 配置文件

MMSA-FET 附带了一些示例配置，可以使用如下所示。

```
# Each supported tool has an example config
fet = FeatureExtractionTool(config="aligned")
fet = FeatureExtractionTool(config="librosa")
fet = FeatureExtractionTool(config="opensmile")
fet = FeatureExtractionTool(config="wav2vec")
fet = FeatureExtractionTool(config="openface")
fet = FeatureExtractionTool(config="mediapipe")
fet = FeatureExtractionTool(config="bert")
fet = FeatureExtractionTool(config="roberta")
```

对于自定义功能，您可以：

1. 编辑默认配置并将字典传递给配置参数，如下例所示：

```
from MSA_FET import FeatureExtractionTool, get_default_config

# here we only extract audio and video features
config_a = get_default_config('opensmile')
config_v = get_default_config('openface')

# modify default config
config_a['audio']['args']['feature_level'] = 'LowLevelDescriptors'

# combine audio and video configs
config = {**config_a, **config_v}

# initialize
fet = FeatureExtractionTool(config=config)
```



1. 提供配置 json 文件。下面的示例提取了所有三种模式的特征。要提取单峰特征，只需从文件中删除不必要的部分即可。

```
{
  "audio": {
    "tool": "librosa",
    "sample_rate": null,
    "args": {
      "mfcc": {
        "n_mfcc": 20,
        "htk": true
      },
      "rms": {},
      "zero_crossing_rate": {},
      "spectral_rolloff": {},
      "spectral_centroid": {}
    }
  },
  "video": {
    "tool": "openface",
    "fps": 25,
    "average_over": 3,
    "args": {
      "hogalign": false,
      "simalign": false,
      "nobadaligned": false,
      "landmark_2D": true,
      "landmark_3D": false,
      "pdmparams": false,
      "head_pose": true,
      "action_units": true,
      "gaze": true,
      "tracked": false
    }
  },
  "text": {
    "model": "bert",
    "device": "cpu",
    "pretrained": "models/bert_base_uncased",
    "args": {}
  }
}
```



## 4. 支持的工具和功能

### 4.1 音频工具

- **Librosa**（[链接](https://librosa.org/doc/latest/index.html))

  支持[此处](https://librosa.org/doc/latest/feature.html)列出的所有 librosa 功能，包括：[mfcc](https://librosa.org/doc/latest/generated/librosa.feature.mfcc.html#librosa.feature.mfcc)、[rms](https://librosa.org/doc/latest/generated/librosa.feature.rms.html#librosa.feature.rms)、[zero_crossing_rate](https://librosa.org/doc/latest/generated/librosa.feature.zero_crossing_rate.html#librosa.feature.zero_crossing_rate)、[spectral_rolloff](https://librosa.org/doc/latest/generated/librosa.feature.spectral_rolloff.html#librosa.feature.spectral_rolloff)、[spectral_centroid](https://librosa.org/doc/latest/generated/librosa.feature.spectral_centroid.html#librosa.feature.spectral_centroid) 等。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#11-librosa)。

- **openSMILE**（[链接](https://audeering.github.io/opensmile-python/))

  支持[此处](https://audeering.github.io/opensmile-python/api-smile.html#featureset)列出的所有功能集，包括：[ComParE_2016](http://www.tangsoo.de/documents/Publications/Schuller16-TI2.pdf)、[GeMAPS、](https://sail.usc.edu/publications/files/eyben-preprinttaffc-2015.pdf)[eGeMAPS、](https://sail.usc.edu/publications/files/eyben-preprinttaffc-2015.pdf)emobase 等。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#12-opensmile)。

- **Wav2vec2** （[链接](https://huggingface.co/docs/transformers/model_doc/wav2vec2))

  由 huggingface 变压器集成。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#13-wav2vec2)。

### 4.2 视频工具

- **OpenFace**（[链接](https://github.com/TadasBaltrusaitis/OpenFace))

  支持 OpenFace 的 FeatureExtraction 二进制文件中的所有功能，包括：2D 和 3D 面部标志、头部姿势、凝视相关、面部动作单元、HOG 二进制文件。这些功能的详细信息可以在 OpenFace Wiki 中找到 [点击](https://github.com/TadasBaltrusaitis/OpenFace/wiki/Output-Format)此处 和 [点击此处](https://github.com/TadasBaltrusaitis/OpenFace/wiki/Action-Units).详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#21-openface)。

- **MediaPipe**（[链接](https://google.github.io/mediapipe/))

  支持[面部](https://google.github.io/mediapipe/solutions/face_mesh.html)网格和[整体](https://google.github.io/mediapipe/solutions/holistic)（面部、手部、姿势）解决方案。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#22-mediapipe)。

- **TalkNet**（[链接](https://github.com/TaoRuijie/TalkNet_ASD))

  TalkNet 用于在视频中有多个人脸的情况下支持主动发言人检测。

### 4.3 文本工具

- **BERT**（[链接](https://huggingface.co/docs/transformers/model_doc/bert))

  由 huggingface 变压器集成。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#31-bert)。

- **XLNet**（[链接](https://huggingface.co/docs/transformers/model_doc/xlnet))

  由 huggingface 变压器集成。详细配置[可以在这里找到](https://github.com/FlameSky-S/MMSA-FET/wiki/Configurations#32-xlnet)。

### 4.4 矫正器

- **Wav2vec CTC 矫治器**

  使用预训练的 Wav2vec ASR 模型为每个单词生成时间戳，然后将视频和音频与文本对齐。目前仅支持英语。





# 8.TFN模型部署

 **Tensor Fusion Network（TFN）在 Keras + Theano 框架下的实现核心部分**，用于 **多模态情感分析（Multimodal Sentiment Analysis）**，以 **CMU-MOSI** 数据集为实验对象。
 它融合了三种模态特征：
 🗣️ **语音（Covarep）**、😊 **视觉（Facet）**、📝 **文本（Text）**。

我会帮你 **分模块详细解释代码逻辑与每一部分的作用**👇

------

##  一、导入依赖与设置随机种子

```
import data_loader as loader
import numpy as np
seed = 123
np.random.seed(seed)
```

- 设定随机种子保证实验可复现。
- `data_loader` 是作者自定义的数据加载脚本，用于从 CMU-MOSI 数据集中提取三种模态特征。

------

##  二、引入依赖库（Theano + Keras + sklearn）

```
from keras.models import Sequential, Model
from keras.layers import Dense, LSTM, Merge, Dropout, Embedding, Reshape, BatchNormalization
from keras.callbacks import EarlyStopping, ModelCheckpoint
from keras.optimizers import SGD, Adamax
```

- 主要使用 **Keras** 构建网络模型（当时版本依赖 Theano 后端）。
- 引入早停（EarlyStopping）与模型保存（ModelCheckpoint）。
- 引入 sklearn 度量指标计算（如准确率、混淆矩阵）。

------

## 三、命令行参数定义

```
parser = argparse.ArgumentParser()
parser.add_argument('--dropout', default=0.2, type=float)
parser.add_argument('--lr', default=0.01, type=float)
parser.add_argument('--units', default=160, type=int)
parser.add_argument('--optimizer', default='adam', type=str)
parser.add_argument('--pretrain', default=0, type=int)
parser.add_argument('-f', '--feature', default=[], type=list)
...
args = parser.parse_args()
```

提供灵活的命令行控制，包括：

- dropout、学习率（10^(-2)）、优化器（Adam/SGD）；
- 是否预训练（仅使用文本模态）；
- 是否使用卷积层、注意力层；
- 是否启用特征选择。

------

## 四、数据加载与预处理

```
train, valid, test = loader.load_word_level_features(max_segment_len, tr_split)
```

- 加载 MOSI 数据集中的 **文本、音频、视觉特征**。
- 每条样本是一个视频片段的三模态数据（对齐后特征长度一致）。

### 特征选择部分

```
if args.feature_selection:
    with open('/media/bighdd5/Paul/mosi/fs_mask.pkl') as f:
        [covarep_ix, facet_ix] = pickle.load(f)
```

- 载入一个事先计算好的 **特征掩码（mask）** 文件，只选择重要的音频/视觉特征维度；
- 否则默认使用全部特征。

### 特征归一化

```
facet_train_max = np.max(np.max(np.abs(facet_train ), axis =0),axis=0)
facet_train = facet_train / facet_train_max
```

- 对 `facet`（视觉特征）按通道归一化，保证不同维度范围一致。

------

## 五、划分训练与测试特征

```
covarep_train = np.mean(covarep_train, axis=1)
facet_train = np.mean(facet_train, axis=1)
```

- 对每个片段取 **时间维度的平均值**（即一个视频段只保留全局特征向量）。

```
X_train = np.concatenate((covarep_train, facet_train, text_train), axis=1)
```

- 拼接得到最终输入特征矩阵。

------

##  六、单模态子网络结构定义

### 音频模态（Covarep）

```
Covarep_model = Sequential()
Covarep_model.add(BatchNormalization(input_shape=(f_Covarep_num,)))
Covarep_model.add(Dropout(0.2))
Covarep_model.add(Dense(32, activation='relu'))
Covarep_model.add(Dense(32, activation='relu'))
```

###  视觉模态（Facet）

与上面类似，也是三层 Dense 网络，每层 ReLU 激活。

### 文本模态

```
text_model = Sequential()
text_model.add(Embedding(...))
text_model.add(LSTM(128))
text_model.add(Dense(64))
```

- 使用预训练词向量嵌入（`word_embedding`）；
- 通过 LSTM 获取句子语义；
- 输出一个 64 维向量。

------

## 七、模态融合（Fusion Layer）

###  Cartesian Fusion（张量融合，TFN 的核心）

```
if use_cartesian_fusion:
    dot_layer1 = Merge([Covarep_biased_model, Facet_biased_model], mode='dot', dot_axes=1)
    dot_layer2 = Merge([fusion_model_tmp, text_biased_model], mode='dot', dot_axes=1)
```

#### 逻辑：

- “张量融合”通过 **所有模态之间的笛卡尔积（外积）** 将模态交互显式建模。
- 先融合音频和视觉 → 再与文本模态进行张量积 → Flatten 成一维特征向量。
- 这种设计能显式捕捉到**模态间的高阶交互关系**（例如语气+表情+文字组合的情绪特征）。

------

## 八、输出层与训练配置

```
fusion_model.add(Dense(args.units, activation='relu'))
fusion_model.add(Dense(1, activation='sigmoid'))
fusion_model.add(Dense(1, weights=[np.array([[b]]), np.array([-a])], trainable=False))
```

- 最后两层是情感值映射层（线性变换使输出范围对齐到 [−a,b][-a, b][−a,b]，论文中常用于回归任务）。
- Loss：`mae`（平均绝对误差）。

### 训练配置：

```
EarlyStopping(monitor='val_loss', patience=train_patience)
ModelCheckpoint(weights_path, save_best_only=True)
```

### 优化器选择：

```
optimizer = {'sgd': sgd, 'adam':adam}
fusion_model.compile(loss=args.loss, optimizer=optimizer[args.optimizer])
```

------

## 九、模型训练与评估

```
fusion_model.fit([...], y_train, validation_split=val_split, nb_epoch=train_epoch)
fusion_model.load_weights(weights_path)
predictions = fusion_model.predict([...])
```

训练完成后，重新载入最佳验证集模型进行预测。

### 性能评估

```
mae = np.mean(np.absolute(predictions-y_test))
print "corr:", np.corrcoef(predictions, y_test)[0][1]
print confusion_matrix(true_label, predicted_label)
```

计算：

- 平均绝对误差（MAE）
- 相关系数（Corr）
- 混淆矩阵、准确率、分类报告（通过 `sklearn.metrics`）

------

##  十、整体结构概览

```
        ┌──────────────┐
        │ Text (LSTM)  │──┐
        └──────────────┘  │
        ┌──────────────┐  │   张量积融合（Tensor Fusion）
Audio → │ Covarep (MLP)│──┼──► 外积交互 → Dense → 输出情感
        └──────────────┘  │
        ┌──────────────┐  │
Visual→ │ Facet  (MLP) │──┘
        └──────────────┘
```

输出是一个连续值（情感强度），也可二值化成正/负情感。

------

## 总结

| 模块               | 功能                             |
| ------------------ | -------------------------------- |
| `data_loader`      | 加载多模态特征（文本/语音/视觉） |
| Covarep/FACET 模型 | 提取音频与视觉低层特征           |
| Text 模型          | LSTM 获取句子语义                |
| Tensor Fusion      | 通过外积实现模态间高阶融合       |
| Dense 层           | 情感回归/分类                    |
| 评价指标           | MAE、相关系数、分类准确率        |







# 9.报错总结

## 1.无法成功扫描数据集和特征集加入到数据库中

```yaml
2025-11-10 21:02:56,064 - app - ==================== scan_datasets started ====================
2025-11-10 21:02:56,069 - app - Admin check passed.
2025-11-10 21:02:56,070 - app - Attempting to read config file from: D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\config.json
2025-11-10 21:02:56,075 - app - Config file loaded. Found datasets: ['SIMS', 'CMU-MOSI']
2025-11-10 21:02:56,076 - app - --- Processing dataset: SIMS ---
2025-11-10 21:02:56,076 - app - Attempting to read label file: D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\SIMS/label.csv
2025-11-10 21:02:56,097 - app - Label file loaded. Number of rows: 2281
2025-11-10 21:02:56,098 - app - Starting to loop through rows and prepare for DB insertion...
2025-11-10 21:02:56,100 - app - [WinError 6] 句柄无效。
Traceback (most recent call last):
  File "d:\TFMSV\M-SENA\tfmsv-backend\app\data.py", line 119, in scan_datasets
    for i in tqdm(range(len(label_df)), desc=f"Processing {dataset_name}"): # 删掉 disable=True
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\tqdm\std.py", line 1096, in __init__
    self.sp = self.status_printer(self.fp)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\tqdm\std.py", line 448, in status_printer
    getattr(sys.stderr, 'flush', lambda: None)()
OSError: [WinError 6] 句柄无效。
2025-11-10 21:02:58,967 - app - API called: /user/verifyToken
2025-11-10 21:02:59,512 - app - API called: /modelEnd/getAllSettings
2025-11-10 21:02:59,528 - app - API called: /featureEnd/getFeatureList
2025-11-10 21:03:02,692 - app - API called: /featureEnd/scanDefaultFeatures
2025-11-10 21:03:02,722 - app - (MySQLdb.IntegrityError) (1062, "Duplicate entry 'SIMS-Default_unaligned-Unknown-Unknown-Unknown' for key 'feature.ix_dataset_feature_T_A_V'")
[SQL: INSERT INTO `Feature` (dataset_name, feature_name, `feature_T`, `feature_A`, `feature_V`, feature_path, description) VALUES (%s, %s, %s, %s, %s, %s, %s)]
[parameters: ('SIMS', 'Default_unaligned', 'Unknown', 'Unknown', 'Unknown', 'D:\\TFMSV\\M-SENA\\tfmsv-backend\\MM-Codes\\datasets\\SIMS/Processed/features/unaligned_39.pkl', '')]
(Background on this error at: https://sqlalche.me/e/14/gkpj)
Traceback (most recent call last):
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 1900, in _execute_context
    self.dialect.do_execute(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\default.py", line 736, in do_execute
    cursor.execute(statement, parameters)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\cursors.py", line 179, in execute
    res = self._query(mogrified_query)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\cursors.py", line 330, in _query
    db.query(q)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\connections.py", line 261, in query
    _mysql.connection.query(self, query)
MySQLdb.IntegrityError: (1062, "Duplicate entry 'SIMS-Default_unaligned-Unknown-Unknown-Unknown' for key 'feature.ix_dataset_feature_T_A_V'")

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "d:\TFMSV\M-SENA\tfmsv-backend\app\feature.py", line 90, in scan_default_features
    db.session.commit()
  File "<string>", line 2, in commit
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 1451, in commit
    self._transaction.commit(_to_root=self.future)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 829, in commit
    self._prepare_impl()
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 808, in _prepare_impl
    self.session.flush()
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 3446, in flush
    self._flush(objects)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 3586, in _flush
    transaction.rollback(_capture_exception=True)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\util\langhelpers.py", line 70, in __exit__
    compat.raise_(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\util\compat.py", line 211, in raise_
    raise exception
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\session.py", line 3546, in _flush
    flush_context.execute()
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\unitofwork.py", line 456, in execute
    rec.execute(self)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\unitofwork.py", line 630, in execute
    util.preloaded.orm_persistence.save_obj(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\persistence.py", line 245, in save_obj
    _emit_insert_statements(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\orm\persistence.py", line 1238, in _emit_insert_statements
    result = connection._execute_20(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 1705, in _execute_20
    return meth(self, args_10style, kwargs_10style, execution_options)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\sql\elements.py", line 334, in _execute_on_connection
    return connection._execute_clauseelement(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 1572, in _execute_clauseelement
    ret = self._execute_context(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 1943, in _execute_context
    self._handle_dbapi_exception(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 2124, in _handle_dbapi_exception
    util.raise_(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\util\compat.py", line 211, in raise_
    raise exception
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\base.py", line 1900, in _execute_context
    self.dialect.do_execute(
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\sqlalchemy\engine\default.py", line 736, in do_execute
    cursor.execute(statement, parameters)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\cursors.py", line 179, in execute
    res = self._query(mogrified_query)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\cursors.py", line 330, in _query
    db.query(q)
  File "D:\ANACONDA\envs\TFMSV\lib\site-packages\MySQLdb\connections.py", line 261, in query
    _mysql.connection.query(self, query)
sqlalchemy.exc.IntegrityError: (MySQLdb.IntegrityError) (1062, "Duplicate entry 'SIMS-Default_unaligned-Unknown-Unknown-Unknown' for key 'feature.ix_dataset_feature_T_A_V'")
[SQL: INSERT INTO `Feature` (dataset_name, feature_name, `feature_T`, `feature_A`, `feature_V`, feature_path, description) VALUES (%s, %s, %s, %s, %s, %s, %s)]
[parameters: ('SIMS', 'Default_unaligned', 'Unknown', 'Unknown', 'Unknown', 'D:\\TFMSV\\M-SENA\\tfmsv-backend\\MM-Codes\\datasets\\SIMS/Processed/features/unaligned_39.pkl', '')]
(Background on this error at: https://sqlalche.me/e/14/gkpj)
2025-11-10 21:03:07,282 - app - API called: /user/verifyToken
2025-11-10 21:03:07,518 - app - API called: /modelEnd/getAllSettings
2025-11-10 21:03:08,569 - app - API called: /user/verifyToken
2025-11-10 21:03:08,705 - app - API called: /modelEnd/getAllSettings
2025-11-10 21:03:08,726 - app - API called: /featureEnd/getFeatureList
```

- **1.问题所在**：错误发生在 `tqdm` 库内部，当它试图刷新标准错误流 (`sys.stderr`) 来更新进度条时。
- **原因**：这个错误在 Windows 上很常见，特别是当你在非标准的命令行环境（如某些 IDE 的内置终端、或者通过特定方式启动的子进程）中运行 Python 脚本时。在这种环境下，`sys.stdout` 或 `sys.stderr` 可能不是一个有效的控制台句柄，导致 `tqdm` 无法在上面绘制进度条。你的 Flask WSGI 服务器可能就是以这种方式运行的。
- **验证**：你的日志显示，在 `tqdm` 出错之前，`Label file loaded. Number of rows: 2281` 已经成功打印。这证明了我们之前的猜测：`label.csv` 文件被成功找到了，并且里面有数据！问题不在于文件，而在于 `tqdm` 这个库无法在当前环境下工作。



- **2.问题所在**：程序试图向 `Feature` 表中插入一条新记录。
- **原因**：`Feature` 表有一个**唯一约束 (unique index)**，叫做 `ix_dataset_feature_T_A_V`。这个约束要求 `(dataset_name, feature_name, feature_T, feature_A, feature_V)` 这几个字段的组合必须是唯一的，不能重复。你的程序试图插入一条 `('SIMS', 'Default_unaligned', 'Unknown', 'Unknown', 'Unknown')` 的记录，但数据库里**已经存在**一条一模一样的记录了，所以 MySQL 拒绝了这次插入，并抛出了 `(1062, "Duplicate entry ...")` 错误。
- **根源**：这个 `/featureEnd/scanDefaultFeatures` 接口的设计可能存在缺陷。它应该在插入前先检查记录是否存在，或者使用 `INSERT ... ON DUPLICATE KEY UPDATE`（就像你在 `scan_datasets` 里做的那样）来避免这个错误。



## 2.前端界面无法成功播放视频

从你的 `__init__.py` 看，Flask 的创建方式是：

```
app = Flask(__name__)
```

这意味着 Flask 默认的静态文件目录是：

```
<M-SENA项目根目录>\static\
```

所以当你访问：

```
http://localhost:8096/SIMS/Raw/video_0001/0001.mp4
```

Flask 实际会去找：

```
D:\TFMSV\M-SENA\static\SIMS\Raw\video_0001\0001.mp4
```

然而你的视频文件在：

```
D:\TFMSV\M-SENA\tfmsv-backend\MM-Codes\datasets\SIMS\Raw\video_0001\0001.mp4
```

⚠️ 所以 Flask 找不到这个文件 → 返回 404。

```
(TFMSV) D:\TFMSV\M-SENA>curl -I http://localhost:8096/SIMS/
HTTP/1.0 404 File not found
Server: SimpleHTTP/0.6 Python/3.8.20
Date: Tue, 11 Nov 2025 07:11:55 GMT
Connection: close
Content-Type: text/html;charset=utf-8
Content-Length: 469


(TFMSV) D:\TFMSV\M-SENA>curl -I http://localhost:8096/     
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.8.20
Date: Tue, 11 Nov 2025 07:12:13 GMT
Content-type: text/html; charset=utf-8
Content-Length: 297


(TFMSV) D:\TFMSV\M-SENA>    我在http://localhost:8096/  这个界面下看到的是Directory listing for /    但是里面是空的
```





## 3.无法完成标注工作

### 初始化 `annotation` 表

给每个样本生成一条默认空标注记录：

```
INSERT INTO annotation (user_name, dataset_name, video_id, clip_id, label_M, label_T, label_A, label_V)
SELECT 
    'mnlc' AS mnlc,  
    d.dataset_name,
    d.video_id,
    d.clip_id,
    NULL AS label_M,
    NULL AS label_T,
    NULL AS label_A,
    NULL AS label_V
FROM dsample d
WHERE d.dataset_name = 'SIMS'
  AND NOT EXISTS (
    SELECT 1 FROM annotation a 
    WHERE a.dataset_name = d.dataset_name 
      AND a.video_id = d.video_id 
      AND a.clip_id = d.clip_id
);


INSERT INTO annotation (user_name, dataset_name, video_id, clip_id, label_M, label_T, label_A, label_V)
SELECT 
    'mnlc' AS mnlc,  
    d.dataset_name,
    d.video_id,
    d.clip_id,
    NULL AS label_M,
    NULL AS label_T,
    NULL AS label_A,
    NULL AS label_V
FROM dsample d
WHERE d.dataset_name = 'CMU-MOSI'
  AND NOT EXISTS (
    SELECT 1 FROM annotation a 
    WHERE a.dataset_name = d.dataset_name 
      AND a.video_id = d.video_id 
      AND a.clip_id = d.clip_id
);
```

然后再次打开标注界面时，后端就能找到“未标注”的样本。





## 4.无法完成训练工作

### 1.tqdm进度条问题

```python
    def __init_mosi(self):
        # 错误：用了方括号 []，被当成了字典取值 , 包括要加 False
        if self.args['custom_feature', False]:
            # use custom feature file extracted with MMSA-FET
            with open(self.args['custom_feature'], 'rb') as f:
                data = pickle.load(f)
                
     def __init_mosi(self):
        # 正确：使用 .get() 方法
        if self.args.get('custom_feature', False):
            # use custom feature file extracted with MMSA-FET
            with open(self.args['custom_feature'], 'rb') as f:
                data = pickle.load(f)
```





### 2.对于 **回归（regression）** 任务，模型输出的不是分类结果

![image-20251119214055478](TFMSV.assets/image-20251119214055478.png)

根据你给的训练参数：

```
'train_mode': 'regression',
'num_classes': 3,
```

对于 **回归（regression）** 任务，模型输出的不是分类结果，因此：

- 没有 accuracy（acc）
- 更不可能有 Non-zero accuracy（Non0_acc、Non0_acc_2）
- 回归任务应该只有：
  - MAE
  - MSE
  - RMSE
  - PCC（皮尔逊相关）

但 **你的 run.py 却强行在 regression 模式下取分类指标**：

```
results_dict['final_results']['test']['Non0_acc_2']
```

→ 所以直接 KeyError。

------

 **为什么 M-SENA 在 regression 模式下会访问分类指标？**

因为你的 `run.py` 这部分代码默认 TFN 在多分类任务下运行。

比如：

```
# run.py 768
results_dict['final_results']['test']['Non0_acc_2'],
```

这一段逻辑本来用于 **三分类任务（例如 sentiment）**。

但你这次用的是：

```
train_mode = regression
dataset = SIMS (cn)
```

SIMS 的 TFN **支持回归任务**，代码没判断任务类型



### 3.和我们平时学YOLO的区别：

#### 1. 为什么比 YOLO 快这么多？（核心原因）

YOLO 的训练（吃生肉）

- **输入**：原始图片（例如 640×640640×640 像素）。
- **计算量**：模型（Backbone）需要用卷积层（CNN）逐个像素去扫描、提取特征。这就像给你一堆生土豆，你要从洗土豆、削皮、切丝开始做起。
- **数据量**：通常包含成千上万张图片，每张图片都要算很久。

你的 MMSA 模型（吃熟食）

- 输入

  预提取特征 (Pre-extracted Features)

  - 请看你的日志参数：`featurePath: '.../unaligned_39.pkl'`。
  - 这意味着：**最耗时的“洗菜切菜”工作（特征提取）已经做完了！**
  - 在这个 `.pkl` 文件里，视频不再是图片帧，而是已经变成了数学向量（例如：视频特征维度 709，文本 768，音频 33）。

- **计算量**：TFN 模型不需要处理数百万个像素点，它只需要处理这几个数学向量的乘法。计算量比 YOLO 小了好几个数量级。

- **数据量**：日志显示 `train_samples: 1368`。只有 1000 多条数据，这在深度学习里是非常迷你的数据集。即使在 CPU 上，跑一个 Epoch（轮次）可能也就几秒钟。

### 2. 是否有早停机制？（Early Stopping）有的兄弟有的

```
'early_stop': 8
```

这意味着：**如果验证集（Validation Set）的 Loss 在连续 8 个 Epoch 内没有下降，模型就会认为“我学不动了，再学也是过拟合”，然后强行停止训练。**

由于 SIMS 数据集比较小，模型可能在前 10-20 个 Epoch 就收敛了，加上早停机制，所以训练过程（前 4 分钟）非常快就结束了。

### 3. 为什么 PCA 跑了那么久？（17分钟）

日志显示：

- **训练耗时**：约 4 分钟 (23:07 - 23:11)。
- **PCA 耗时**：约 17 分钟 (23:11 - 23:28)。

**原因：**
PCA（主成分分析）是一种数学降维算法。虽然你的训练很快，但在 CPU 上对大量高维向量（尤其是 Text 的 768 维 + Video 的 709 维）进行矩阵分解（SVD）是非常慢的。
这就好比：你花 5 分钟炒好了一盘菜（训练），但花 20 分钟在摆盘雕花（PCA降维画图）。



## 10.项目深度理解

**多模态融合模型（Multimodal Fusion Models）**。它们的核心能力是将**文本(T)、音频(A)、视频(V)**三种信息结合起来，生成一个特征向量。

至于这个向量最后用来做什么（是预测一个连续分数做**回归**，还是预测一个类别做**分类**），完全取决于：

1. **最后一层全连接层（Output Layer）的维度**（1维是回归，2维/多维是分类）。
2. **损失函数（Loss Function）**（L1Loss/MSELoss 是回归，CrossEntropy 是分类）。
3. **你的配置文件 `config_regression.json`**：你的代码里默认加载了这个配置，把 `train_mode` 设为了 `regression`，所以目前它们都在跑回归任务。

- 若 `is_tune=True`：进行超参搜索（grid/random），多次训练，结果写到一个 csv。
- 若 `is_tune=False`：按多个 seeds 重复训练，统计平均/方差，结果写到 csv。

#### 1. 早期/简单融合 (Early/Simple Fusion)

这些是基准模型，结构简单，跑得快。

- LF_DNN (Late Fusion DNN)

  - **作用**：最简单的“后融合”。先把 T, A, V 分别通过神经网络提取特征，最后把三个特征拼起来，扔给一个全连接层。
  - **特点**：基准线（Baseline），用来测试融合有没有必要。

- EF_LSTM (Early Fusion LSTM)

  - **作用**：最简单的“前融合”。先把 T, A, V 的原始特征拼在一起，然后扔给一个 LSTM（长短期记忆网络）去处理时间序列。
  - **特点**：适合捕捉时间依赖，但容易忽略模态间的差异。

  

  **学习率为0.01时 此时会有问题**

  模型训练结果（Loss, Corr）出现了 `NaN`，说明模型并没有正常收敛。

  问题可以在 `SENA_run` 这里直接规避，而且基本可以确定就是某些指标或预测值变成了 `NaN`，在通过 `mysql.connector` 作为参数传给 SQL 时，被转成了字面量 `nan`，从而触发：

  > Unknown column 'nan' in 'field list'

  在你贴出的这两个函数里，**只要会把浮点数写入 MySQL 的地方都有风险**，主要是：

  （`ef_lstm`）在训练过程中，某个指标（例如 Loss 损失值、Accuracy 准确率等）计算结果出现了 **`NaN`** (Not a Number，非数值)。这通常是因为梯度爆炸、学习率过大或除以零导致的。

  1. 将模型计算出的 `NaN`（非法数值）转换为了 `None`。
  2. **当前冲突**：你的 MySQL 数据库表结构规定 `corr` 列**必须有值**，不能是空的。
  3. **结果**：`NaN` -> `safe_float` -> `None` -> **数据库报错**。

![image-20251123112502535](TFMSV.assets/image-20251123112502535.png)



![image-20251123113722463](TFMSV.assets/image-20251123113722463.png)

#### 2. 张量融合 (Tensor Fusion) - 捕捉模态间交互

这类模型试图通过数学方法算出“文本和视频同时发生变化”时的特征。

- TFN (Tensor Fusion Network)

  (你正在用的模型)

  - **作用**：计算 T, A, V 三个向量的外积（Cartesian Product），生成一个巨大的 3D 张量，捕捉所有模态两两之间、三三之间的交互。
  - **特点**：效果好，但计算量大，生成的特征维度极高。

- LMF (Low-rank Multimodal Fusion)

  - **作用**：TFN 的“轻量版”。利用低秩矩阵分解技术，近似计算 TFN 的那个大张量。
  - **特点**：效果接近 TFN，但参数更少，速度更快。

#### 3. 记忆与门控机制 (Memory & Gating)

这类模型专门处理长序列，试图记住历史信息。

- MFN (Memory Fusion Network)
  - **作用**：给每种模态一个 LSTM，然后用一个特殊的“Delta-memory”注意力机制来跨时间、跨模态地整合信息。
  - **特点**：适合处理上下文依赖很强的视频。
- Graph_MFN
  - **作用**：在 MFN 的基础上引入了图神经网络（GNN），把模态看作图的节点。
- MFM (Multimodal Factorization Model)
  - **作用**：试图将特征分解为“多模态共享特征”和“模态独有特征”，以此提高学习效率。

#### 4. 基于 Transformer (Transformer-based) - 目前最强

这类模型利用 Attention 机制，是目前 SOTA（State-of-the-Art）的主流。

- MulT (Multimodal Transformer)

  - **作用**：核心是“跨模态注意力（Cross-modal Attention）”。比如，用文本去“注意”视频里的视觉信息，反之亦然。
  - **特点**：不需要对齐数据（Unimodal alignment），处理长序列能力极强，效果通常最好。

  ![image-20251122145221354](TFMSV.assets/image-20251122145221354.png)

- MISA (Modality-Invariant and -Specific Representations)

  - **作用**：将每个模态的特征投影到两个空间：一个是所有模态共享的（Invariant），一个是该模态独有的（Specific）。
  - **特点**：解耦特征，鲁棒性强，噪音少。

#### 5. 基于 BERT (BERT-based)

利用强大的预训练语言模型 BERT。

- BERT_MAG (BERT with Multimodal Adaptation Gate)
  - **作用**：把 BERT 作为主干，然后把音频和视频特征通过一个“门（Gate）”注入到 BERT 的内部层级中。
  - **特点**：因为有 BERT 加持，文本理解能力极强，通常在以语言为主的数据集上霸榜。

#### 6. 其他变体

- Self_MM / MMIM
  - **作用**：利用自监督学习（Self-supervised learning）或者互以信息最大化（Mutual Information Maximization）来辅助训练。
  - **特点**：不需要大量标签也能学得不错，旨在解决标签稀缺或模态不平衡问题。
- MLF_DNN / MTFN / MLMF
  - 这些通常是上述模型的多任务版本（Multi-task）或其他变体。



#### 7.回归模式下的指标（train_mode='regression'）

1）MOSI/MOSEI 回归（`__eval_mosei_regression`）：

对于连续情感分数（如 [-3,3]），MetricsTop 做了：

- MAE：`MAE = mean(|pred - truth|)`；
- Corr：皮尔逊相关系数 `np.corrcoef(preds, truths)[0][1]`；
- 7/5/3 级情绪等级准确率：
  - 将连续分数裁剪到 [-3,3] / [-2,2] / [-1,1]，并**四舍五入**为 7/5/3 档等级，求多类别准确率；
- Has0 / Non0 的二分类准确率与加权 F1：
  - 将分数 `>=0` 视为正类、`<0` 视为负类；
  - Non0 只在非 0 分数的样本上统计。

2）SIMS 回归（`__eval_sims_regression`）：

- 将预测与标签裁剪到 [-1,1]；
- 构造 2/3/5 类等级划分：
  - 2 类：[-1,0] 与 (0,1]；
  - 3 类：[-1,-0.1]、(-0.1,0.1]、(0.1,1]；
  - 5 类：[-1,-0.7]、(-0.7,-0.1]、(-0.1,0.1]、(0.1,0.7]、(0.7,1]；
- 计算对应的多分类准确率 `Mult_acc_2/3/5` 和二分类 F1；
- 同时给出 MAE 与 Corr。



#### 8.对于run函数的解读

1. `_set_logger(log_dir, model_name, dataset_name, verbose_level)`

作用：
为 MMSA 训练运行过程，创建一个专用的 logger，写日志文件 + 控制台输出。

主要步骤：

1. 构造日志文件路径：`log_dir / f"{model_name}-{dataset_name}.log"`。
2. 创建 logger：`logging.getLogger('MMSA')`，等级设为 DEBUG。
3. 创建文件 handler：
   - 写到上述 log 文件；
   - 格式：`时间 - 名称 [级别] - 内容`；
   - 等级 DEBUG。
4. 创建控制台 handler：
   - `verbose_level` 0/1/2 分别对应 ERROR / INFO / DEBUG；
   - 用于在 stdout 上显示训练信息。
5. 把两个 handler 加入 logger 并返回。

------

2. `MMSA_run(...)`

这是**MMSA 库的通用入口函数**，在不走 M-SENA 平台时，你直接用这个跑训练和测试。

```python
def MMSA_run(
    model_name: str, dataset_name: str, config_file: str = None,
    config: dict = None, seeds: list = [], is_tune: bool = False,
    tune_times: int = 50, custom_feature: str = None, feature_T: str = None, 
    feature_A: str = None, feature_V: str = None, gpu_ids: list = [0],
    num_workers: int = 4, verbose_level: int = 1,
    model_save_dir: str = Path().home() / "MMSA" / "saved_models",
    res_save_dir: str = Path().home() / "MMSA" / "results",
    log_dir: str = Path().home() / "MMSA" / "logs",
)
```

核心做两件事：

- 若 `is_tune=True`：进行超参搜索（grid/random），多次训练，结果写到一个 csv。
- 若 `is_tune=False`：按多个 seeds 重复训练，统计平均/方差，结果写到 csv。

### 2.1 初始化部分

- 把 `model_name`、`dataset_name` 小写化。

- 如果没指定

  ```
  config_file
  ```

  - `is_tune=True` → 默认使用 `config/config_tune.json`；
  - 否则使用 `config/config_regression.json`。

- 确保 `model_save_dir`、`res_save_dir`、`log_dir` 目录存在。

- 若 `seeds` 为空，则默认 `[1111, 1112, 1113, 1114, 1115]`。

- 调用 `_set_logger` 生成 logger。

### 2.2 调参模式 `is_tune=True`

逻辑概览：

1. log：开始 Tuning。

2. 调 `get_config_tune(model_name, dataset_name, config_file)`，读取调参配置，得到一个 `initial_args`。

3. 设置一些基础项：

   - `model_save_path`：模型权重保存路径。
   - `device`：通过 `assign_gpu` 选定 GPU。
   - 硬编码 `train_mode = 'regression'`（兼容原代码）。
   - 设置特征路径：`custom_feature` / `feature_T/A/V`。

4. 调用 `torch.cuda.set_device(initial_args['device'])`，确保 LSTM 等模块跑在指定 GPU 上。

5. 准备调参结果保存目录 `res_save_dir / "tune"`，csv 文件形如 `mosei-tfn.csv`。

6. 如果该 csv 已存在，就先读出来，从中恢复已经调过的参数集合 `has_debuged`（避免重复尝试相同超参）。

7. 然后进行

    

   ```
   tune_times
   ```

    

   次循环，每次：

   - 复制 `initial_args` 为 `args`；
   - 调 `get_config_tune` 再随机采一组合适参数放到 `args` 里；
   - 若传入 `config`，则用 `args.update(config)` 做覆盖；
   - 设置 `cur_seed`，记录日志；
   - 取出当前这组调参维度 `d_paras` 的值，看是否已调过，调过就 `continue`。
   - 设置随机种子 `setup_seed(seeds[0])`；
   - 调 `_run(args, num_workers, is_tune=True)` 训练+测试；
   - 把当前超参以及对应结果写入 csv。

**注意**：这里 `_run` 的返回是一个结果 dict（如 MAE、corr、F1 等），用于比较调参性能。

------

### 2.3 正常模式 `is_tune=False`

逻辑概览：

1. 用 `get_config_regression` 读取训练配置，为当前模型+数据集准备超参 `args`。
2. 设置：
   - `model_save_path`；
   - `device`；
   - 强制 `train_mode = 'regression'`（兼容老代码）；
   - 三种特征路径字段；
   - 若传入额外 `config`，则 `args.update(config)` 覆盖部分参数。
3. `torch.cuda.set_device(args['device'])` 指定 GPU。
4. 日志输出 args 和 seeds。
5. 结果存储目录换成 `res_save_dir / "normal"`。
6. 对每个 seed 循环：
   - `setup_seed(seed)` 固定随机性；
   - `args['cur_seed'] = i + 1`；
   - 调 `_run(args, num_workers, is_tune=False)`，拿到一次训练+测试的结果（一个 dict）。
   - 把结果追加到 `model_results`。
7. 用 `numpy` 统计不同 seeds 的结果均值和方差，对每个指标（如 MAE、corr 等）输出 `(mean, std)`，写入 csv。

