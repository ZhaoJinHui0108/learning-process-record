# LangChain 学习笔记

## 目录
1. [LangChain 简介](#langchain-简介)
2. [LangChain 核心概念](#langchain-核心概念)
3. [LangChain 使用示例](#langchain-使用示例)

---

## LangChain 简介

### 什么是 LangChain
LangChain 是一个用于开发由语言模型驱动的应用程序的框架。它简化了构建 LLM 应用程序的过程，提供了模块化的组件和工具。

### LangChain 主要模块
| 模块 | 说明 |
|------|------|
| **Models** | 支持多种 LLM 模型 |
| **Prompts** | Prompt 模板管理 |
| **Indexes** | 文档加载和检索 |
| **Chains** | 链式调用 |
| **Agents** | 自主代理 |
| **Memory** | 对话记忆 |

---

## LangChain 核心概念

### 1. Models (模型)
LangChain 支持多种模型提供商：
- OpenAI (GPT-3.5, GPT-4)
- Anthropic (Claude)
- Hugging Face (开源模型)
- Azure OpenAI
- Google PaLM
- 等等

### 2. Prompts (提示模板)
```python
from langchain.prompts import PromptTemplate

template = PromptTemplate(
    input_variables=["product"],
    template="为以下产品写一个营销文案: {product}"
)
```

### 3. Chains (链)
```python
from langchain.chains import LLMChain

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run(product="iPhone")
```

### 4. Agents (代理)
Agent 使用 LLM 来决定执行什么操作。

### 5. Memory (记忆)
存储对话历史和上下文。

---

## LangChain 使用示例

### 示例1: 基础 LLM 调用

```python
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage, AIMessage

class SimpleLLMExample:
    """简单的 LLM 调用示例"""
    
    def __init__(self, api_key: str = None):
        self.llm = OpenAI(api_key=api_key, temperature=0.7)
        self.chat = ChatOpenAI(api_key=api_key, temperature=0.7)
    
    def basic_completion(self, prompt: str) -> str:
        """基础文本补全"""
        return self.llm(prompt)
    
    def chat_completion(self, user_message: str) -> str:
        """对话补全"""
        messages = [
            SystemMessage(content="你是一个有帮助的助手。"),
            HumanMessage(content=user_message)
        ]
        return self.chat(messages).content
    
    def conversational(self, history: list, new_message: str) -> str:
        """多轮对话"""
        messages = [SystemMessage(content="你是一个有帮助的助手。")]
        
        for role, content in history:
            if role == "user":
                messages.append(HumanMessage(content=content))
            else:
                messages.append(AIMessage(content=content))
        
        messages.append(HumanMessage(content=new_message))
        
        return self.chat(messages).content

def main():
    example = SimpleLLMExample()
    
    print("=== Basic Completion ===")
    result = example.basic_completion("给我讲一个关于机器人的笑话")
    print(f"Result: {result}\n")
    
    print("=== Chat Completion ===")
    result = example.chat_completion("Python是什么?")
    print(f"Result: {result}\n")
    
    print("=== Conversational ===")
    history = [
        ("user", "我叫小明"),
        ("assistant", "你好小明！有什么我可以帮助你的吗？")
    ]
    result = example.conversational(history, "我叫什么名字?")
    print(f"Result: {result}")

if __name__ == '__main__':
    main()
```

### 示例2: Prompt 模板

```python
from langchain.prompts import PromptTemplate, ChatPromptTemplate
from langchain.prompts.chat import SystemMessagePromptTemplate, HumanMessagePromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

class ProductInfo(BaseModel):
    name: str = Field(description="产品名称")
    price: float = Field(description="产品价格")
    features: list = Field(description="产品特点列表")

class PromptTemplateExample:
    """Prompt 模板示例"""
    
    def __init__(self, api_key: str = None):
        self.chat = ChatOpenAI(api_key=api_key, temperature=0.7)
    
    def simple_template(self):
        """简单模板"""
        template = "用{language}写一个Hello World程序"
        prompt = PromptTemplate(
            template=template,
            input_variables=["language"]
        )
        
        formatted = prompt.format(language="Python")
        print(f"Formatted Prompt: {formatted}")
        
        result = self.chat.predict(formatted)
        print(f"Result:\n{result}")
    
    def chat_template(self):
        """对话模板"""
        template = ChatPromptTemplate.from_messages([
            SystemMessagePromptTemplate.from_template("你是一个{profession}专家。"),
            HumanMessagePromptTemplate.from_template("{question}")
        ])
        
        messages = template.format_messages(
            profession="编程",
            question="什么是递归?"
        )
        
        result = self.chat(messages)
        print(f"Result: {result.content}")
    
    def structured_output(self):
        """结构化输出"""
        parser = PydanticOutputParser(pydantic_object=ProductInfo)
        
        template = PromptTemplate(
            template="根据以下描述提取产品信息:\n{product_description}\n{format_instructions}",
            input_variables=["product_description"],
            partial_variables={"format_instructions": parser.get_format_instructions()}
        )
        
        description = "iPhone 15 Pro Max是一款高端智能手机，价格999美元，具有强大的相机系统和长续航电池。"
        
        prompt = template.format(product_description=description)
        result = self.chat.predict(prompt)
        
        parsed = parser.parse(result)
        print(f"Parsed Result: {parsed}")

def main():
    example = PromptTemplateExample()
    
    print("=== Simple Template ===")
    example.simple_template()
    
    print("\n=== Chat Template ===")
    example.chat_template()

if __name__ == '__main__':
    main()
```

### 示例3: Chains (链式调用)

```python
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain, SimpleSequentialChain, SequentialChain
from langchain.chains.router import MultiPromptChain
from langchain.chains.router.llm_router import RouterOutputParser
from langchain.schema import HumanMessage

class ChainExample:
    """链式调用示例"""
    
    def __init__(self, api_key: str = None):
        self.llm = OpenAI(api_key=api_key, temperature=0.7)
        self.chat = ChatOpenAI(api_key=api_key, temperature=0.7)
    
    def simple_chain(self):
        """简单链"""
        prompt = PromptTemplate(
            input_variables=["product"],
            template="为以下产品写一句广告语: {product}"
        )
        
        chain = LLMChain(llm=self.llm, prompt=prompt)
        result = chain.run("智能手表")
        print(f"Advertising slogan: {result}")
    
    def sequential_chain(self):
        """顺序链"""
        prompt1 = PromptTemplate(
            input_variables=["topic"],
            template="用一句话描述{topic}: "
        )
        
        prompt2 = PromptTemplate(
            input_variables=["description"],
            template="基于以下描述写一首诗:\n{description}"
        )
        
        chain1 = LLMChain(llm=self.llm, prompt=prompt1, output_key="description")
        chain2 = LLMChain(llm=self.llm, prompt=prompt2, output_key="poem")
        
        sequential = SequentialChain(
            chains=[chain1, chain2],
            input_variables=["topic"],
            output_variables=["description", "poem"]
        )
        
        result = sequential({"topic": "爱情"})
        print(f"Description: {result['description']}")
        print(f"\nPoem:\n{result['poem']}")
    
    def parallel_chain(self):
        """并行链"""
        prompt1 = PromptTemplate(
            input_variables=["word"],
            template="解释单词'{word}'的意思:"
        )
        
        prompt2 = PromptTemplate(
            input_variables=["word"],
            template="用'{word}'造一个句子:"
        )
        
        chain1 = LLMChain(llm=self.llm, prompt=prompt1, output_key="definition")
        chain2 = LLMChain(llm=self.llm, prompt=prompt2, output_key="sentence")
        
        from langchain.chains import ParallelChain
        parallel = ParallelChain(chains=[chain1, chain2], input_variables=["word"])
        
        result = parallel({"word": "人工智能"})
        print(f"Definition: {result['definition']}")
        print(f"Sentence: {result['sentence']}")

def main():
    example = ChainExample()
    
    print("=== Simple Chain ===")
    example.simple_chain()
    
    print("\n=== Sequential Chain ===")
    example.sequential_chain()

if __name__ == '__main__':
    main()
```

### 示例4: Agents (代理)

```python
from langchain.agents import Agent, Tool
from langchain.agents.react.base import ReActDocstoreAgent
from langchain.docstore import Wikipedia
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.schema import AgentFinish, AgentAction

class ToolExample:
    """工具和代理示例"""
    
    def __init__(self, api_key: str = None):
        self.llm = OpenAI(api_key=api_key, temperature=0)
        self.tools = self._create_tools()
    
    def _create_tools(self):
        """创建工具"""
        def search_wikipedia(query: str) -> str:
            return f"关于'{query}'的信息: 这是一个维基百科风格的摘要..."
        
        def calculate(expression: str) -> str:
            try:
                result = eval(expression)
                return f"计算结果: {result}"
            except:
                return "计算错误"
        
        tools = [
            Tool(
                name="Wikipedia",
                func=search_wikipedia,
                description="当需要查找信息时使用此工具"
            ),
            Tool(
                name="Calculator",
                func=calculate,
                description="当需要进行数学计算时使用此工具"
            )
        ]
        
        return tools
    
    def react_agent(self):
        """ReAct 代理"""
        prompt = PromptTemplate(
            template="""使用以下工具回答问题:

{tools}

使用以下格式:

Question: 输入问题
Thought: 思考需要做什么
Action: 要使用的工具名称
Action Input: 工具的输入
Observation: 工具的输出
... (这个思考/动作/观察可以重复多次)
Thought: 我现在知道最终答案了
Final Answer: 最终答案

Question: {input}""",
            input_variables=["tools", "input"]
        )
        
        from langchain.agents import initialize_agent, AgentType
        
        agent = initialize_agent(
            self.tools,
            self.llm,
            agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
            verbose=True
        )
        
        result = agent.run("Python的创始人是谁?")
        print(f"Result: {result}")
    
    def conversational_agent(self):
        """对话代理"""
        from langchain.agents import initialize_agent, AgentType
        from langchain.memory import ConversationBufferMemory
        
        memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
        
        agent = initialize_agent(
            self.tools,
            self.llm,
            agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
            memory=memory,
            verbose=True
        )
        
        result = agent.run("你好!你叫什么名字?")
        print(f"Result: {result}")

def main():
    example = ToolExample()
    
    print("=== ReAct Agent ===")
    example.react_agent()

if __name__ == '__main__':
    main()
```

### 示例5: Memory (记忆)

```python
from langchain.memory import ConversationBufferMemory, ConversationSummaryMemory
from langchain.memory import ChatMessageHistory
from langchain.llms import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

class MemoryExample:
    """记忆示例"""
    
    def __init__(self, api_key: str = None):
        self.llm = OpenAI(api_key=api_key, temperature=0.7)
    
    def buffer_memory(self):
        """缓冲记忆"""
        memory = ConversationBufferMemory(memory_key="chat_history")
        
        chain = LLMChain(
            llm=self.llm,
            prompt=PromptTemplate(
                input_variables=["chat_history", "input"],
                template="""之前的对话:
{chat_history}

人类: {input}
AI: """
            ),
            memory=memory
        )
        
        response1 = chain.run("我叫小明")
        print(f"AI: {response1}")
        
        response2 = chain.run("我叫什么名字?")
        print(f"AI: {response2}")
        
        print(f"\nMemory contents: {memory.buffer}")
    
    def summary_memory(self):
        """摘要记忆 - 适合长对话"""
        memory = ConversationSummaryMemory(llm=self.llm)
        
        messages = [
            {"role": "user", "content": "我今天买了新电脑"},
            {"role": "assistant", "content": "恭喜你！是什么牌子的电脑?"},
            {"role": "user", "content": "是一台MacBook Pro"},
            {"role": "assistant", "content": "很好的选择！配置怎么样?"},
        ]
        
        for msg in messages:
            if msg["role"] == "user":
                memory.chat_memory.add_user_message(msg["content"])
            else:
                memory.chat_memory.add_ai_message(msg["content"])
        
        summary = memory.load_memory_variables({})
        print(f"Summary: {summary['history']}")
    
    def entity_memory(self):
        """实体记忆"""
        from langchain.memory.entity import EntityMemory
        from langchain.llms import OpenAI
        
        memory = EntityMemory(
            llm=self.llm,
            entity_store="memory_store"
        )
        
        memory.save_context(
            {"input": "我的朋友Tom是一个软件工程师"},
            {"output": "很高兴认识Tom！软件工程师是个很有趣的职业。"}
        )
        
        result = memory.load_memory_variables({"input": "Tom是做什么工作的?"})
        print(f"Retrieved: {result}")

def main():
    example = MemoryExample()
    
    print("=== Buffer Memory ===")
    example.buffer_memory()

if __name__ == '__main__':
    main()
```

### 示例6: Vector Store 和 Retrieval

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.text_splitter import CharacterTextSplitter
from langchain.document_loaders import TextLoader

class RetrievalExample:
    """向量存储和检索示例"""
    
    def __init__(self, api_key: str = None):
        self.embeddings = OpenAIEmbeddings(api_key=api_key)
    
    def create_vectorstore(self, documents: list):
        """创建向量存储"""
        text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
        
        texts = text_splitter.split_text("\n\n".join(documents))
        
        docsearch = FAISS.from_texts(texts, self.embeddings)
        
        return docsearch
    
    def similarity_search(self, docsearch, query: str, k: int = 3):
        """相似性搜索"""
        docs = docsearch.similarity_search(query, k=k)
        
        print(f"Query: {query}")
        print(f"Found {len(docs)} similar documents:\n")
        
        for i, doc in enumerate(docs, 1):
            print(f"[{i}] {doc.page_content[:200]}...")
            print()
    
    def similarity_search_with_score(self, docsearch, query: str):
        """带分数的相似性搜索"""
        docs_with_scores = docsearch.similarity_search_with_score(query)
        
        for doc, score in docs_with_scores:
            print(f"Score: {score:.4f}")
            print(f"Content: {doc.page_content[:200]}...")
            print()

def main():
    documents = [
        "Python是一种高级编程语言，由Guido van Rossum创建。",
        "JavaScript是一种用于Web开发的脚本语言。",
        "Java是一种面向对象的编程语言，最初由Sun Microsystems开发。",
        "机器学习是人工智能的一个分支，专注于开发能够学习的算法。",
        "深度学习是机器学习的一个子集，使用神经网络进行学习。"
    ]
    
    example = RetrievalExample()
    
    print("Creating vector store...")
    docsearch = example.create_vectorstore(documents)
    
    print("\n=== Similarity Search ===")
    example.similarity_search(docsearch, "什么是Python?")
    
    print("\n=== Search with Scores ===")
    example.similarity_search_with_score(docsearch, "编程语言有哪些？")

if __name__ == '__main__':
    main()
```
