# Conda、Jupyter

## Conda配置	

​	下载Annaconda时，会自动捆绑一个特定版本的 Python，如果需要其他Python版本，可以通过Conda创建独立环境并指定版本。通过Annaconda安装的包会默认安装在Conda环境中，而非系统Python。如果系统已安装 Python，Anaconda 的 Python 会通过环境变量优先级覆盖（仅在激活 Conda 环境时生效）。Anaconda已经自动为我们安装了Jupter Notebook及其他工具，还有python中超过180个科学包及其依赖项。

- Conda新环境工作目录

  Conda 默认将所有环境（包括新创建的）存储在以下路径中

  ```bash
  C:\Users\<你的用户名>\Anaconda3\envs\<环境名>
  ```

  如何修改环境的存储路径呢

  在`C:\Users\.condarc`文件（conda配置文件）中添加如下配置，这样每次创建新环境时就会在该指定的路径中

  ```yaml
  envs_dirs:
    - 存储的路径
  ```

- 为Conda配置清华镜像源

  同样的是在上面文件中（.condarc）添加如下配置

  ```yaml
  show_channel_urls: true
  default_channels:
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
  custom_channels:
    conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
    pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  ```

  可以使用命令`conda info`查看是否配置成功

  ![image-20250620172728469](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250620172728469.png)

- Conda多环境

  多环境的好处是Python版本以及依赖项不受影响，各自独立。创建一个新环境并下载指定的包及其依赖项

  ```shell
  conda create --name <ENV_NAME> <PACKAGE>=<VERSION>
  ```

  激活环境

  ```shell
  conda activate <ENV_NAME>
  ```

  查看所有的环境

  ```shell
  conda info --envs
  conda env list
  ```

- Conda目录

  - /pkgs：此目录包含已解压的软件包
  - /envs：创建的其他 conda 环境的系统位置

## Jupyter Notebook配置

​	安装Jupyter Notebook的前提是需要安装了Python（3.3版本及以上，或2.7版本），这说明当你使用conda创建新环境，Python版本不适配Jupyter时，在Annaconda控制台中下载新环境中的Jupyter是会失败的。常规来说，安装了Anaconda发行版时已经自动为你安装了Jupyter Notebook，我们通过annaconda navigator打开的Jupyter都是在系统家目录下。

- conda环境 --> Jupyter kernel

  当我们在conda中创建完新环境后想在Jupyter中使用时，必须将环境注册为Jupyter kernel，首先安装ipykernel，需要先激活目标环境再安装

  ipykernel的作用：

  1. 作为Jupyter Notebook和Python环境之间的桥梁
  1. 提供与Jupyter前端通信的协议实现
  1. 管理代码执行、变量状态和输出显示

  ```shell
  pip install ipykernel
  ```

  注册内核到Jupyter

  ```shell
  python -m ipykernel install --user --name [内核名] --display-name "[设置内核在 Jupyter 界面中显示的名称]"
  ```

  执行上述命令后可以到`C:\Users\[你的用户名]\AppData\Roaming\jupyter\kernels\[创建内核名]\`下看到kenrnel.json文件

  ```json
  {
   "argv": [
    "D:\\Anaconda\\envs\\chatgpt\\python.exe",// 当前Python解释器的路径
    "-m",
    "ipykernel_launcher",
    "-f",
    "{connection_file}"
   ],
   "display_name": "chatgpt",// 显示名称
   "language": "python",
   "metadata": {
    "debugger": true
   }
  }
  ```
  
  注册成功后新建Jupyter就可以选择我们新建的内核啦
  
  ![image-20250622154442263](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250622154442263.png)


- 选择Jupyter的服务器运行时文件和默认启动工作目录

  每次启动 Jupyter Notebook/Lab 时都会生成新的临时文件，如果没有修改服务器运行时文件的存储路径，那么可以在下面的路径中查看`C:/Users/17726/AppData/Roaming/jupyter/runtime`

  jpserver-<PID>.json（记录服务器信息，如端口、Token）

  jpserver-<PID>-open.html（用于浏览器自动跳转）

  那么如何修改呢？可以查看当前 Jupyter 配置路径

  ```bash
  jupyter --config-dir
  ```

  如果不存在则需要先生成默认配置，该配置文件会存储在`C:\Users\<你的用户名>\.jupyter\jupyter_notebook_config.py`

  ```bash
  jupyter notebook --generate-config
  ```

  生成后修改默认配置中的如下选项

  ```python
  c.ServerApp.runtime_dir = ''  # 替换为你想存储的路径
  ```

  定义 Jupyter Notebook 的默认启动目录（即打开浏览器时显示的根目录），固定工作目录，避免路径混乱

  修改 Jupyter 的配置文件 `jupyter_notebook_config.py`，添加默认启动目录

  ```python
  c.ServerApp.notebook_dir = ''  # 替换为你想存储的路径
  ```

  

​	
