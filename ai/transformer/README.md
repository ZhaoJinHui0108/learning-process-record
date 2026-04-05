# Transformer 模型详解

## 目录
1. [Transformer 基本原理](#transformer-基本原理)
2. [可运行的 Transformer 实例](#可运行的-transformer-实例)

---

## Transformer 基本原理

### Transformer 架构
Transformer由Vaswani等人在2017年提出，主要包含以下组件：

1. **输入嵌入 (Input Embedding)**: 将token转换为向量
2. **位置编码 (Positional Encoding)**: 添加位置信息
3. **编码器 (Encoder)**: N层堆叠
   - 多头自注意力 (Multi-Head Attention)
   - 前馈网络 (Feed Forward Network)
   - 残差连接和层归一化
4. **解码器 (Decoder)**: N层堆叠
   - 掩码多头自注意力 (Masked Multi-Head Attention)
   - 编码器-解码器注意力 (Cross Attention)
   - 前馈网络

### 核心组件

#### Self-Attention (自注意力)
```
Self-Attention(Q, K, V) = softmax(QK^T / √d_k)V

Q = X * W_q  (Query)
K = X * W_k  (Key)
V = X * W_v  (Value)
```

#### Multi-Head Attention
```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) * W_o

head_i = SelfAttention(QW_q^i, KW_k^i, VW_v^i)
```

#### 位置编码
```python
# Sinusoidal Position Encoding
PE(pos, 2i) = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

---

## 可运行的 Transformer 实例

### 示例: 使用 PyTorch 实现 Transformer 机器翻译

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import math

class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)
        
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)

class TransformerTranslator(nn.Module):
    def __init__(self, src_vocab_size, tgt_vocab_size, d_model=256, nhead=8, 
                 num_encoder_layers=3, num_decoder_layers=3, dim_feedforward=512, 
                 dropout=0.1):
        super().__init__()
        
        self.d_model = d_model
        self.src_embedding = nn.Embedding(src_vocab_size, d_model)
        self.tgt_embedding = nn.Embedding(tgt_vocab_size, d_model)
        self.pos_encoder = PositionalEncoding(d_model, dropout=dropout)
        
        self.transformer = nn.Transformer(
            d_model=d_model,
            nhead=nhead,
            num_encoder_layers=num_encoder_layers,
            num_decoder_layers=num_decoder_layers,
            dim_feedforward=dim_feedforward,
            dropout=dropout,
            batch_first=True
        )
        
        self.fc = nn.Linear(d_model, tgt_vocab_size)
        
        self._init_weights()
    
    def _init_weights(self):
        for p in self.parameters():
            if p.dim() > 1:
                nn.init.xavier_uniform_(p)
    
    def generate_square_subsequent_mask(self, sz):
        mask = torch.triu(torch.ones(sz, sz), diagonal=1).bool()
        return mask
    
    def forward(self, src, tgt):
        src_pad_mask = (src == 0)
        tgt_pad_mask = (tgt == 0)
        
        tgt_mask = self.generate_square_subsequent_mask(tgt.size(1)).to(tgt.device)
        
        src_emb = self.pos_encoder(self.src_embedding(src) * math.sqrt(self.d_model))
        tgt_emb = self.pos_encoder(self.tgt_embedding(tgt) * math.sqrt(self.d_model))
        
        output = self.transformer(
            src_emb, tgt_emb,
            tgt_mask=tgt_mask,
            src_key_padding_mask=src_pad_mask,
            tgt_key_padding_mask=tgt_pad_mask,
            memory_key_padding_mask=src_pad_mask
        )
        
        return self.fc(output)
    
    def encode(self, src):
        src_pad_mask = (src == 0)
        src_emb = self.pos_encoder(self.src_embedding(src) * math.sqrt(self.d_model))
        return self.transformer.encoder(src_emb, src_key_padding_mask=src_pad_mask)
    
    def decode(self, tgt, memory):
        tgt_mask = self.generate_square_subsequent_mask(tgt.size(1)).to(tgt.device)
        tgt_pad_mask = (tgt == 0)
        tgt_emb = self.pos_encoder(self.tgt_embedding(tgt) * math.sqrt(self.d_model))
        return self.transformer.decoder(
            tgt_emb, memory,
            tgt_mask=tgt_mask,
            tgt_key_padding_mask=tgt_pad_mask
        )

class DummyDataset(Dataset):
    def __init__(self, src_data, tgt_data, src_vocab, tgt_vocab, max_len=20):
        self.data = []
        for src, tgt in zip(src_data, tgt_data):
            src_idx = [src_vocab.get(w, 1) for w in src.split()[:max_len]]
            tgt_idx = [tgt_vocab.get(w, 1) for w in tgt.split()[:max_len]]
            self.data.append((src_idx, tgt_idx))
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        src, tgt = self.data[idx]
        return (
            torch.tensor(src + [0] * (20 - len(src))),
            torch.tensor(tgt + [0] * (20 - len(tgt)))
        )

def greedy_decode(model, src, max_len=20, start_symbol=2):
    model.eval()
    with torch.no_grad():
        memory = model.encode(src)
        ys = torch.ones(1, 1).fill_(start_symbol).long().to(src.device)
        
        for i in range(max_len - 1):
            out = model.decode(ys, memory)
            prob = model.fc(out[:, -1])
            _, next_word = torch.max(prob, dim=1)
            next_word = next_word.item()
            
            ys = torch.cat([ys, torch.ones(1, 1).type_as(src.data).fill_(next_word)], dim=1)
            
            if next_word == 3:
                break
    
    return ys

def main():
    torch.manual_seed(42)
    
    # 简单的中英对照数据
    src_sentences = [
        "hello how are you",
        "i am fine thank you",
        "what is your name",
        "my name is tom"
    ]
    
    tgt_sentences = [
        "你好 你好 吗",
        "我 很 好 谢谢 你",
        "你 叫 什么 名字",
        "我 叫 汤姆"
    ]
    
    # 构建词汇表
    src_vocab = {'<pad>': 0, '<unk>': 1, '<sos>': 2, '<eos>': 3}
    tgt_vocab = {'<pad>': 0, '<unk>': 1, '<sos>': 2, '<eos>': 3}
    
    for sent in src_sentences:
        for word in sent.split():
            if word not in src_vocab:
                src_vocab[word] = len(src_vocab)
    
    for sent in tgt_sentences:
        for word in sent.split():
            if word not in tgt_vocab:
                tgt_vocab[word] = len(tgt_vocab)
    
    idx_to_tgt = {v: k for k, v in tgt_vocab.items()}
    
    print(f"Source Vocabulary Size: {len(src_vocab)}")
    print(f"Target Vocabulary Size: {len(tgt_vocab)}")
    
    # 创建数据集
    dataset = DummyDataset(src_sentences, tgt_sentences, src_vocab, tgt_vocab)
    dataloader = DataLoader(dataset, batch_size=2, shuffle=True)
    
    # 创建模型
    model = TransformerTranslator(
        src_vocab_size=len(src_vocab),
        tgt_vocab_size=len(tgt_vocab),
        d_model=128,
        nhead=4,
        num_encoder_layers=2,
        num_decoder_layers=2,
        dim_feedforward=256
    )
    
    criterion = nn.CrossEntropyLoss(ignore_index=0)
    optimizer = optim.Adam(model.parameters(), lr=0.0001)
    
    # 训练
    print("\nTraining Transformer...")
    model.train()
    for epoch in range(100):
        total_loss = 0
        for src, tgt in dataloader:
            tgt_input = tgt[:, :-1]
            tgt_output = tgt[:, 1:]
            
            optimizer.zero_grad()
            
            output = model(src, tgt_input)
            loss = criterion(output.reshape(-1, len(tgt_vocab)), tgt_output.reshape(-1))
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
        
        if (epoch + 1) % 20 == 0:
            print(f"Epoch [{epoch+1}/100], Loss: {total_loss/len(dataloader):.4f}")
    
    # 测试翻译
    print("\nTranslation Results:")
    model.eval()
    test_src = torch.tensor([[src_vocab.get(w, 1) for w in src_sentences[0].split()]])
    
    translation = greedy_decode(model, test_src, start_symbol=2)
    translation_words = [idx_to_tgt[idx.item()] for idx in translation[0] 
                        if idx.item() not in [0, 2, 3]]
    print(f"Input: {src_sentences[0]}")
    print(f"Output: {' '.join(translation_words)}")

if __name__ == '__main__':
    main()
```

### Transformer 各部分说明

| 组件 | 说明 |
|------|------|
| **PositionalEncoding** | 为序列中的每个位置添加独特的位置编码，使模型能够学习序列顺序 |
| **TransformerTranslator** | 主模型类，包含完整的Transformer架构 |
| **src_embedding** | 源语言嵌入层，将源序列token转换为d_model维向量 |
| **tgt_embedding** | 目标语言嵌入层，用于解码器输入 |
| **transformer** | PyTorch内置的Transformer模块，包含编码器和解码器 |
| **generate_square_subsequent_mask** | 生成上三角掩码，防止解码器看到未来信息 |
| **forward** | 前向传播，处理完整的序列到序列任务 |
| **greedy_decode** | 贪婪解码策略，逐词生成翻译结果 |
