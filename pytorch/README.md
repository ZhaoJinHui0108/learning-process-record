# PyTorch 学习笔记

## 目录
1. [PyTorch 基础](#pytorch-基础)
2. [神经网络实例](#神经网络实例)

---

## PyTorch 基础

### 张量 (Tensor) 操作
```python
import torch
import numpy as np

# 创建张量
x = torch.tensor([1.0, 2.0, 3.0])
y = torch.zeros(3, 4)
z = torch.rand(2, 3)

# 从NumPy创建
np_array = np.array([[1, 2], [3, 4]])
tensor_from_np = torch.from_numpy(np_array)

# 张量运算
a = torch.tensor([1.0, 2.0])
b = torch.tensor([3.0, 4.0])
c = a + b
d = torch.matmul(a, b)

# GPU支持
if torch.cuda.is_available():
    device = torch.device('cuda')
    tensor_gpu = x.to(device)
```

---

## 神经网络实例

### 示例1: 简单线性回归
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import matplotlib.pyplot as plt

class LinearRegressionModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(1, 1)
    
    def forward(self, x):
        return self.linear(x)

class DummyDataset(Dataset):
    def __init__(self, x_data, y_data):
        self.x = torch.FloatTensor(x_data).reshape(-1, 1)
        self.y = torch.FloatTensor(y_data).reshape(-1, 1)
    
    def __len__(self):
        return len(self.x)
    
    def __getitem__(self, idx):
        return self.x[idx], self.y[idx]

def train_model():
    torch.manual_seed(42)
    
    x_data = torch.randn(100, 1) * 10
    y_data = 3 * x_data + 5 + torch.randn(100, 1) * 2
    
    dataset = DummyDataset(x_data.numpy(), y_data.numpy())
    dataloader = DataLoader(dataset, batch_size=16, shuffle=True)
    
    model = LinearRegressionModel()
    criterion = nn.MSELoss()
    optimizer = optim.SGD(model.parameters(), lr=0.01)
    
    num_epochs = 100
    losses = []
    
    for epoch in range(num_epochs):
        epoch_loss = 0.0
        for batch_x, batch_y in dataloader:
            optimizer.zero_grad()
            outputs = model(batch_x)
            loss = criterion(outputs, batch_y)
            loss.backward()
            optimizer.step()
            epoch_loss += loss.item()
        
        avg_loss = epoch_loss / len(dataloader)
        losses.append(avg_loss)
        
        if (epoch + 1) % 20 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {avg_loss:.4f}')
    
    print(f'\nFinal parameters:')
    print(f'  Weight: {model.linear.weight.item():.4f}')
    print(f'  Bias: {model.linear.bias.item():.4f}')
    
    plt.plot(losses)
    plt.xlabel('Epoch')
    plt.ylabel('Loss')
    plt.title('Training Loss')
    plt.savefig('training_loss.png')
    
    return model

if __name__ == '__main__':
    model = train_model()
    print('\nModel trained successfully!')
```

### 示例2: 手写数字识别 (MNIST)
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3, padding=1)
        self.relu1 = nn.ReLU()
        self.pool1 = nn.MaxPool2d(2, 2)
        
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.relu2 = nn.ReLU()
        self.pool2 = nn.MaxPool2d(2, 2)
        
        self.fc1 = nn.Linear(64 * 7 * 7, 128)
        self.relu3 = nn.ReLU()
        self.dropout = nn.Dropout(0.5)
        self.fc2 = nn.Linear(128, 10)
    
    def forward(self, x):
        x = self.pool1(self.relu1(self.conv1(x)))
        x = self.pool2(self.relu2(self.conv2(x)))
        x = x.view(-1, 64 * 7 * 7)
        x = self.dropout(self.relu3(self.fc1(x)))
        x = self.fc2(x)
        return x

def train_mnist():
    transform = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,))
    ])
    
    train_dataset = datasets.MNIST('./data', train=True, download=True, transform=transform)
    test_dataset = datasets.MNIST('./data', train=False, transform=transform)
    
    train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
    test_loader = DataLoader(test_dataset, batch_size=1000)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f'Using device: {device}')
    
    model = CNN().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)
    
    num_epochs = 5
    for epoch in range(num_epochs):
        model.train()
        running_loss = 0.0
        correct = 0
        total = 0
        
        for batch_idx, (data, target) in enumerate(train_loader):
            data, target = data.to(device), target.to(device)
            
            optimizer.zero_grad()
            outputs = model(data)
            loss = criterion(outputs, target)
            loss.backward()
            optimizer.step()
            
            running_loss += loss.item()
            _, predicted = outputs.max(1)
            total += target.size(0)
            correct += predicted.eq(target).sum().item()
        
        print(f'Epoch [{epoch+1}/{num_epochs}], '
              f'Loss: {running_loss/len(train_loader):.4f}, '
              f'Accuracy: {100.*correct/total:.2f}%')
    
    model.eval()
    test_loss = 0.0
    correct = 0
    total = 0
    
    with torch.no_grad():
        for data, target in test_loader:
            data, target = data.to(device), target.to(device)
            outputs = model(data)
            loss = criterion(outputs, target)
            test_loss += loss.item()
            _, predicted = outputs.max(1)
            total += target.size(0)
            correct += predicted.eq(target).sum().item()
    
    print(f'\nTest Accuracy: {100.*correct/total:.2f}%')
    
    torch.save(model.state_dict(), 'mnist_cnn.pth')
    print('Model saved to mnist_cnn.pth')

if __name__ == '__main__':
    train_mnist()
```

### 示例3: RNN 文本分类
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import numpy as np

class TextClassificationModel(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.GRU(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_classes)
    
    def forward(self, x):
        embedded = self.embedding(x)
        _, hidden = self.rnn(embedded)
        output = self.fc(hidden.squeeze(0))
        return output

class TextDataset(Dataset):
    def __init__(self, texts, labels, word_to_idx, max_len=100):
        self.texts = texts
        self.labels = labels
        self.word_to_idx = word_to_idx
        self.max_len = max_len
    
    def __len__(self):
        return len(self.texts)
    
    def __getitem__(self, idx):
        text = self.texts[idx]
        label = self.labels[idx]
        
        indices = [self.word_to_idx.get(w, 0) for w in text.split()[:self.max_len]]
        indices += [0] * (self.max_len - len(indices))
        
        return torch.LongTensor(indices), torch.LongTensor([label])

def train_rnn_classifier():
    texts = [
        "this movie is great",
        "this movie is terrible",
        "great film highly recommend",
        "waste of time boring",
        "loved it amazing",
        "horrible worst ever",
    ]
    labels = [1, 0, 1, 0, 1, 0]
    
    word_to_idx = {
        'this': 1, 'movie': 2, 'is': 3, 'great': 4, 'terrible': 5,
        'film': 6, 'highly': 7, 'recommend': 8, 'waste': 9, 'of': 10,
        'time': 11, 'boring': 12, 'loved': 13, 'it': 14, 'amazing': 15,
        'horrible': 16, 'worst': 17, 'ever': 18
    }
    
    dataset = TextDataset(texts, labels, word_to_idx, max_len=10)
    dataloader = DataLoader(dataset, batch_size=2, shuffle=True)
    
    vocab_size = len(word_to_idx) + 2
    embed_dim = 16
    hidden_dim = 32
    num_classes = 2
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = TextClassificationModel(vocab_size, embed_dim, hidden_dim, num_classes).to(device)
    
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.01)
    
    model.train()
    for epoch in range(100):
        total_loss = 0
        for texts, labels in dataloader:
            texts, labels = texts.to(device), labels.squeeze().to(device)
            
            optimizer.zero_grad()
            outputs = model(texts)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
        
        if (epoch + 1) % 20 == 0:
            print(f'Epoch [{epoch+1}/100], Loss: {total_loss/len(dataloader):.4f}')
    
    model.eval()
    test_texts = ["great movie amazing", "terrible boring film"]
    test_indices = []
    
    for text in test_texts:
        indices = [word_to_idx.get(w, 0) for w in text.split()[:10]]
        indices += [0] * (10 - len(indices))
        test_indices.append(indices)
    
    test_tensor = torch.LongTensor(test_indices).to(device)
    
    with torch.no_grad():
        outputs = model(test_tensor)
        predictions = torch.argmax(outputs, dim=1)
        probs = torch.softmax(outputs, dim=1)
    
    print('\nPredictions:')
    for text, pred, prob in zip(test_texts, predictions, probs):
        sentiment = "Positive" if pred.item() == 1 else "Negative"
        print(f'  "{text}" -> {sentiment} (confidence: {prob[pred].item():.4f})')

if __name__ == '__main__':
    train_rnn_classifier()
```

### 示例4: 自编码器 (Autoencoder)
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms
import matplotlib.pyplot as plt

class Autoencoder(nn.Module):
    def __init__(self, encoding_dim=32):
        super().__init__()
        
        self.encoder = nn.Sequential(
            nn.Linear(784, 256),
            nn.ReLU(),
            nn.Linear(256, encoding_dim),
            nn.ReLU()
        )
        
        self.decoder = nn.Sequential(
            nn.Linear(encoding_dim, 256),
            nn.ReLU(),
            nn.Linear(256, 784),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        encoded = self.encoder(x)
        decoded = self.decoder(encoded)
        return decoded
    
    def encode(self, x):
        return self.encoder(x)

def train_autoencoder():
    transform = transforms.Compose([
        transforms.ToTensor(),
        transforms.Lambda(lambda x: x.view(-1))
    ])
    
    dataset = datasets.MNIST('./data', train=True, download=True, transform=transform)
    dataloader = DataLoader(dataset, batch_size=128, shuffle=True)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = Autoencoder(encoding_dim=32).to(device)
    
    criterion = nn.BCELoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)
    
    num_epochs = 10
    for epoch in range(num_epochs):
        total_loss = 0
        for data, _ in dataloader:
            data = data.to(device)
            
            optimizer.zero_grad()
            outputs = model(data)
            loss = criterion(outputs, data)
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
        
        print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {total_loss/len(dataloader):.4f}')
    
    torch.save(model.state_dict(), 'autoencoder.pth')
    
    model.eval()
    with torch.no_grad():
        sample = dataset[0][0].view(-1, 784).to(device)
        reconstructed = model(sample).cpu().view(28, 28)
        
        fig, axes = plt.subplots(1, 2)
        axes[0].imshow(dataset[0][0].view(28, 28), cmap='gray')
        axes[0].set_title('Original')
        axes[1].imshow(reconstructed.numpy(), cmap='gray')
        axes[1].set_title('Reconstructed')
        plt.savefig('autoencoder_result.png')
    
    print('\nAutoencoder trained and results saved!')

if __name__ == '__main__':
    train_autoencoder()
```

### 示例5: 迁移学习 (Transfer Learning)
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models

def train_transfer_learning():
    transform = transforms.Compose([
        transforms.Resize(255),
        transforms.CenterCrop(224),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])
    
    train_dataset = datasets.FakeData(1000, transform=transform)
    val_dataset = datasets.FakeData(200, transform=transform)
    
    train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
    val_loader = DataLoader(val_dataset, batch_size=32)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    
    model = models.resnet18(pretrained=True)
    num_features = model.fc.in_features
    model.fc = nn.Linear(num_features, 10)
    
    model = model.to(device)
    
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.SGD(model.parameters(), lr=0.001, momentum=0.9)
    
    for param in model.parameters():
        param.requires_grad = False
    
    for param in model.fc.parameters():
        param.requires_grad = True
    
    num_epochs = 5
    for epoch in range(num_epochs):
        model.train()
        running_loss = 0.0
        correct = 0
        total = 0
        
        for inputs, labels in train_loader:
            inputs, labels = inputs.to(device), labels.to(device)
            
            optimizer.zero_grad()
            outputs = model(inputs)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            running_loss += loss.item()
            _, predicted = outputs.max(1)
            total += labels.size(0)
            correct += predicted.eq(labels).sum().item()
        
        print(f'Epoch [{epoch+1}/{num_epochs}], '
              f'Loss: {running_loss/len(train_loader):.4f}, '
              f'Acc: {100.*correct/total:.2f}%')
    
    model.eval()
    correct = 0
    total = 0
    with torch.no_grad():
        for inputs, labels in val_loader:
            inputs, labels = inputs.to(device), labels.to(device)
            outputs = model(inputs)
            _, predicted = outputs.max(1)
            total += labels.size(0)
            correct += predicted.eq(labels).sum().item()
    
    print(f'\nValidation Accuracy: {100.*correct/total:.2f}%')

if __name__ == '__main__':
    train_transfer_learning()
```
