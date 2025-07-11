

# Model I/O

管理大语言模型（Models）及其输入（Prompts）和格式化输出（Output Parsers）

数据流：Prompt ---> Model ---> Output Parser

## Prompts

提示模板负责将用户输入格式化为可传递给语言模型的格式

langchain官方文档中Prompt的类层次结构

BasePromptTemplate --> PipelinePromptTemplate
                       					  StringPromptTemplate --> PromptTemplate
                                                											FewShotPromptTemplate
                                                											FewShotPromptWithTemplates
                       					  BaseChatPromptTemplate --> AutoGPTPrompt
                                                  												ChatPromptTemplate --> AgentScratchPadChatPromptTemplate

BaseMessagePromptTemplate --> MessagesPlaceholder
                              								BaseStringMessagePromptTemplate --> ChatMessagePromptTemplate
                                                                  																	HumanMessagePromptTemplate
                                                                  																	AIMessagePromptTemplate
                                                                  																	SystemMessagePromptTemplate

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

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_ollama import ChatOllama  # 或其他聊天模型

template = ChatPromptTemplate([
    ("system", "你是一位{role}，回复信息时语气偏向{style}。"),
    ("human", "介绍关于{topic}的一个核心知识点。")
])

model = ChatOllama(model="qwen3:1.7b", temperature=0.7)

chain = template | model

response = chain.invoke({
    "role": "技术专家",
    "style": "专业但易懂",
    "topic": "深度学习"
})
```

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

langchain与很多LLM提供商集成，一般的包名为：langchain-{provider}。这里我使用ollama部署qwen3作为聊天模型，langchain_community中已经集成了提供商，所以我在当前环境中下载langchain_community包

注：通过ollama作为聊天模型时需要先在本地将ollama运行起来

这里我们安装langchain-community，集成了langchain-ollama

```shell
conda install -c conda-forge langchain-community
```

如果能正常输出的话就说明已经连接成功了

```python
from langchain_community.chat_models import ChatOllama

chat_model = ChatOllama(model="")// 输入你在ollama中下载的模型
chat_model.invoke("你是谁")
```

#### 如何执行函数/工具调用

为什么要使用工具调用呢？

1. 可以标准化接口，因为langchain集成很多LLM提供商，原生的LLM都引入了工具调用能力，但是采取不同的格式和约定。1. LangChain通过统一的.bind_tools() 方法方便开发者在不同的提供商之间切换，不需要修改代码逻辑
2. 利用工具调用可以使得LLM访问、交互和操作外部资源
3. 可以返回结构化的工具调用信息，LangChain还处理了工具调用可能出现的错误，如无效的JSON参数

注：有些聊天模型可能不支持工具调用，具体可以查看官方文档

- 多种工具定义

  1. Python函数
  2. Pydantic 类
  3. TypedDict 类

  这里只给出Python函数和Pydantic类的示例，具体可以查看LangChain的官方文档，主要是用到了bind_tools这个方法

  ```python
  from langchain_ollama.chat_models import ChatOllama
  def add(a: int, b: int) -> int:
      return a + b
  
  def multiply(a: int, b: int) -> int:
      return a * b
  
  tools = [add,multiply]
  llm = ChatOllama(
      model="qwen3:1.7b",
      temperature=0.7,
  )
  llm_with_tools  = llm.bind_tools(tools)
  query = "3 * 12等于多少？"
  llm_with_tools.invoke(query)
  ```

  ```python
  from pydantic import BaseModel, Field
  from langchain_ollama.chat_models import ChatOllama
  
  class add(BaseModel):
      a: int = Field(..., description="第一个数")
      b: int = Field(..., description="第二个数")
  
  class multiply(BaseModel):
      a: int = Field(..., description="第一个数")
      b: int = Field(..., description="第二个数")
      
  tools = [add, multiply]
  llm = ChatOllama(
      model="qwen3:1.7b",
      temperature=0.7,
  )
  llm_with_tools  = llm.bind_tools(tools)
  query = "3 * 12等于多少？"
  llm_with_tools.invoke(query)
  ```

  使用 bind_tools 绑定工具后，ChatModel 在 invoke 时会获得对可用工具的感知 。模型会接收到工具的定义信息，包括工具名称、描述和参数架构，这些信息会在每次调用时传递给模型

  ![image-20250710184624108](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250710184624108.png)

  llm_with_tools.invoke(query).tool_calls会返回工具名称、参数值字典以及（可选的）标识符的带类型的字典，如果没有工具调用的信息则该列表返回为空

  ```json
  tool_calls=[
      {'name': 'multiply', 
       'args': {'a': 3, 'b': 12}, 
       'id': 'bb1a50cc-039a-4d28-b793-71705ab91b43', 
       'type': 'tool_call'}
  ]
  ```

  可以用tool_choice强制LLM选择我们特定的工具

  ```python
  llm_with_tools  = llm.bind_tools(tools,,tool_choice="add")
  ```

#### 让模型返回结构化输出

结构化的输出用于下游系统

- 几种从模型获取结构化输出的策略

  1. with_structured_output
  2. Few-shot prompting 
  3. 其他的可以查阅官方文档

  ```python
  from langchain_ollama.chat_models import ChatOllama
  from langchain_core.tools import tool
  from pydantic import BaseModel, Field
  from typing import Optional
  from pydantic import BaseModel, Field
  
  llm = ChatOllama(
      model="qwen3:1.7b",
      temperature=0.7,
  )
  
  json_schema = {
      "title": "joke",
      "description": "Joke to tell user.",
      "type": "object",
      "properties": {
          "setup": {
              "type": "string",
              "description": "The setup of the joke",
          },
          "punchline": {
              "type": "string",
              "description": "The punchline to the joke",
          },
          "rating": {
              "type": "integer",
              "description": "How funny the joke is, from 1 to 10",
          },
      },
      "required": ["setup", "punchline","rating"],
  }
  structured_llm = llm.with_structured_output(json_schema)
  structured_llm.invoke("Tell me a joke about cats")
  ```

#### 流式输出

```python
from langchain_ollama.chat_models import ChatOllama
from pydantic import BaseModel, Field

