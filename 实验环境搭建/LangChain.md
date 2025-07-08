# Model I/O

管理大语言模型（Models）及其输入（Prompts）和格式化输出（Output Parsers）

数据流：Prompt ---> Model ---> Output Parser

## Prompts

提示模板负责将用户输入格式化为可传递给语言模型的格式

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