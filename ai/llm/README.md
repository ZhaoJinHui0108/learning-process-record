# LLM (Large Language Models) 学习笔记

## 目录
1. [LLM 简介](#llm-简介)
2. [主流 LLM](#主流-llm)
3. [LLM 调用示例](#llm-调用示例)
4. [LLM 应用场景](#llm-应用场景)

---

## LLM 简介

### 什么是 LLM
大型语言模型（Large Language Models，LLM）是通过大规模预训练学习语言规律的深度学习模型，能够理解和生成人类语言。

### LLM 的特点
- **大规模参数**: 通常数十亿到数千亿参数
- **预训练 + 微调**: 先在大规模数据上预训练，再在特定任务上微调
- **涌现能力**: 规模增大后出现意想不到的能力
- **多任务能力**: 一个模型处理多种任务
- **上下文学习**: Few-shot learning

### Transformer 架构
LLM 主要基于 Transformer 架构：
- Encoder-only (如 BERT)
- Decoder-only (如 GPT)
- Encoder-Decoder (如 T5, BART)

---

## 主流 LLM

### OpenAI 系列
| 模型 | 参数量 | 特点 | 适用场景 |
|------|--------|------|----------|
| GPT-4 | ~1.5万亿 | 最强推理能力 | 复杂任务 |
| GPT-4 Turbo | ~1万亿 | 更快、更便宜 | 生产环境 |
| GPT-3.5 Turbo | ~1750亿 | 性价比高 | 日常对话 |
| GPT-4o | 多模态 | 实时对话 | 多模态交互 |

### 开源模型
| 模型 | 开发者 | 参数量 | 特点 |
|------|--------|--------|------|
| Llama 3 | Meta | 8B-70B | 开源领先 |
| Mistral | Mistral AI | 7B-70B | 高效率 |
| Qwen | 阿里 | 7B-110B | 中文优秀 |
| GLM | 智谱 | 130B | 中文优化 |
| Baichuan | 百川 | 7B-13B | 中文友好 |

### 国内模型
| 模型 | 开发公司 | 特点 |
|------|----------|------|
| GPT-4o-mini | OpenAI | 轻量高效 |
| Claude 3.5 | Anthropic | 安全可靠 |
| Gemini | Google | 多模态强 |
| 文心一言 | 百度 | 中文增强 |
| 通义千问 | 阿里 | 开源可用 |
| Kimi | 月之暗面 | 长上下文 |

---

## LLM 调用示例

### 示例1: OpenAI API 调用

```python
import openai

class OpenAIClient:
    """OpenAI API 调用封装"""
    
    def __init__(self, api_key: str = None):
        self.client = openai.OpenAI(api_key=api_key)
    
    def chat_completion(
        self,
        messages: list,
        model: str = "gpt-3.5-turbo",
        temperature: float = 0.7,
        max_tokens: int = 1000
    ) -> str:
        """对话补全"""
        response = self.client.chat.completions.create(
            model=model,
            messages=messages,
            temperature=temperature,
            max_tokens=max_tokens
        )
        return response.choices[0].message.content
    
    def simple_chat(self, prompt: str, system: str = None) -> str:
        """简单对话"""
        messages = []
        
        if system:
            messages.append({"role": "system", "content": system})
        
        messages.append({"role": "user", "content": prompt})
        
        return self.chat_completion(messages)
    
    def stream_chat(self, messages: list):
        """流式对话"""
        stream = self.client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages,
            stream=True
        )
        
        for chunk in stream:
            if chunk.choices[0].delta.content:
                print(chunk.choices[0].delta.content, end="")

def main():
    import os
    
    api_key = os.environ.get("OPENAI_API_KEY")
    client = OpenAIClient(api_key=api_key)
    
    print("=== Simple Chat ===")
    response = client.simple_chat(
        prompt="用一句话解释什么是机器学习",
        system="你是一个AI助手，用简洁的语言回答问题"
    )
    print(f"Response: {response}\n")
    
    print("=== Stream Chat ===")
    print("Output: ")
    messages = [{"role": "user", "content": "给我讲一个关于人工智能的睡前故事"}]
    client.stream_chat(messages)
    print("\n")

if __name__ == '__main__':
    main()
```

### 示例2: 本地模型调用 (Hugging Face)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

class HuggingFaceClient:
    """Hugging Face 本地模型调用"""
    
    def __init__(self, model_name: str = "microsoft/phi-2"):
        self.model_name = model_name
        self.tokenizer = AutoTokenizer.from_pretrained(
            model_name,
            trust_remote_code=True
        )
        self.model = AutoModelForCausalLM.from_pretrained(
            model_name,
            device_map="auto",
            torch_dtype=torch.float32,
            trust_remote_code=True
        )
    
    def generate(
        self,
        prompt: str,
        max_length: int = 200,
        temperature: float = 0.7,
        top_p: float = 0.9
    ) -> str:
        """文本生成"""
        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)
        
        outputs = self.model.generate(
            **inputs,
            max_length=max_length,
            temperature=temperature,
            top_p=top_p,
            do_sample=True
        )
        
        return self.tokenizer.decode(outputs[0], skip_special_tokens=True)
    
    def chat(self, prompt: str) -> str:
        """对话格式"""
        formatted_prompt = f"User: {prompt}\nAssistant:"
        response = self.generate(formatted_prompt)
        
        response = response.split("Assistant:")[-1].strip()
        return response

def main():
    print("Loading model... (This may take a while on first run)")
    
    try:
        client = HuggingFaceClient("microsoft/phi-2")
        
        print("=== Text Generation ===")
        prompt = "Python is a high-level programming language that"
        result = client.generate(prompt, max_length=100)
        print(f"Prompt: {prompt}")
        print(f"Generated: {result}\n")
        
        print("=== Chat ===")
        response = client.chat("What is the capital of France?")
        print(f"Response: {response}")
    
    except Exception as e:
        print(f"Error: {e}")
        print("Make sure you have enough GPU memory or use a smaller model")

if __name__ == '__main__':
    main()
```

### 示例3: LangChain + OpenAI

```python
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage
from langchain.prompts import PromptTemplate, ChatPromptTemplate
from langchain.chains import LLMChain

class LangChainLLMExample:
    """LangChain + LLM 示例"""
    
    def __init__(self, api_key: str = None):
        self.llm = OpenAI(api_key=api_key, temperature=0.7)
        self.chat = ChatOpenAI(api_key=api_key, temperature=0.7)
    
    def basic_completion(self):
        """基础补全"""
        result = self.llm("Write a haiku about programming:")
        print(f"Haiku: {result}\n")
    
    def chat_completion(self):
        """对话补全"""
        messages = [
            SystemMessage(content="你是一个创意写作助手。"),
            HumanMessage(content="给我写一个关于AI的科幻短故事的开头")
        ]
        
        result = self.chat(messages)
        print(f"Story: {result.content}\n")
    
    def chain_example(self):
        """链式调用"""
        template = """你是一个产品营销专家。
请为以下产品写一段营销文案:

产品名称: {product_name}
产品特点: {features}
目标用户: {target_audience}

营销文案:"""
        
        prompt = PromptTemplate(
            template=template,
            input_variables=["product_name", "features", "target_audience"]
        )
        
        chain = LLMChain(llm=self.llm, prompt=prompt)
        
        result = chain.run({
            "product_name": "智能手表",
            "features": "心率监测、睡眠追踪、NFC支付、防水",
            "target_audience": "追求健康生活的都市白领"
        })
        
        print(f"Marketing Copy: {result}\n")
    
    def conversational_chain(self):
        """对话链"""
        from langchain.memory import ConversationBufferMemory
        from langchain.chains import ConversationChain
        
        memory = ConversationBufferMemory()
        conversation = ConversationChain(
            llm=self.chat,
            memory=memory,
            verbose=True
        )
        
        response1 = conversation.predict(input="我叫小明，是一名软件工程师")
        print(f"AI: {response1}")
        
        response2 = conversation.predict(input="我喜欢机器学习和人工智能")
        print(f"AI: {response2}")
        
        response3 = conversation.predict(input="我叫什么名字?")
        print(f"AI: {response3}")

def main():
    import os
    
    api_key = os.environ.get("OPENAI_API_KEY")
    
    if not api_key:
        print("Please set OPENAI_API_KEY environment variable")
        return
    
    example = LangChainLLMExample(api_key=api_key)
    
    print("=== Basic Completion ===")
    example.basic_completion()
    
    print("=== Chain Example ===")
    example.chain_example()

if __name__ == '__main__':
    main()
```

### 示例4: 多模态模型调用

```python
import base64
from pathlib import Path

class MultimodalLLMExample:
    """多模态 LLM 示例"""
    
    def __init__(self, api_key: str = None):
        from openai import OpenAI
        self.client = OpenAI(api_key=api_key)
    
    def image_analysis(self, image_path: str, prompt: str = None) -> str:
        """图像分析"""
        if prompt is None:
            prompt = "请描述这张图片的内容"
        
        def encode_image(image_path):
            with open(image_path, "rb") as image_file:
                return base64.b64encode(image_file.read()).decode('utf-8')
        
        image_data = encode_image(image_path)
        
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": prompt},
                        {
                            "type": "image_url",
                            "image_url": {
                                "url": f"data:image/jpeg;base64,{image_data}"
                            }
                        }
                    ]
                }
            ],
            max_tokens=500
        )
        
        return response.choices[0].message.content
    
    def image_generation(self, prompt: str) -> str:
        """图像生成"""
        response = self.client.images.generate(
            model="dall-e-3",
            prompt=prompt,
            size="1024x1024",
            n=1
        )
        
        return response.data[0].url
    
    def text_to_speech(self, text: str, voice: str = "alloy") -> bytes:
        """文本转语音"""
        response = self.client.audio.speech.create(
            model="tts-1",
            voice=voice,
            input=text
        )
        
        return response.content

