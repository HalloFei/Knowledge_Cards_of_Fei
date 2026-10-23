---
tags:
  - Method
  - RAG
  - Generation
  - Knowledge
  - Base
  - Retrieval
---
```
{
  "term": "RAG",
  "full_form": "Retrieval-Augmented Generation",
  "folder": "Method",
  "definition": "检索增强生成，结合向量检索与语言模型生成，提升事实性和知识密度。",
  "tags": ["retrieval", "generation", "knowledge-base"],
  "storage_path": "LLM in general/Terms/Method/检索增强生成 Retrieval-Augmented Generation RAG.md"
}
```
# 0. Retrieval-Augmented Generation 

## 0.1 定义

Retrieval-Augmented Generation, also known as RAG, is a technology that combines retrieval systems with large language models to enhance the accuracy and relevance of generated answers by first retrieving relevant information from a knowledge base and then using that information to generate a response. This approach helps mitigate hallucinations and ensures that the output is grounded in reliable, up-to-date knowledge.


# N. 知识库 Knowledge Base


CRUD = Create+Read+Update+Delete 
Create includes：
- Source of Creation 
	1. P0 文件上传
	2. P1 LLM联网生成
	3. P0 外部链接调用
	4. P1 外部提交
- Creation with additional Process 
	1. 处理知识分块
		1. 保留独立语义
	2. 生成ID+摘要+正文
	3. 存储
		1. 源文件
		2. 摘要+正文
		3. 向量值


# 2. RAG 

## 2.1 Retrieval 混合检索

