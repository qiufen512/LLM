# Model I/O

管理大语言模型（Models）及其输入（Prompts）和格式化输出（Output Parsers）

数据流：Prompt ---> Model ---> Output Parser

## Prompts

提示模板负责将用户输入格式化为可传递给语言模型的格式

PromptTemplate是一个带标记的文本字符串，可以接受来自最终用户的一组参数并生成提示

- 基础模板

  ```python
  from langchain import PromptTemplate
  prompt_template = PromptTemplate.from_template(
      "Tell me a {adjective} joke about {content}."
  )
  prompt_template.format(adjective="sad",content="chickens")
  // 输出
  'Tell me a sad joke about chickens.'
  ```

- 分阶段的注入参数：Partial

  - 字符串的部分格式化

  ```python
  from langchain import PromptTemplate
  prompt_template = PromptTemplate.from_template(
      "Tell me a {adjective} joke about {content}."
  )
  partialPrompt = prompt_template.partial(adjective="sad")
  partialPrompt.format(content="chickens")
  ```

  ```python
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

## Lanuage Models

- LLM

- ChatModel

## Output Parsers

输出解析器负责将 LLM 的输出解析为更结构化的格式

# Data Connection

建设私域知识库的向量数据存储（Vetor Stores）、内容数据获取（Document Loaders）和转化（Transformers）以及向量数据查询（Retrievers）

数据流：Loaders ---> Transformers ---> Embeddings Model ---> Vector Stores ---> Retrievers

## Vector Stores

## Document Retrievers

## Document Loaders

## Document Transformers / Splitters

文本切分、代码切分、Token切分

# Memory

用于存储和获取对话历史记录的功能模块

# Chains

# Agents