llm = ChatOllama(
    model="qwen3:1.7b",
    temperature=0.7,
)
for chunk in llm.stream("Write me a 1 verse song about goldfish on the moon"):
    print(chunk.content, end="|", flush=True)
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

可以将大语言模型、提示、输出解析器等多个组件串联起来，形成一个完整的处理流水线

- 为了可以看到链路的全过程，我们可以顺便接入LangSmith

  首先下载LangSmith的安装包

  ```python
  pip install -U langsmith
  ```

  在这个网站申请API_KEY: `https://smith.langchain.com/settings`，记得创建完复制下来，没有的话无法查看API KEY详情

  ![image-20250711200633781](C:\Users\17726\AppData\Roaming\Typora\typora-user-images\image-20250711200633781.png)

  

- LLMChain 最基础的链

  ```python
  from langchain_ollama import ChatOllama
  from langchain import PromptTemplate
  from langchain.chains import LLMChain
  
  prompt = PromptTemplate(
      input_variables=["product"],
      template="为{product}写一个创意广告语"
  )
  
  llm = ChatOllama(model= "qwen3:1.7b" ,temperature=0.7)
  
  chain = LLMChain(llm=llm, prompt=prompt)
  
  result = chain.invoke("智能手机")
  print(result)
  ```

- SimpleSequentialChain 简单顺序链

  上一个的输出可以作为下一个的输入

  ```python
  from langchain_ollama import ChatOllama
  from langchain import PromptTemplate
  from langchain.chains import LLMChain
  from langchain.chains import SimpleSequentialChain
  
  llm = ChatOllama(model= "qwen3:1.7b" ,temperature=0.7)
  synopsis_template = """"
  你是一位编剧。给定一个剧本标题，你的工作是为该标题写一个概要。
  标题：{title}
  编剧：这是上述剧本的概要：
  """
  synopsis_prompt = PromptTemplate.from_template(synopsis_template)
  synopsis_chain = LLMChain(llm=llm, prompt=synopsis_prompt)
  
  review_template = """"
  你是《纽约时报》的剧本评论家。给定一个剧本的概要，你的工作是为该剧本写一篇评论。
  剧本概要：{synopsis}
  《纽约时报》评论家对上述剧本的评论：
  """
  review_prompt = PromptTemplate.from_template(review_template)
  review_chain = LLMChain(llm=llm, prompt=review_prompt)
  
  overall_chain = SimpleSequentialChain(
      chains=[synopsis_chain, review_chain],
      verbose=True
  )
  
  result = overall_chain.invoke("悲惨世界")
  print(result)
  ```

  使用LCEL高效简洁的链接 LangChain 组件

  ```python
  from langchain_ollama import ChatOllama
  from langchain_core.prompts import ChatPromptTemplate
  from langchain_core.output_parsers import StrOutputParser
  from langchain_core.runnables import RunnablePassthrough
  
  llm = ChatOllama(model="qwen3:1.7b", temperature=0.7)
  output_parser = StrOutputParser()
  
  synopsis_prompt = ChatPromptTemplate.from_template("""
  你是一位编剧。给定一个剧本标题，你的工作是为该标题写一个概要。
  标题：{title}
  编剧：这是上述剧本的概要：
  """)
  
  review_prompt = ChatPromptTemplate.from_template("""
  你是《纽约时报》的剧本评论家。给定一个剧本的概要，你的工作是为该剧本写一篇评论。
  剧本概要：{synopsis}
  《纽约时报》评论家对上述剧本的评论：
  """)
  
  synopsis_chain = synopsis_prompt | llm | output_parser
  review_chain = review_prompt | llm | output_parser
  
  overall_chain = (
      synopsis_chain
      | (lambda synopsis: {"synopsis": synopsis})
      | review_chain
  )
  
  result = overall_chain.invoke({"title": "悲惨世界"})
  print(result)
  ```

其他的链路可以查看官方文档，这里不赘述



# Agents