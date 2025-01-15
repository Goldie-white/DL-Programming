# 用VS Code实现Python远程主机编程

**连接远程主机：**

(1) 进入扩展市场，安装Remote-SSH

(2) 点击左下角的远程连接图标，选择Connect to Host-Add New SSH Host开始配置SSH连接

![image-20250115104354113](E:\桌面\Programming Foundations of DL\images\image-20250115104354113.png)

(3) SSH连接命令的格式：

```
ssh username@hostname -p port
```

将username、hostname（或者IP）和port替换为实际值，比如

```
ssh mickey@123.123.0.123 -p 100
```

按下回车后，VSCode会提示你保存这些信息到配置文件（通常位于`~/.ssh/config`或`C:\Users\your_username\.ssh\config`），这是方便以后快速连接；然后输入密码完成连接

**连接以后：**

(1) 查看系统版本

```
uname -a
```

(2) 安装python和miniconda（根据系统版本选择）

```
# 下载安装脚本
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# 用bash执行刚才安装的下载脚本
bash Miniconda3-latest-Linux-x86_64.sh
```

(3) 安装python和jupyter扩展（扩展商店）

(4) 把conda命令添加到环境变量中

```
export PATH=~/miniconda3/bin:$PATH
```

(5) 安装`ipykernel`：一个允许你在 Jupyter 笔记本和其他 IPython 前端中使用 Python 内核的包

```
conda install ipykernel --update-deps --force-reinstall
```

(6) 创建一个虚拟环境`diffusion`

```
# 创建环境
conda create -n diffusion python=3.10
# 激活环境
conda activate diffusion
# 验证当前环境
conda info --envs  # 或 conda env list  # 查看所有环境，当前环境会有星号标记
```

(7) 如果下载墙外网站，可以找镜像源替代下载地址，例如将 huggingface.co 改为 hf-mirror.com. 

注意，有时候直接替换源不好操作，例如下一段代码（调用模型默认采用原链接）：

```python
from diffusers import StableDiffusionPipeline
import torch

model_id = "SfinOe/stable-diffusion-v1.5"
pipe = StableDiffusionPipeline.from_pretrained(
    model_id, 
    torch_dtype=torch.float16,
    safety_checker=None  # 可选：如果遇到safety checker相关错误
)
pipe = pipe.to("cuda")

prompt = "A photo of a professor teaching machine learning in a bright, modern classroom"
image = pipe(prompt).images[0]  
    
image.save("professor_teach_ML.png")
```

由于模型调用的过程中会默认用原地址，而且层层调用比较难找到使用链接的具体位置，从而难以进行镜像替换，所以改为现在本地下载模型，然后从本地调用模型。

先在命令行下载模型到本地：

```
# 设置环境变量
export HF_ENDPOINT=https://hf-mirror.com

# 使用 huggingface-cli 从镜像下载
huggingface-cli download --resume-download SfinOe/stable-diffusion-v1.5 --local-dir /root/.cache/huggingface/hub/models--SfinOe--stable-diffusion-v1-5
```

检查模型是否下载到本地：

```python
import os

# 检查默认的 huggingface 缓存目录
cache_dir = "/root/.cache/huggingface/hub/"
model_dir = os.path.join(cache_dir, "models--SfinOe--stable-diffusion-v1-5")

def check_model_cache():
    print(f"检查缓存目录: {model_dir}")
    
    # 检查目录是否存在
    if not os.path.exists(model_dir):
        print("❌ 模型目录不存在")
        return
    
    print("✓ 模型目录存在")
    
    # 列出目录内容
    print("\n目录内容:")
    for root, dirs, files in os.walk(model_dir):
        level = root.replace(model_dir, '').count(os.sep)
        indent = ' ' * 4 * level
        print(f"{indent}{os.path.basename(root)}/")
        subindent = ' ' * 4 * (level + 1)
        for f in files:
            print(f"{subindent}{f}")

check_model_cache()
```

然后直接用本地文件即可

```python
from diffusers import StableDiffusionPipeline
import torch

# 直接使用本地路径
local_model_path = "/root/.cache/huggingface/hub/models--SfinOe--stable-diffusion-v1-5"

pipe = StableDiffusionPipeline.from_pretrained(
    local_model_path,
    torch_dtype=torch.float16,
    safety_checker=None,
    local_files_only=True  # 强制只使用本地文件
)
pipe = pipe.to("cuda")

prompt = "A photo of a professor teaching machine learning in a bright, modern classroom"
image = pipe(prompt).images[0]  
    
image.save("professor_teach_ML.png")
```

