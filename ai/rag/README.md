# RAG (Retrieval-Augmented Generation) 学习笔记

## 目录
1. [RAG 简介](#rag-简介)
2. [RAG 架构](#rag-架构)
3. [RAG 实现示例](#rag-实现示例)

---

## RAG 简介

### 什么是 RAG
RAG（检索增强生成）是一种结合了信息检索和语言模型的技术。它通过从外部知识库中检索相关信息来增强语言模型的生成能力。

### 为什么需要 RAG
- **知识时效性**: LLM 知识有截止日期，RAG 可以提供最新信息
- **减少幻觉**: 基于检索的事实生成更可靠
- **可解释性**: 检索到的文档提供了答案的来源
- **成本效益**: 比微调更经济
- **隐私性**: 敏感数据可以存储在本地

### RAG vs Fine-tuning
| 方面 | RAG | Fine-tuning |
|------|-----|-------------|
| 成本 | 低 | 高 |
| 更新知识 | 容易 | 需要重新训练 |
| 事实准确性 | 高 | 中等 |
| 定制风格 | 中等 | 高 |
| 需要标注数据 | 否 | 是 |

---

## RAG 架构

### 核心组件
```
┌─────────────────────────────────────────────────────────┐
│                      RAG System                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────────────────────────────────────────┐  │
│   │              Document Processing                 │  │
│   │  ┌─────────┐  ┌──────────┐  ┌───────────────┐  │  │
│   │  │Loading  │→ │Chunking  │→ │Embedding     │  │  │
│   │  └─────────┘  └──────────┘  └───────────────┘  │  │
│   └─────────────────────────────────────────────────┘  │
│                           │                              │
│                           ▼                              │
│   ┌─────────────────────────────────────────────────┐  │
│   │              Vector Store (VectorDB)             │  │
│   │            (存储向量和原始文档)                    │  │
│   └─────────────────────────────────────────────────┘  │
│                           │                              │
│                           ▼                              │
│   ┌─────────────────────────────────────────────────┐  │
│   │              Retrieval (检索)                     │  │
│   │            (相似度搜索)                           │  │
│   └─────────────────────────────────────────────────┘  │
│                           │                              │
│                           ▼                              │
│   ┌─────────────────────────────────────────────────┐  │
│   │              Generation (生成)                    │  │
│   │            (LLM 生成答案)                         │  │
│   └─────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 文档处理流程
1. **Document Loading**: 加载各种格式的文档
2. **Text Splitting**: 将文档分割成小块
3. **Embedding**: 将文本转换为向量
4. **Storage**: 存储到向量数据库

### 检索类型
| 类型 | 说明 | 适用场景 |
|------|------|----------|
| **Dense Retrieval** | 基于向量相似度 | 语义匹配 |
| **Sparse Retrieval** | 基于关键词匹配 | 精确查询 |
| **Hybrid** | 结合密集和稀疏 | 综合最佳 |
| **rerank** | 对结果重排序 | 提高精度 |

---

## RAG 实现示例

### 示例: 基础 RAG 系统

```python
import os
import re
import torch
from transformers import AutoTokenizer, AutoModel
import numpy as np
from collections import defaultdict

class SimpleVectorStore:
    def __init__(self):
        self.vectors = []
        self.documents = []
        self.metadata = []
    
    def add(self, document: str, vector: np.ndarray, metadata: dict = None):
        self.documents.append(document)
        self.vectors.append(vector)
        self.metadata.append(metadata or {})
    
    def search(self, query_vector: np.ndarray, top_k: int = 5) -> list:
        similarities = []
        for i, vec in enumerate(self.vectors):
            sim = self._cosine_similarity(query_vector, vec)
            similarities.append((i, sim))
        
        similarities.sort(key=lambda x: x[1], reverse=True)
        
        results = []
        for idx, score in similarities[:top_k]:
            results.append({
                'document': self.documents[idx],
                'score': score,
                'metadata': self.metadata[idx]
            })
        return results
    
    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        dot = np.dot(a, b)
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        return dot / (norm_a * norm_b + 1e-8)

class SimpleEmbedder:
    def __init__(self, model_name='bert-base-uncased'):
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModel.from_pretrained(model_name)
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        self.model.to(self.device)
    
    def embed(self, text: str) -> np.ndarray:
        inputs = self.tokenizer(
            text,
            return_tensors='pt',
            truncation=True,
            max_length=512,
            padding=True
        ).to(self.device)
        
        with torch.no_grad():
            outputs = self.model(**inputs)
        
        embedding = outputs.last_hidden_state[:, 0, :].cpu().numpy()[0]
        return embedding / np.linalg.norm(embedding)

class SimpleRAG:
    def __init__(self):
        self.embedder = SimpleEmbedder()
        self.vector_store = SimpleVectorStore()
    
    def load_documents(self, documents: list):
        for doc in documents:
            if isinstance(doc, dict):
                text = doc['content']
                metadata = {k: v for k, v in doc.items() if k != 'content'}
            else:
                text = doc
                metadata = {}
            
            chunks = self._chunk_text(text)
            
            for i, chunk in enumerate(chunks):
                vector = self.embedder.embed(chunk)
                chunk_metadata = {**metadata, 'chunk_id': i}
                self.vector_store.add(chunk, vector, chunk_metadata)
        
        print(f"Loaded {len(documents)} documents")
    
    def _chunk_text(self, text: str, chunk_size: int = 200, overlap: int = 50) -> list:
        sentences = re.split(r'[.!?]+', text)
        chunks = []
        current_chunk = ""
        
        for sentence in sentences:
            sentence = sentence.strip()
            if not sentence:
                continue
            
            if len(current_chunk) + len(sentence) < chunk_size:
                current_chunk += " " + sentence
            else:
                if current_chunk:
                    chunks.append(current_chunk.strip())
                current_chunk = sentence
        
        if current_chunk:
            chunks.append(current_chunk.strip())
        
        return chunks
    
    def retrieve(self, query: str, top_k: int = 3) -> list:
        query_vector = self.embedder.embed(query)
        results = self.vector_store.search(query_vector, top_k)
        return results

def generate_response(context: str, question: str) -> str:
    prompt = f"""Based on the following context, answer the question.

Context:
{context}

Question: {question}

Answer:"""
    
    return f"Based on the retrieved context, the answer would be generated by an LLM."

def main():
    documents = [
        {
            'content': '''Python is a high-level, interpreted programming language.
            It was created by Guido van Rossum and first released in 1991.
            Python emphasizes code readability with its notable use of significant whitespace.
            It supports multiple programming paradigms, including structured, procedural,
            and object-oriented programming.''',
            'source': 'python_wiki'
        },
        {
            'content': '''Machine learning is a subset of artificial intelligence (AI).
            It focuses on the development of computer programs that can access data
            and use it to learn for themselves. The learning process begins with
            observations or data, such as direct experience, or instruction,
            in order to look for patterns in data and make better decisions.''',
            'source': 'ml_basics'
        },
        {
            'content': '''Deep learning is part of a broader family of machine learning
            methods based on artificial neural networks with representation learning.
            Learning can be supervised, semi-supervised or unsupervised.
            Deep learning architectures such as deep neural networks have been applied
            to fields including computer vision, speech recognition, and NLP.''',
            'source': 'deep_learning'
        },
        {
            'content': '''Natural Language Processing (NLP) is a field of AI focused on
            the interaction between computers and humans through natural language.
            The ultimate goal of NLP is to enable computers to understand, interpret,
            and generate human language in a valuable way.''',
            'source': 'nlp_basics'
        }
    ]
    
    print("Initializing RAG system...")
    rag = SimpleRAG()
    
    print("\nLoading documents...")
    rag.load_documents(documents)
    
    queries = [
        "What is Python?",
        "Tell me about machine learning",
        "What is deep learning?"
    ]
    
    print("\n" + "="*60)
    print("RAG Query Examples")
    print("="*60)
    
    for query in queries:
        print(f"\nQuery: {query}")
        print("-" * 40)
        
        results = rag.retrieve(query, top_k=2)
        
        print(f"Retrieved {len(results)} relevant chunks:")
        for i, result in enumerate(results, 1):
            print(f"\n  [{i}] Score: {result['score']:.4f}")
            print(f"      Source: {result['metadata'].get('source', 'unknown')}")
            print(f"      Content: {result['document'][:150]}...")
        
        context = "\n\n".join([r['document'] for r in results])
        response = generate_response(context, query)
        print(f"\n  Generated Answer: {response}")

if __name__ == '__main__':
    main()
```

### RAG 优化技术

```python
class AdvancedRAG:
    """高级 RAG 技术集合"""
    
    def __init__(self):
        self.hybrid_store = HybridVectorStore()
        self.reranker = Reranker()
        self.query_expander = QueryExpander()
    
    def hybrid_retrieval(self, query: str, top_k: int = 10):
        """混合检索: 结合密集和稀疏检索"""
        dense_results = self.vector_store.search(query, top_k * 2)
        sparse_results = self.keyword_index.search(query, top_k * 2)
        
        combined = self._combine_results(dense_results, sparse_results)
        return combined
    
    def rerank_results(self, query: str, results: list, top_k: int = 5):
        """对检索结果进行重排序"""
        pairs = [(query, r['document']) for r in results]
        scores = self.reranker.score(pairs)
        
        reranked = sorted(zip(results, scores), key=lambda x: x[1], reverse=True)
        return [r[0] for r in reranked[:top_k]]
    
    def query_expansion(self, query: str) -> list:
        """查询扩展"""
        expanded = self.query_expander.expand(query)
        return expanded
    
    def iterative_retrieval(self, query: str, num_iterations: int = 3):
        """迭代检索"""
        current_query = query
        all_results = []
        
        for _ in range(num_iterations):
            results = self.retrieve(current_query, top_k=5)
            all_results.extend(results)
            
            feedback = self._generate_feedback(results, query)
            current_query = self._refine_query(current_query, feedback)
        
        return self._deduplicate(all_results)

class HybridVectorStore:
    """混合向量存储"""
    
    def __init__(self):
        self.dense_store = SimpleVectorStore()
        self.sparse_index = {}
    
    def add(self, doc_id: str, document: str, vector: np.ndarray):
        self.dense_store.add(doc_id, document, vector)
        
        terms = document.lower().split()
        for term in terms:
            if term not in self.sparse_index:
                self.sparse_index[term] = []
            self.sparse_index[term].append((doc_id, vector))
    
    def search(self, query: str, vector: np.ndarray, top_k: int = 5) -> list:
        return self.dense_store.search(vector, top_k)

class Reranker:
    """结果重排序"""
    
    def score(self, pairs: list) -> list:
        """使用交叉编码器对结果进行评分"""
        scores = []
        for query, document in pairs:
            score = self._compute_relevance(query, document)
            scores.append(score)
        return scores
    
    def _compute_relevance(self, query: str, document: str) -> float:
        query_terms = set(query.lower().split())
        doc_terms = set(document.lower().split())
        overlap = len(query_terms & doc_terms)
        return overlap / len(query_terms)

class QueryExpander:
    """查询扩展"""
    
    def __init__(self):
        self.synonyms = {
            'python': ['programming language', 'code'],
            'ml': ['machine learning', 'AI'],
            'dl': ['deep learning', 'neural networks']
        }
    
    def expand(self, query: str) -> list:
        expanded = [query]
        terms = query.lower().split()
        
        for term in terms:
            if term in self.synonyms:
                expanded.extend(self.synonyms[term])
        
        return list(set(expanded))
```

### RAG 评估指标

```python
class RAGEvaluator:
    """RAG 系统评估"""
    
    def evaluate_retrieval(self, retrieved_docs: list, relevant_docs: list) -> dict:
        """评估检索性能"""
        retrieved_ids = set([d['id'] for d in retrieved_docs])
        relevant_ids = set(relevant_docs)
        
        true_positives = len(retrieved_ids & relevant_ids)
        
        precision = true_positives / len(retrieved_ids) if retrieved_ids else 0
        recall = true_positives / len(relevant_ids) if relevant_ids else 0
        f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0
        
        return {
            'precision': precision,
            'recall': recall,
            'f1': f1
        }
    
    def evaluate_generation(self, generated: str, reference: str) -> dict:
        """评估生成质量"""
        generated_words = set(generated.lower().split())
        reference_words = set(reference.lower().split())
        
        overlap = len(generated_words & reference_words)
        
        precision = overlap / len(generated_words) if generated_words else 0
        recall = overlap / len(reference_words) if reference_words else 0
        f1 = 2 * precision * recall / (precision + recall) if (precision + recall) > 0 else 0
        
        return {
            'precision': precision,
            'recall': recall,
            'f1': f1
        }
    
    def evaluate_pipeline(self, query: str, rag_system: SimpleRAG, reference_docs: list) -> dict:
        """端到端评估"""
        retrieved = rag_system.retrieve(query)
        generated = generate_response('\n\n'.join([d['document'] for d in retrieved]), query)
        
        retrieval_metrics = self.evaluate_retrieval(retrieved, reference_docs)
        
        return {
            'retrieval': retrieval_metrics
        }
```
