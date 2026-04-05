# Prompt Engineering 学习笔记

## 目录
1. [Prompt 基础](#prompt-基础)
2. [Prompt 技巧](#prompt-技巧)
3. [高级 Prompt 技术](#高级-prompt-技术)
4. [Prompt 模板](#prompt-模板)

---

## Prompt 基础

### 什么是 Prompt
Prompt（提示）是与 LLM 交互的方式，通过自然语言描述来指导模型生成所需的输出。

### Prompt 的基本组成
```python
"""
1. 指令 (Instruction): 告诉模型要做什么
2. 上下文 (Context): 提供背景信息
3. 输入数据 (Input Data): 需要处理的内容
4. 输出格式 (Output Indicator): 指定输出的格式
"""
```

### 基本 Prompt 示例

```python
# 简单指令
prompt1 = "把以下句子翻译成英文: 今天天气真好"

# 带上下文的指令
prompt2 = """作为一位专业的翻译者，请将以下中文文本翻译成英文。
只输出翻译结果，不要解释。

中文文本: 今天天气真好"""

# 带格式要求的指令
prompt3 = """请用JSON格式输出以下信息:
{
  "name": 姓名,
  "age": 年龄,
  "city": 城市
}

输入: 张三，28岁，北京
"""
```

---

## Prompt 技巧

### 1. 清晰具体的指令
```python
# 不好的 Prompt
prompt_bad = "写关于狗的事"

# 好的 Prompt
prompt_good = """请写一篇300字的文章，介绍金毛寻回犬的特点、饲养注意事项和适合的家庭环境。
文章应该包括:
1. 品种特点
2. 日常护理
3. 运动需求
4. 适合人群
"""
```

### 2. 使用分隔符
```python
prompt = """请总结以下文章的主要内容。

===
{article_content}
===

总结应该包含:
- 核心观点
- 关键数据或事实
- 结论
"""
```

### 3. 指定输出格式
```python
# JSON格式
prompt_json = """提取以下文本中的关键信息:

输入: "苹果公司成立于1976年，总部位于加州库比蒂诺，CEO是蒂姆·库克。"

输出格式:
{
  "公司名称": "",
  "成立年份": "",
  "总部": "",
  "CEO": ""
}
"""

# 列表格式
prompt_list = """将以下水果按颜色分类:

水果: 苹果、香蕉、蓝莓、橙子、草莓、葡萄

按以下JSON格式输出:
{
  "红色": [],
  "黄色": [],
  "蓝色/紫色": [],
  "橙色": []
}
"""

# Markdown格式
prompt_md = """写一个技术博客的大纲，主题是"如何在Python中实现REST API"。
使用以下Markdown格式:

# 标题

## 概述
[内容]

## 主要内容
### 小节1
### 小节2

## 总结
"""
```

### 4. Few-shot Learning (少样本学习)
```python
prompt_fewshot = """根据例子的格式，生成新的句子。

例子1:
输入: 今天天气很好
输出: 阳光明媚，心情也跟着明朗起来

例子2:
输入: 工作压力很大
输出: 任务堆积如山，让人喘不过气来

请按照上述格式生成:
输入: 学习新技能
输出:"""
```

### 5. 思维链 (Chain of Thought)
```python
prompt_cot = """请逐步分析以下问题:

问题: 如果一个商店有50个苹果，卖掉了20个，又进货30个，现在有多少个苹果?

请按以下步骤思考:
1. 初始苹果数量
2. 卖掉后的数量
3. 进货后的数量
4. 最终答案

逐步计算:"""
```

### 6. 指定角色
```python
prompt_role = """你是一位拥有20年经验的高级软件架构师。
请用专业的视角分析以下系统设计问题:

问题: 设计一个日活跃用户1000万的社交App的后端架构

请从以下方面分析:
1. 整体架构设计
2. 数据库选型
3. 缓存策略
4. 消息队列
5. 微服务划分
"""
```

---

## 高级 Prompt 技术

### 1. 系统 Prompt 和用户 Prompt
```python
system_prompt = """你是一个专业的代码审查助手。
- 只关注代码质量和性能问题
- 提供具体的改进建议
- 用代码示例说明改进方法"""

user_prompt = """请审查以下Python代码:

def calculate(numbers):
    result = 0
    for i in range(len(numbers)):
        result = result + numbers[i]
    return result
"""
```

### 2. 条件 Prompt
```python
prompt_conditional = """根据用户输入的类型，执行相应的操作:

输入类型检测:
- 如果是"翻译": 翻译成英文
- 如果是"摘要": 提供简短摘要
- 如果是"代码": 解释代码功能

输入: {user_input}

首先判断类型，然后执行对应操作。"""
```

### 3. 迭代优化 Prompt
```python
prompt_iterative = """作为一个AI助手，我会根据用户的反馈不断改进回答。

请用以下格式回复:

初始回答:
[你的回答]

自我评估:
[检查回答是否满足要求]

如需要改进:
[改进后的版本]

---

请用以上格式回答: 解释什么是机器学习"""
```

### 4. 约束条件
```python
prompt_constraint = """请在遵守以下约束的条件下回答问题:

约束:
- 回答必须在100字以内
- 禁止使用专业术语
- 必须举例说明
- 用通俗易懂的语言

问题: 什么是HTTP协议?"""
```

### 5. 多步骤任务拆分
```python
prompt_multistep = """完成以下复杂任务，分步骤进行:

任务: 帮用户规划一次北京三日游

步骤1 - 收集信息:
- 询问用户的偏好（文化/美食/景点）
- 了解预算范围
- 确认出行时间

步骤2 - 制定计划:
- 根据收集的信息安排每日行程
- 推荐必去景点
- 推荐当地美食

步骤3 - 费用估算:
- 估算交通费用
- 估算门票费用
- 估算餐饮费用

请先执行步骤1，询问用户相关信息。"""
```

### 6. 上下文注入
```python
prompt_context = """基于以下上下文信息回答问题。

上下文:
- 用户是一名大学生
- 预算有限（每天200元）
- 对历史和文化感兴趣
- 计划周末出行

问题: 推荐一个适合的一日游目的地"""
```

---

## Prompt 模板

### 1. 问答模板
```python
qa_template = """请根据以下信息回答问题。

背景信息:
{context}

问题:
{question}

回答要求:
- 简洁明了
- 基于提供的信息回答
- 如果信息不足，说明需要更多信息

回答:"""
```

### 2. 摘要模板
```python
summary_template = """请为以下内容生成摘要。

内容:
{content}

摘要要求:
- 字数限制: {max_words}字
- 突出重点
- 保持原文核心意思

摘要:"""
```

### 3. 翻译模板
```python
translation_template = """请将以下{source_lang}文本翻译成{target_lang}。

翻译风格:
- 保持原文语气
- 符合目标语言的表达习惯
- 专业术语保持一致

原文:
{text}

翻译:"""
```

### 4. 分类模板
```python
classification_template = """请将以下文本分类到合适的类别中。

可选类别:
{categories}

文本:
{text}

输出格式:
类别: [分类结果]
置信度: [0-100%]
理由: [简要说明]

分类结果:"""
```

### 5. 代码生成模板
```python
code_template = """请生成满足以下需求的代码:

编程语言: {language}
需求描述:
{requirements}

代码要求:
- 遵循{style}代码风格
- 包含适当的注释
- 处理可能的异常情况
- {additional_requirements}

代码:"""
```

### 6. 对话模板
```python
dialogue_template = """作为{persona}，与用户进行对话。

角色设定:
{role_description}

当前情境:
{situation}

对话要求:
- 保持角色一致性
- 语气符合角色设定
- 自然流畅

对话:
用户: {user_input}
{persona}:"""
```

### 7. 分析模板
```python
analysis_template = """请对以下内容进行全面分析。

分析对象:
{content}

分析维度:
{analysis_dimensions}

分析要求:
- 数据驱动
- 客观中立
- 提供具体见解

分析结果:"""
```

### 8. 创意写作模板
```python
writing_template = """请按照以下要求进行创意写作。

写作类型: {writing_type}
主题: {theme}

具体要求:
- 篇幅: {length}
- 风格: {style}
- 受众: {audience}
- {specific_requirements}

内容:"""
```

### 完整示例

```python
class PromptEngine:
    """Prompt 工程类"""
    
    @staticmethod
    def build_qa_prompt(context: str, question: str) -> str:
        return f"""请根据以下信息回答问题。

背景信息:
{context}

问题:
{question}

回答要求:
- 简洁明了
- 基于提供的信息回答
- 如果信息不足，说明需要更多信息

回答:"""
    
    @staticmethod
    def build_code_review_prompt(code: str) -> str:
        return f"""你是一位代码审查专家。请审查以下代码:

```{code}
```

请从以下角度进行审查:
1. 代码正确性
2. 性能问题
3. 安全漏洞
4. 代码可读性
5. 最佳实践

请给出具体的改进建议。"""
    
    @staticmethod
    def build_summarization_prompt(content: str, max_words: int = 100) -> str:
        return f"""请为以下内容生成摘要。

内容:
{content}

摘要要求:
- 字数限制: {max_words}字
- 突出重点
- 保持原文核心意思

摘要:"""

def main():
    engine = PromptEngine()
    
    print("=== QA Prompt ===")
    prompt = engine.build_qa_prompt(
        context="Python是一种高级编程语言。",
        question="Python是什么?"
    )
    print(prompt)
    
    print("\n=== Code Review Prompt ===")
    prompt = engine.build_code_review_prompt(
        code="def add(a, b): return a + b"
    )
    print(prompt)

if __name__ == '__main__':
    main()
```