def main():
    import os
    
    api_key = os.environ.get("OPENAI_API_KEY")
    
    if not api_key:
        print("Please set OPENAI_API_KEY environment variable")
        return
    
    example = MultimodalLLMExample(api_key=api_key)
    
    print("=== Image Generation ===")
    url = example.image_generation(
        "A cute cat sitting on a windowsill at sunset"
    )
    print(f"Generated image URL: {url}\n")
    
    print("=== Image Analysis ===")
    print("Note: Provide a real image path to analyze")

if __name__ == '__main__':
    main()
```

---

## LLM 应用场景

### 1. 文本生成
```python
# 代码生成
prompt = """请用Python写一个快速排序算法的实现:
"""

# 创意写作
prompt = """写一首关于人工智能的现代诗:
"""

# 商业写作
prompt = """为一封辞职信写一个专业模板:
"""
```

### 2. 问答系统
```python
# 知识问答
prompt = """基于以下上下文回答问题:
上下文: Python是一种高级编程语言...
问题: Python是什么时候创建的?
"""

# 推理问答
prompt = """推理以下问题:
问题: 张三比李四大2岁，李四比王五小3岁。王五25岁。张三多大?
"""
```

### 3. 文本分类
```python
prompt = """对以下文本进行情感分类:

文本: 这个产品太棒了，完全超出我的预期！

分类选项: 正面、负面、中性

输出格式: 分类: [结果]
置信度: [百分比]
"""

