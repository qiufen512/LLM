

# LLM API配置

## OpenAI

- 获取 OpenAI API key 

  在该网站中获取OpenAI的API key：`https://platform.openai.com/settings/organization/api-keys`，首次获取需要创建

- 在你的项目文件根目录下创建.env文件

  ![image-20250622162538766](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250622162538766.png)

  文件中内容如下

  ```json
  OPENAI_API_KEY=""  # 替换为你的实际 API KEY
  ```

- 为当前环境安装需要用到的python库，首先激活当前环境后安装，推荐使用conda安装核心包

  ```shell
  conda install -c conda-forge openai python-dotenv langchain -y
  ```

  使用下面的的命令查看是否安装到当前环境中

  ```python
  import sys
  import openai, dotenv, langchain
  
  print("Python 路径:", sys.executable)
  print("openai 路径:", openai.__file__)
  print("dotenv 路径:", dotenv.__file__)
  print("langchain 路径:", langchain.__file__)
  ```

  如图所示则当前Python环境则已经安装OpenAI所需要的python库了

  ![image-20250622205131979](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250622205131979.png)

- 读取.env文件到当前环境中

  ```python
  import os
  from dotenv import load_dotenv, find_dotenv
  def get_openai_key():
      _ = load_dotenv(find_dotenv())
      return os.environ['OPENAI_API_KEY']
  openai.api_key = get_openai_key()
  print(f"已成功加载OpenAI API Key (前5位): {openai.api_key[:5]}******")
  ```

  如图所示则已正确加载

  ![image-20250622205600584](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250622205600584.png)

## 文心一言

- 登录千帆ModelBuilder控制台，点击用户账号 >安全认证，进入Access Key管理界面

  获取安全认证Access Key/Secret Key后在.env中填入

  ```json
  QIANFAN_ACCESS_KEY=""  # 替换为你的实际ACCESS KEY
  QIANFAN_SECRET_KEY=""  # 替换为你的实际SECRET_KEY
  ```

  也可以参考千帆大模型Python SDK的调用方式

  ```python
  import os
  os.environ["QIANFAN_ACCESS_KEY"] = "..."
  os.environ["QIANFAN_SECRET_KEY"] = "..."
  ```

- 为当前环境下载千帆的python库

  ```python
  pip install qianfan
  ```

- 通过下面代码验证是否调用成功

  ```python
  import os
  from dotenv import load_dotenv, find_dotenv
  import qianfan
  
  # 加载环境变量
  _ = load_dotenv(find_dotenv())
  
  # 设置 AK/SK
  qianfan.AK(os.getenv("QIANFAN_AK"))
  qianfan.SK(os.getenv("QIANFAN_SK"))
  
  def gen_wenxin_messages(prompt):
      messages = [{"role": "user", "content": prompt}]
      return messages
  
  def get_completion(prompt, model="ERNIE-Bot", temperature=0.01):
      chat_comp = qianfan.ChatCompletion()
      resp = chat_comp.do(
          messages=gen_wenxin_messages(prompt),
          model=model,
          temperature=temperature,
          system="你是一名个人助理-⼩鲸鱼"
      )
      return resp["result"]
  
  # 测试调用
  response = get_completion("你好，你是谁")
  print(response)
  ```

## 还有其他模型的调用，具体可以参考各大模型的官网~