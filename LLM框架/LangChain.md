# Model I/O

管理大语言模型（Models）及其输入（Prompts）和格式化输出（Output Parsers）

数据流：Prompt ---> Model ---> Output Parser

## Prompts

提示模板负责将用户输入格式化为可传递给语言模型的格式

### PromptTemplate

PromptTemplate是一个带标记的文本字符串，可以接受来自最终用户的一组参数并生成提示，主要与传统 LLM 配合使用（字符串输入输出）

- 基础模板：PromptTemplate.from_template

  ```python
  from langchain import PromptTemplate
  prompt = PromptTemplate.from_template(
      "请为以下主题写一篇{length}字的{style}文章：{topic}"
  )
  
  formatted_prompt = prompt.format(
      length="500",
      style="技术博客",
      topic="人工智能的发展趋势"
  )
  
  prompt.format(length="500",style="技术博客",topic="人工智能的发展趋势")
  ```

- 分阶段的注入参数：partial

  - 字符串的部分格式化

  ```python
  from langchain import PromptTemplate
  prompt_template = PromptTemplate.from_template(
      "请为以下主题写一篇{length}字的{style}文章：{topic}"
  )
  partialPrompt = prompt_template.partial(length="500",style="技术博客")
  partialPrompt.format(topic="人工智能的发展趋势")
  ```

  ```python
  from langchain import PromptTemplate
  prompt = PromptTemplate(
      template="{foo}{bar}", input_variables=["bar"], partial_variables={"foo": "foo"}
  )
  print(prompt.format(bar="baz"))
  ```
  
  - 带函数的部分格式化
  
  ```python
  from datetime import datetime
  
  def _get_datetime():
      now = datetime.now()
      return now.strftime("%m/%d/%Y, %H:%M:%S")
  
  prompt = PromptTemplate(
      template="Tell me a {adjective} joke about the day {date}",
      input_variables=["adjective", "date"],
  )
  partial_prompt = prompt.partial(date=_get_datetime)
  print(partial_prompt.format(adjective="funny"))
  ```

### ChatPromptTemplate

主要与 ChatModel 配合使用（消息列表输入输出）





## Lanuage Models

LLM：只支持纯文本输入；无法区分不同角色的消息；适合简单的文本生成任务

ChatModel：支持多种消息类型：1.SystemMessage ：系统指令，告诉AI要做什么；2.HumanMessage ：用户输入的消息；3.AIMessage ：AI回复的消息

### LLM

```python
from langchain_community.llms import Ollama

# 初始化 LLM
llm = Ollama(
    model="qwen3:1.7b",
    temperature=0.7,
)

llm.invoke("完成这条句子：今天的天气是")
```

### ChatModel

langchain与很多提供商集成，可以通过下面的网站查看每家提供商的python包，包名为：langchain-{provider}。同时可以查看每个提供商的python包的使用方式

[]: https://python.langchain.com/api_reference/index.html

这里我使用ollama作为聊天模型，langchain_community中已经集成了提供商，所以我在当前环境中下载langchain_community包

注：通过ollama作为聊天模型时需要在本地将ollama运行起来

这里我们安装langchain-community，集成了langchain-ollama

```shell
conda install -c conda-forge langchain-community
```

测试能否正常响应

```python
from langchain_community.chat_models import ChatOllama

chat_model = ChatOllama(model="")// 输入你在ollama中下载的模型
chat_model.invoke("你是谁")
```

## Output Parsers

输出解析器负责将 LLM 的输出解析为更结构化的格式

# Data Connection

建设私域知识库的向量数据存储（Vetor Stores）、内容数据获取（Document Loaders）和转化（Transformers）以及向量数据查询（Retrievers）

数据流：Loaders ---> Transformers ---> Embeddings Model ---> Vector Stores ---> Retrievers

## Vector Stores

## Document Retrievers

## Document Loaders

Document Loaders从文档加载数据并将数据转化成“文档”格式（一段文本和关联的元数据）



## Document Transformers / Splitters

文本切分、代码切分、Token切分

# Memory

用于存储和获取对话历史记录的功能模块

# Chains

# Agents