# 神经网络基础实例

## 目录
1. [神经网络基础](#神经网络基础)
2. [可运行的神经网络实例](#可运行的神经网络实例)

---

## 神经网络基础

### 神经网络组成
- **神经元 (Neuron)**: 基本计算单元
- **层 (Layer)**: 神经元的集合
  - 输入层: 接收特征
  - 隐藏层: 特征变换
  - 输出层: 输出预测
- **权重 (Weight)**: 连接强度
- **偏置 (Bias)**: 调整输出
- **激活函数 (Activation)**: 引入非线性

### 激活函数
| 函数 | 公式 | 特点 |
|------|------|------|
| Sigmoid | 1/(1+e^(-x)) | 输出0-1，易饱和 |
| Tanh | (e^x-e^(-x))/(e^x+e^(-x)) | 输出-1到1 |
| ReLU | max(0,x) | 计算快，不饱和 |
| Leaky ReLU | max(0.01x, x) | 避免死亡神经元 |

### 损失函数
| 任务 | 损失函数 |
|------|----------|
| 回归 | MSE, MAE |
| 二分类 | Binary Cross Entropy |
| 多分类 | Cross Entropy |

---

## 可运行的神经网络实例

### 示例1: 数字识别神经网络

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import matplotlib.pyplot as plt

class SimpleNN(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.layer1 = nn.Linear(input_size, hidden_size)
        self.relu1 = nn.ReLU()
        self.layer2 = nn.Linear(hidden_size, hidden_size)
        self.relu2 = nn.ReLU()
        self.layer3 = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        x = self.relu1(self.layer1(x))
        x = self.relu2(self.layer2(x))
        x = self.layer3(x)
        return x

class MNISTLikeDataset(Dataset):
    def __init__(self, num_samples=1000):
        torch.manual_seed(42)
        self.num_samples = num_samples
        
        clusters = []
        labels = []
        
        for digit in range(10):
            center = torch.randn(2) * 2 + torch.tensor([digit % 3, digit // 3])
            cluster = torch.randn(num_samples // 10, 2) * 0.5 + center
            clusters.append(cluster)
            labels.append(torch.full((num_samples // 10,), digit))
        
        self.data = torch.cat(clusters, dim=0)
        self.labels = torch.cat(labels, dim=0).long()
        
        self.mean = self.data.mean(dim=0)
        self.std = self.data.std(dim=0)
        self.data = (self.data - self.mean) / (self.std + 1e-8)
    
    def __len__(self):
        return self.num_samples
    
    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

def train_model():
    torch.manual_seed(42)
    
    dataset = MNISTLikeDataset(num_samples=1000)
    dataloader = DataLoader(dataset, batch_size=32, shuffle=True)
    
    model = SimpleNN(input_size=2, hidden_size=32, output_size=10)
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.01)
    
    num_epochs = 100
    losses = []
    accuracies = []
    
    for epoch in range(num_epochs):
        epoch_loss = 0
        correct = 0
        total = 0
        
        for data, labels in dataloader:
            optimizer.zero_grad()
            
            outputs = model(data)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            epoch_loss += loss.item()
            _, predicted = torch.max(outputs.data, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()
        
        avg_loss = epoch_loss / len(dataloader)
        accuracy = 100 * correct / total
        losses.append(avg_loss)
        accuracies.append(accuracy)
        
        if (epoch + 1) % 20 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {avg_loss:.4f}, Accuracy: {accuracy:.2f}%')
    
    print(f'\nFinal Accuracy: {accuracies[-1]:.2f}%')
    
    plt.figure(figsize=(12, 4))
    
    plt.subplot(1, 2, 1)
    plt.plot(losses)
    plt.xlabel('Epoch')
    plt.ylabel('Loss')
    plt.title('Training Loss')
    
    plt.subplot(1, 2, 2)
    plt.plot(accuracies)
    plt.xlabel('Epoch')
    plt.ylabel('Accuracy (%)')
    plt.title('Training Accuracy')
    
    plt.tight_layout()
    plt.savefig('training_results.png')
    
    return model, dataset

def test_model(model, dataset):
    model.eval()
    with torch.no_grad():
        data, labels = dataset[:100]
        outputs = model(data)
        _, predicted = torch.max(outputs, 1)
        accuracy = (predicted == labels).sum().item() / len(labels) * 100
    print(f'Test Accuracy: {accuracy:.2f}%')
    
    plt.figure(figsize=(8, 6))
    plt.scatter(data[:, 0], data[:, 1], c=predicted.numpy(), cmap='tab10', alpha=0.7)
    plt.colorbar()
    plt.title('Predicted Classes')
    plt.savefig('predictions.png')

if __name__ == '__main__':
    model, dataset = train_model()
    test_model(model, dataset)
    print('\nTraining complete!')
```

### 示例2: 卷积神经网络图像分类

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import numpy as np
import matplotlib.pyplot as plt

class SimpleCNN(nn.Module):
    def __init__(self):
        super().__init__()
        
        self.conv_layers = nn.Sequential(
            nn.Conv2d(1, 16, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            
            nn.Conv2d(16, 32, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2, 2)
        )
        
        self.fc_layers = nn.Sequential(
            nn.Linear(64 * 3 * 3, 128),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(128, 10)
        )
    
    def forward(self, x):
        x = self.conv_layers(x)
        x = x.view(x.size(0), -1)
        x = self.fc_layers(x)
        return x

class SyntheticImageDataset(Dataset):
    def __init__(self, num_samples=1000, img_size=24):
        torch.manual_seed(42)
        np.random.seed(42)
        
        self.num_samples = num_samples
        self.img_size = img_size
        
        self.images = []
        self.labels = []
        
        for i in range(num_samples):
            label = i % 10
            
            img = np.zeros((1, img_size, img_size), dtype=np.float32)
            
            base_x = np.random.randint(3, img_size - 6)
            base_y = np.random.randint(3, img_size - 6)
            
            angle = label * 36 + np.random.randint(-10, 10)
            length = np.random.randint(5, 8)
            
            rad = np.radians(angle)
            dx = length * np.cos(rad)
            dy = length * np.sin(rad)
            
            x2, y2 = int(base_x + dx), int(base_y + dy)
            x2 = np.clip(x2, 0, img_size - 1)
            y2 = np.clip(y2, 0, img_size - 1)
            
            for x in range(min(base_x, x2), max(base_x, x2) + 1):
                for y in range(min(base_y, y2), max(base_y, y2) + 1):
                    img[0, y, x] = 1.0
            
            noise = np.random.randn(1, img_size, img_size) * 0.1
            img = np.clip(img + noise, 0, 1)
            
            self.images.append(torch.from_numpy(img))
            self.labels.append(label)
    
    def __len__(self):
        return self.num_samples
    
    def __getitem__(self, idx):
        return self.images[idx], self.labels[idx]

def train_cnn():
    torch.manual_seed(42)
    
    train_dataset = SyntheticImageDataset(num_samples=800)
    test_dataset = SyntheticImageDataset(num_samples=200)
    
    train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
    test_loader = DataLoader(test_dataset, batch_size=32)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f'Using device: {device}')
    
    model = SimpleCNN().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)
    
    num_epochs = 20
    train_losses = []
    train_accs = []
    
    for epoch in range(num_epochs):
        model.train()
        running_loss = 0.0
        correct = 0
        total = 0
        
        for images, labels in train_loader:
            images, labels = images.to(device), labels.to(device)
            
            optimizer.zero_grad()
            outputs = model(images)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            
            running_loss += loss.item()
            _, predicted = torch.max(outputs.data, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()
        
        avg_loss = running_loss / len(train_loader)
        accuracy = 100 * correct / total
        train_losses.append(avg_loss)
        train_accs.append(accuracy)
        
        if (epoch + 1) % 5 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {avg_loss:.4f}, Acc: {accuracy:.2f}%')
    
    model.eval()
    test_correct = 0
    test_total = 0
    
    with torch.no_grad():
        for images, labels in test_loader:
            images, labels = images.to(device), labels.to(device)
            outputs = model(images)
            _, predicted = torch.max(outputs.data, 1)
            test_total += labels.size(0)
            test_correct += (predicted == labels).sum().item()
    
    print(f'\nTest Accuracy: {100 * test_correct / test_total:.2f}%')
    
    plt.figure(figsize=(10, 4))
    plt.subplot(1, 2, 1)
    plt.plot(train_losses)
    plt.title('Training Loss')
    
    plt.subplot(1, 2, 2)
    plt.plot(train_accs)
    plt.title('Training Accuracy')
    
    plt.tight_layout()
    plt.savefig('cnn_training.png')
    
    torch.save(model.state_dict(), 'simple_cnn.pth')

if __name__ == '__main__':
    train_cnn()
```

### 示例3: 循环神经网络序列预测

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import numpy as np

class RNNModel(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        self.rnn = nn.RNN(input_size, hidden_size, num_layers, batch_first=True, dropout=0.2)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size)
        
        out, _ = self.rnn(x, h0)
        out = self.fc(out[:, -1, :])
        return out

class SequenceDataset(Dataset):
    def __init__(self, num_sequences=1000, seq_length=10):
        torch.manual_seed(42)
        np.random.seed(42)
        
        self.num_sequences = num_sequences
        self.seq_length = seq_length
        
        sequences = []
        labels = []
        
        for _ in range(num_sequences):
            start = np.random.randint(-10, 10)
            
            x = np.array([start + i + np.random.randn() * 0.5 for i in range(seq_length)])
            
            next_val = start + seq_length
            
            sequences.append(torch.tensor(x, dtype=torch.float32))
            labels.append(torch.tensor(next_val, dtype=torch.float32))
        
        sequences = torch.stack(sequences)
        labels = torch.stack(labels)
        
        self.mean = sequences.mean()
        self.std = sequences.std()
        sequences = (sequences - self.mean) / (self.std + 1e-8)
        labels = (labels - self.mean) / (self.std + 1e-8)
        
        self.X = sequences.unsqueeze(-1)
        self.y = labels
    
    def __len__(self):
        return self.num_sequences
    
    def __getitem__(self, idx):
        return self.X[idx], self.y[idx]

def train_rnn():
    torch.manual_seed(42)
    
    dataset = SequenceDataset(num_sequences=1000, seq_length=10)
    dataloader = DataLoader(dataset, batch_size=32, shuffle=True)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f'Using device: {device}')
    
    model = RNNModel(
        input_size=1,
        hidden_size=32,
        num_layers=2,
        output_size=1
    ).to(device)
    
    criterion = nn.MSELoss()
    optimizer = optim.Adam(model.parameters(), lr=0.01)
    
    num_epochs = 100
    
    for epoch in range(num_epochs):
        model.train()
        total_loss = 0
        
        for sequences, labels in dataloader:
            sequences, labels = sequences.to(device), labels.to(device)
            
            optimizer.zero_grad()
            outputs = model(sequences)
            loss = criterion(outputs.squeeze(), labels)
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
        
        if (epoch + 1) % 20 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {total_loss/len(dataloader):.4f}')
    
    model.eval()
    with torch.no_grad():
        test_seq = torch.tensor([[[i + np.random.randn() * 0.5] for i in range(10)]], dtype=torch.float32)
        test_seq = (test_seq - dataset.mean) / (dataset.std + 1e-8)
        test_seq = test_seq.to(device)
        
        prediction = model(test_seq)
        
        predicted_val = prediction.item() * (dataset.std.item() + 1e-8) + dataset.mean.item()
        actual_val = 10 + np.random.randn() * 0.5
        
        print(f'\nTest Prediction: {predicted_val:.2f}')
        print(f'Expected value (approx): 10')
    
    torch.save(model.state_dict(), 'rnn_model.pth')

if __name__ == '__main__':
    train_rnn()
```

### 示例4: 生成对抗网络 (GAN)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import matplotlib.pyplot as plt

class Generator(nn.Module):
    def __init__(self, latent_dim=100, output_dim=2):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(latent_dim, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 256),
            nn.BatchNorm1d(256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 512),
            nn.BatchNorm1d(512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, output_dim)
        )
    
    def forward(self, z):
        return self.net(z)

class Discriminator(nn.Module):
    def __init__(self, input_dim=2):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 1),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        return self.net(x)

class CircleDataset(Dataset):
    def __init__(self, num_samples=1000, radius=2, noise=0.1):
        torch.manual_seed(42)
        
        angles = torch.rand(num_samples) * 2 * torch.pi
        r = radius + torch.randn(num_samples) * noise
        
        x = r * torch.cos(angles)
        y = r * torch.sin(angles)
        
        self.data = torch.stack([x, y], dim=1)
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        return self.data[idx]

def train_gan():
    torch.manual_seed(42)
    
    dataset = CircleDataset(num_samples=1000)
    dataloader = DataLoader(dataset, batch_size=64, shuffle=True)
    
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f'Using device: {device}')
    
    latent_dim = 100
    generator = Generator(latent_dim).to(device)
    discriminator = Discriminator().to(device)
    
    criterion = nn.BCELoss()
    optimizer_G = optim.Adam(generator.parameters(), lr=0.001)
    optimizer_D = optim.Adam(discriminator.parameters(), lr=0.001)
    
    num_epochs = 200
    
    for epoch in range(num_epochs):
        for real_data in dataloader:
            batch_size = real_data.size(0)
            real_data = real_data.to(device)
            
            real_labels = torch.ones(batch_size, 1).to(device)
            fake_labels = torch.zeros(batch_size, 1).to(device)
            
            optimizer_D.zero_grad()
            
            real_output = discriminator(real_data)
            loss_D_real = criterion(real_output, real_labels)
            
            z = torch.randn(batch_size, latent_dim).to(device)
            fake_data = generator(z)
            fake_output = discriminator(fake_data.detach())
            loss_D_fake = criterion(fake_output, fake_labels)
            
            loss_D = loss_D_real + loss_D_fake
            loss_D.backward()
            optimizer_D.step()
            
            optimizer_G.zero_grad()
            
            z = torch.randn(batch_size, latent_dim).to(device)
            fake_data = generator(z)
            fake_output = discriminator(fake_data)
            loss_G = criterion(fake_output, real_labels)
            
            loss_G.backward()
            optimizer_G.step()
        
        if (epoch + 1) % 50 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Loss_D: {loss_D.item():.4f}, Loss_G: {loss_G.item():.4f}')
    
    generator.eval()
    with torch.no_grad():
        z = torch.randn(500, latent_dim).to(device)
        generated_samples = generator(z).cpu()
    
    plt.figure(figsize=(8, 8))
    plt.scatter(dataset.data[:, 0], dataset.data[:, 1], alpha=0.3, label='Real Data', s=10)
    plt.scatter(generated_samples[:, 0], generated_samples[:, 1], alpha=0.5, label='Generated', s=10)
    plt.legend()
    plt.title('GAN Generated Circle Data')
    plt.savefig('gan_results.png')
    
    torch.save(generator.state_dict(), 'generator.pth')
    print('\nGAN training complete!')

if __name__ == '__main__':
    train_gan()
```