# 多标签分类
prompt = """对以下新闻进行分类（可多选）:

新闻: 苹果公司发布新款iPhone

类别: 科技、商业、娱乐、社会

输出格式:
主要类别: [主要类别]
次要类别: [次要类别]
"""
```

### 4. 文本摘要
```python
prompt = """请为以下文章写一个100字的摘要:

文章内容: [长篇文章...]

要求:
- 突出核心观点
- 语言简洁
- 不超过100字

摘要:"""
```

### 5. 翻译
```python
prompt = """将以下中文翻译成英文，保持原文风格:

中文: 春风又绿江南岸，明月何时照我还

翻译:"""

# 专业翻译
prompt = """请将以下医学文本翻译成中文，注意专业术语:

English: The patient presented with symptoms of acute respiratory distress syndrome...

翻译:"""
```

### 6. 代码相关
```python
# 代码解释
prompt = """解释以下Python代码的功能:

def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)

解释:"""

# 代码调试
prompt = """找出以下代码中的bug并修复:

def find_duplicate(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return num
        seen.add(num)
    return None

问题: 有时候会返回None但实际上存在重复
"""

# 代码转换
prompt = """将以下Java代码转换为Python:

public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

Python版本:
"""
```

### 7. 数据处理
```python
# 实体提取
prompt = """从以下文本中提取实体:

文本: 微软公司CEO Satya Nadella于2014年在西雅图发布了Windows 10。

提取以下信息:
- 公司名:
- 人名:
- 产品名:
- 地点:
- 时间:
"""

# 关系抽取
prompt = """分析以下文本中实体之间的关系:

文本: 马斯克是特斯拉公司的CEO，同时也是SpaceX的创始人。

关系:
- [实体1] 和 [实体2] 之间的关系
"""
```

### 8. 对话系统
```python
# 客服机器人
system_prompt = """你是一个在线客服机器人。
- 用友好、专业的语气回答
- 如果无法回答，转人工
- 不要提供虚假信息"""

# 个人助手
system_prompt = """你是一个个人助理，可以帮助用户:
- 回答问题
- 安排日程
- 提供建议
- 撰写文档
请始终保持耐心和专业。"""
```
