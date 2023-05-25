---
layout: post
title: "使用PyTorch构建CNN网络训练MNIST数据集"
date: 2023-05-26 00:00:52 +0800
comments: true
categories: 
---

#### 前言
本文记录了使用PyTorch构建一个简单的CNN网络，并使用MNIST数据集进行训练和测试。记录整个推理过程

#### 1、MNIST数据集
MNIST数据集是一个手写数字识别数据集，包含了60,000个28x28像素的手写数字图像（灰度图片），每个数字都有10个训练样本和10个测试样本。该数据集由C. E. Mnih等人于1999年首次发布，并被广泛应用于机器学习领域的分类、识别和生成模型的研究中。  
MNIST数据集的特点：  

  1、 每个数字都是手写的，因此具有一定的噪声和不规则性；  
  2、 每个数字只有一种写法，因此可以视为二元分类问题；  
  3、 数据集中的数字图像比较简单，易于处理和识别；  
  4、 数据集中的数字图像没有标签，需要通过预处理和特征提取等方法来自动识别。  
<!--more-->  

#### 2、构建卷积神经网络
整个神经网络一共5层，2个卷积层，2个线性层，1个全链接输出层,如下：  
Conv1->Relu->pool->Conv2->Relu->pool->Linear1->Relu->Linear2->Sofrmax
网络架构的说明：  

1、因为MNIST数据是一个灰度的28*28的图片，所以数据输入层只有1个输入通道。  
2、 卷积层的卷积核都使用3*3大小核。  
3、 权重参数使用Adam或者SGD来更新参数。  
4、 使用SGD方法，经过10epoch，准确率可以达到99.16%。  
代码如下
```
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchvision import datasets, transforms
import torchvision
from torch.autograd import Variable
import torch.optim as optim
import matplotlib.pyplot as plt
import sys

BATCH_SIZE = 200 #每单次训练时加载的数据量，如果是用GPU跑的话，可以设置得高一点。
EPOCHS = 10  # 总共训练批次
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")  # 让torch判断是否使用GPU，建议使用GPU环境，因为会快很多

train_loader = torch.utils.data.DataLoader(
    datasets.MNIST('../datasets/data', train=True, download=True,
                   transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize(mean=[0.5],std=[0.5])  # transforms.Normalize()将数据进行归一化处理，
                   ])),
    batch_size=BATCH_SIZE, shuffle=True)

test_loader = torch.utils.data.DataLoader(
    datasets.MNIST('../datasets/data', train=False, transform=transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.5],std=[0.5])
    ])),
    batch_size=BATCH_SIZE, shuffle=True)

# 网络结构 conv->Relu->pool->conv->Relu->pool->Linear->Relu->Linear->Sofrmax
class MNISTConvNet(nn.Module):
  def __init__(self):
    super().__init__()
    # 公式：OH=(H + 2P - FH)/S + 1; OW= (W + 2P - FW)/S + 1
    self.conv1 = nn.Conv2d(in_channels=1, out_channels=24, kernel_size=3, stride=1, padding=0) # 24, 26*26
    self.conv2 = nn.Conv2d(in_channels=24, out_channels=48, kernel_size=3, stride=1, padding=0) # 48, 11*11
    self.fc1 = nn.Linear(in_features=48*5*5, out_features=200)
    self.fc2 = nn.Linear(in_features=200, out_features=10)
    
  def forward(self, x):
    out = self.conv1(x)
    out = F.relu(out)
    out = F.max_pool2d(input=out, kernel_size=(2,2), stride=2) #24 13*13
    
    out = self.conv2(out)
    out = F.relu(out)
    out = F.max_pool2d(input=out, kernel_size=(2,2), stride=2) #48 5*5
    
    out = out.view(-1,48*5*5)
    out = self.fc1(out)
    out = F.relu(out)
    
    out = self.fc2(out)
    
    out = F.log_softmax(out, dim=1)
    return out

def train(model, optimizer, epoch):
  model.train() #设置模型为训练模式
  for batch_id, (images, labels) in enumerate(train_loader):
    images = images.to(DEVICE) # 将tensors移动到配置的device
    labels = labels.to(DEVICE)
    optimizer.zero_grad()  # 在反向传播的时候先把梯度记录清0
    output = model(images)
    loss = F.nll_loss(output, labels)
    loss.backward()
    optimizer.step()  # 调用step()方法更新梯度参数
    if (batch_id + 1) % 30 == 0:
        print('Train Epoch: {} [{}/{} ({:.0f}%)]\tLoss: {}'.format(
            epoch, batch_id * len(images), len(train_loader.dataset),
                    100. * batch_id / len(train_loader), loss.item()))

  
def test(model):
  model.eval()
  test_loss = 0
  correct = 0
  with torch.no_grad():  # 禁用梯度计算，验证模式一般不需要进行梯度更新及反向传播，所以不需要更新梯度参数
      for images, labels in test_loader:
          images, labels = images.to(DEVICE), labels.to(DEVICE)
          output = model(images)
          test_loss += F.nll_loss(output, labels, reduction='sum').item()  # 将一批的损失相加
          pred = output.max(1, keepdim=True)[1]  # 找到概率最大的下标
          correct += pred.eq(labels.view_as(pred)).sum().item()

  test_loss /= len(test_loader.dataset)
  print('\nTest set: Average loss: {:.4f}, Accuracy: {}/{} ({}%)\n'.format(
      test_loss, correct, len(test_loader.dataset),
      100. * correct / len(test_loader.dataset)))


def predict(model):
  val_loader = torch.utils.data.DataLoader(
    datasets.MNIST('../datasets/data', train=False, transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize(mean=[0.5],std=[0.5])
                   ])),batch_size=4, shuffle=True)
  iterator = iter(val_loader)#获得Iterator对象:
  X_val, Y_val = next(iterator) #循环Iterator对象迭代器，一般和上面的iter()一起使用
  inputs = Variable(X_val)
  #Variable()函数用于将一个张量转换为可训练的变量(也称为梯度变量)。使用Variable()函数可以方便地更新神经网络中的参数，从而实现反向传播算法。
  pred = model(inputs)
  _,pred = torch.max(pred, 1) #找到 tensor里最大的值，torch.max(input, dim)，input是softmax函数输出的一个tensor。 dim是max函数索引的维度0/1，0是每列的最大值，1是每行的最大值
  print("实际结果:",[i for i in Y_val])
  print("推理结果:", [ i for i in pred.data])

  img = torchvision.utils.make_grid(X_val)
  img = img.numpy().transpose(1,2,0)

  std = [0.5,0.5,0.5] #标准差，用于计算标准化后的每个通道的值。
  mean = [0.5,0.5,0.5] #均值，用于计算标准化后的每个通道的值。
  img = img*std+mean
  plt.imshow(img)
  plt.show()
  

def run_train():
  model = MNISTConvNet().to(DEVICE)
  optimizer = optim.Adam(model.parameters()) #Adam方法更新权重参数
  # optimizer = optim.SGD(model.parameters(), lr=1e-1, momentum = 0.9) #SGD方法更新权重参数,学习率设置大一点，过小梯度下降过慢

  for epoch in range(1, EPOCHS + 1):
    train(model, optimizer, epoch)
    torch.save(model.state_dict(), 'mnist_model.pkl') #保存模型
  test(model)
  

def run_predict():
  model = MNISTConvNet().to(DEVICE)
  model.load_state_dict(torch.load('mnist_model.pkl')) #加载之前训练好的模型
  predict(model)

def show_image():
  train_loader = torch.utils.data.DataLoader(
    datasets.MNIST('../datasets/data', train=True, download=True,
                   transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize((0.1307,), (0.3081,))  # transforms.Normalize()将数据进行归一化处理，
                   ])),
    batch_size=64, shuffle=True)
  X_train, Y_train = next(iter(train_loader)) #循环Iterator对象迭代器，一般和上面的iter()一起使用
  print("Image Label is:",[i for i in Y_train])

  img = torchvision.utils.make_grid(X_train)
  img = img.numpy().transpose(1,2,0)

  std = [0.5,0.5,0.5] #标准差，用于计算标准化后的每个通道的值。
  mean = [0.5,0.5,0.5] #均值，用于计算标准化后的每个通道的值。
  img = img*std+mean
  plt.imshow(img)
  plt.show()

if __name__ == '__main__':
  if len(sys.argv) <= 1:
    print("请输入参数：train|predict|img") #train:表示训练数据，predict#表示推理，#img只是单纯地显示一下图片
  else:
    action = sys.argv[1]
    if action == "train":
      run_train() # 用于训练数据
    elif action == "predict":
      run_predict() # 用于预测图片
    elif action == "img":
      show_image() # 只是简单地显示图片
    else:
      print("请输入参数：train|predict|img")

```
### 3、训练
上面代码首先使用Adam方法理更新权重参数，经过5个epoch后，准确率可以达到99.02%，如下
训练时使用如下代码训练：
```
python mnist_conv_net.py train
```
训练日志：
```
Train Epoch: 1 [5800/60000 (10%)] Loss: 0.489694
Train Epoch: 1 [11800/60000 (20%)]  Loss: 0.211527
Train Epoch: 1 [17800/60000 (30%)]  Loss: 0.184858
Train Epoch: 1 [23800/60000 (40%)]  Loss: 0.161435
Train Epoch: 1 [29800/60000 (50%)]  Loss: 0.173204
Train Epoch: 1 [35800/60000 (60%)]  Loss: 0.124986
Train Epoch: 1 [41800/60000 (70%)]  Loss: 0.076274
Train Epoch: 1 [47800/60000 (80%)]  Loss: 0.053319
Train Epoch: 1 [53800/60000 (90%)]  Loss: 0.044781
Train Epoch: 1 [59800/60000 (100%)] Loss: 0.063047
Train Epoch: 2 [5800/60000 (10%)] Loss: 0.077021
Train Epoch: 2 [11800/60000 (20%)]  Loss: 0.023402
Train Epoch: 2 [17800/60000 (30%)]  Loss: 0.050658
Train Epoch: 2 [23800/60000 (40%)]  Loss: 0.076794
Train Epoch: 2 [29800/60000 (50%)]  Loss: 0.058776
Train Epoch: 2 [35800/60000 (60%)]  Loss: 0.035746
Train Epoch: 2 [41800/60000 (70%)]  Loss: 0.093057
Train Epoch: 2 [47800/60000 (80%)]  Loss: 0.085911
Train Epoch: 2 [53800/60000 (90%)]  Loss: 0.066878
Train Epoch: 2 [59800/60000 (100%)] Loss: 0.120800
Train Epoch: 3 [5800/60000 (10%)] Loss: 0.053668
Train Epoch: 3 [11800/60000 (20%)]  Loss: 0.010750
Train Epoch: 3 [17800/60000 (30%)]  Loss: 0.029436
Train Epoch: 3 [23800/60000 (40%)]  Loss: 0.026253
Train Epoch: 3 [29800/60000 (50%)]  Loss: 0.026762
Train Epoch: 3 [35800/60000 (60%)]  Loss: 0.015479
Train Epoch: 3 [41800/60000 (70%)]  Loss: 0.109069
Train Epoch: 3 [47800/60000 (80%)]  Loss: 0.054719
Train Epoch: 3 [53800/60000 (90%)]  Loss: 0.034847
Train Epoch: 3 [59800/60000 (100%)] Loss: 0.019372
Train Epoch: 4 [5800/60000 (10%)] Loss: 0.025633
Train Epoch: 4 [11800/60000 (20%)]  Loss: 0.034000
Train Epoch: 4 [17800/60000 (30%)]  Loss: 0.016235
Train Epoch: 4 [23800/60000 (40%)]  Loss: 0.026829
Train Epoch: 4 [29800/60000 (50%)]  Loss: 0.070490
Train Epoch: 4 [35800/60000 (60%)]  Loss: 0.012940
Train Epoch: 4 [41800/60000 (70%)]  Loss: 0.017831
Train Epoch: 4 [47800/60000 (80%)]  Loss: 0.010007
Train Epoch: 4 [53800/60000 (90%)]  Loss: 0.014664
Train Epoch: 4 [59800/60000 (100%)] Loss: 0.039249
Train Epoch: 5 [5800/60000 (10%)] Loss: 0.052612
Train Epoch: 5 [11800/60000 (20%)]  Loss: 0.045722
Train Epoch: 5 [17800/60000 (30%)]  Loss: 0.012584
Train Epoch: 5 [23800/60000 (40%)]  Loss: 0.013080
Train Epoch: 5 [29800/60000 (50%)]  Loss: 0.033559
Train Epoch: 5 [35800/60000 (60%)]  Loss: 0.026182
Train Epoch: 5 [41800/60000 (70%)]  Loss: 0.001794
Train Epoch: 5 [47800/60000 (80%)]  Loss: 0.058093
Train Epoch: 5 [53800/60000 (90%)]  Loss: 0.038260
Train Epoch: 5 [59800/60000 (100%)] Loss: 0.012027

Test set: Average loss: 0.0290, Accuracy: 9902/10000 (99.02%)
```
可以再试下使用SGD来更新参数，将上面的Adam换成SGD
```
optimizer = optim.SGD(model.parameters(), lr=1e-1, momentum = 0.9) #SGD方法更新权重参数,学习率设置大一点，过小梯度下降过慢
```
这里注意他的学习率，我一开始使用的是1e-3的学习率，结果发现梯度下降很慢，10epoch下来，loss值还有0.2多，精确度只有86%,把他改成1e-1，发现效果好很多，下面是他的训练日志：
```
Train Epoch: 1 [5800/60000 (10%)] Loss: 0.6326162815093994
Train Epoch: 1 [11800/60000 (20%)]  Loss: 0.14702695608139038
Train Epoch: 1 [17800/60000 (30%)]  Loss: 0.11106981337070465
Train Epoch: 1 [23800/60000 (40%)]  Loss: 0.11699119210243225
Train Epoch: 1 [29800/60000 (50%)]  Loss: 0.08016740530729294
Train Epoch: 1 [35800/60000 (60%)]  Loss: 0.043048132210969925
Train Epoch: 1 [41800/60000 (70%)]  Loss: 0.07454550266265869
Train Epoch: 1 [47800/60000 (80%)]  Loss: 0.03606884181499481
Train Epoch: 1 [53800/60000 (90%)]  Loss: 0.018314605578780174
Train Epoch: 1 [59800/60000 (100%)] Loss: 0.032142993062734604
Train Epoch: 2 [5800/60000 (10%)] Loss: 0.017602885141968727
Train Epoch: 2 [11800/60000 (20%)]  Loss: 0.02773950807750225
Train Epoch: 2 [17800/60000 (30%)]  Loss: 0.09004029631614685
Train Epoch: 2 [23800/60000 (40%)]  Loss: 0.02125852182507515
Train Epoch: 2 [29800/60000 (50%)]  Loss: 0.0772605612874031
Train Epoch: 2 [35800/60000 (60%)]  Loss: 0.03700004518032074
Train Epoch: 2 [41800/60000 (70%)]  Loss: 0.05076766386628151
Train Epoch: 2 [47800/60000 (80%)]  Loss: 0.03611752390861511
Train Epoch: 2 [53800/60000 (90%)]  Loss: 0.04180872440338135
Train Epoch: 2 [59800/60000 (100%)] Loss: 0.024729833006858826
Train Epoch: 3 [5800/60000 (10%)] Loss: 0.08424612879753113
Train Epoch: 3 [11800/60000 (20%)]  Loss: 0.004190818872302771
Train Epoch: 3 [17800/60000 (30%)]  Loss: 0.035337768495082855
Train Epoch: 3 [23800/60000 (40%)]  Loss: 0.0022824255283921957
Train Epoch: 3 [29800/60000 (50%)]  Loss: 0.03864956274628639
Train Epoch: 3 [35800/60000 (60%)]  Loss: 0.022920949384570122
Train Epoch: 3 [41800/60000 (70%)]  Loss: 0.007762896828353405
Train Epoch: 3 [47800/60000 (80%)]  Loss: 0.02902868203818798
Train Epoch: 3 [53800/60000 (90%)]  Loss: 0.0689283162355423
Train Epoch: 3 [59800/60000 (100%)] Loss: 0.013603786006569862
Train Epoch: 4 [5800/60000 (10%)] Loss: 0.002816196996718645
Train Epoch: 4 [11800/60000 (20%)]  Loss: 0.015283504500985146
Train Epoch: 4 [17800/60000 (30%)]  Loss: 0.023883331567049026
Train Epoch: 4 [23800/60000 (40%)]  Loss: 0.012441331520676613
Train Epoch: 4 [29800/60000 (50%)]  Loss: 0.010721658356487751
Train Epoch: 4 [35800/60000 (60%)]  Loss: 0.008633618243038654
Train Epoch: 4 [41800/60000 (70%)]  Loss: 0.02001149021089077
Train Epoch: 4 [47800/60000 (80%)]  Loss: 0.008363565430045128
Train Epoch: 4 [53800/60000 (90%)]  Loss: 0.052691176533699036
Train Epoch: 4 [59800/60000 (100%)] Loss: 0.0459044873714447
Train Epoch: 5 [5800/60000 (10%)] Loss: 0.028606999665498734
Train Epoch: 5 [11800/60000 (20%)]  Loss: 0.009156718850135803
Train Epoch: 5 [17800/60000 (30%)]  Loss: 0.003058604197576642
Train Epoch: 5 [23800/60000 (40%)]  Loss: 0.0006700430531054735
Train Epoch: 5 [29800/60000 (50%)]  Loss: 0.010339883156120777
Train Epoch: 5 [35800/60000 (60%)]  Loss: 0.02346840687096119
Train Epoch: 5 [41800/60000 (70%)]  Loss: 0.01867305487394333
Train Epoch: 5 [47800/60000 (80%)]  Loss: 0.00615763058885932
Train Epoch: 5 [53800/60000 (90%)]  Loss: 0.026964306831359863
Train Epoch: 5 [59800/60000 (100%)] Loss: 0.03355356678366661
Train Epoch: 6 [5800/60000 (10%)] Loss: 0.0019284432055428624
Train Epoch: 6 [11800/60000 (20%)]  Loss: 0.006681244820356369
Train Epoch: 6 [17800/60000 (30%)]  Loss: 0.026242844760417938
Train Epoch: 6 [23800/60000 (40%)]  Loss: 0.029215598478913307
Train Epoch: 6 [29800/60000 (50%)]  Loss: 0.03575825318694115
Train Epoch: 6 [35800/60000 (60%)]  Loss: 0.007471080869436264
Train Epoch: 6 [41800/60000 (70%)]  Loss: 0.010791837237775326
Train Epoch: 6 [47800/60000 (80%)]  Loss: 0.0031418739818036556
Train Epoch: 6 [53800/60000 (90%)]  Loss: 0.005480702966451645
Train Epoch: 6 [59800/60000 (100%)] Loss: 0.0001745843474054709
Train Epoch: 7 [5800/60000 (10%)] Loss: 0.011564110405743122
Train Epoch: 7 [11800/60000 (20%)]  Loss: 0.029219292104244232
Train Epoch: 7 [17800/60000 (30%)]  Loss: 0.0013832019176334143
Train Epoch: 7 [23800/60000 (40%)]  Loss: 0.012693165801465511
Train Epoch: 7 [29800/60000 (50%)]  Loss: 0.0006698016077280045
Train Epoch: 7 [35800/60000 (60%)]  Loss: 0.0018981300527229905
Train Epoch: 7 [41800/60000 (70%)]  Loss: 0.010488353669643402
Train Epoch: 7 [47800/60000 (80%)]  Loss: 0.0014791633002460003
Train Epoch: 7 [53800/60000 (90%)]  Loss: 0.029287874698638916
Train Epoch: 7 [59800/60000 (100%)] Loss: 0.029237637296319008
Train Epoch: 8 [5800/60000 (10%)] Loss: 0.0069076246581971645
Train Epoch: 8 [11800/60000 (20%)]  Loss: 0.0002714076545089483
Train Epoch: 8 [17800/60000 (30%)]  Loss: 0.003580649383366108
Train Epoch: 8 [23800/60000 (40%)]  Loss: 0.0044055297039449215
Train Epoch: 8 [29800/60000 (50%)]  Loss: 0.005105248186737299
Train Epoch: 8 [35800/60000 (60%)]  Loss: 0.013003651052713394
Train Epoch: 8 [41800/60000 (70%)]  Loss: 0.002714156173169613
Train Epoch: 8 [47800/60000 (80%)]  Loss: 0.0037790806964039803
Train Epoch: 8 [53800/60000 (90%)]  Loss: 0.000915569078642875
Train Epoch: 8 [59800/60000 (100%)] Loss: 0.0026095432695001364
Train Epoch: 9 [5800/60000 (10%)] Loss: 0.002073458628728986
Train Epoch: 9 [11800/60000 (20%)]  Loss: 0.005496991332620382
Train Epoch: 9 [17800/60000 (30%)]  Loss: 0.009597436524927616
Train Epoch: 9 [23800/60000 (40%)]  Loss: 0.0016645310679450631
Train Epoch: 9 [29800/60000 (50%)]  Loss: 0.0006376765668392181
Train Epoch: 9 [35800/60000 (60%)]  Loss: 0.0005055389483459294
Train Epoch: 9 [41800/60000 (70%)]  Loss: 0.0015127335209399462
Train Epoch: 9 [47800/60000 (80%)]  Loss: 0.0017895273631438613
Train Epoch: 9 [53800/60000 (90%)]  Loss: 0.00020351675630081445
Train Epoch: 9 [59800/60000 (100%)] Loss: 0.003212581854313612
Train Epoch: 10 [5800/60000 (10%)]  Loss: 0.00015950168017297983
Train Epoch: 10 [11800/60000 (20%)] Loss: 0.0013000047765672207
Train Epoch: 10 [17800/60000 (30%)] Loss: 0.0015988206723704934
Train Epoch: 10 [23800/60000 (40%)] Loss: 0.0006124568171799183
Train Epoch: 10 [29800/60000 (50%)] Loss: 0.013285879977047443
Train Epoch: 10 [35800/60000 (60%)] Loss: 0.00040567651740275323
Train Epoch: 10 [41800/60000 (70%)] Loss: 0.00692130858078599
Train Epoch: 10 [47800/60000 (80%)] Loss: 0.0020271940156817436
Train Epoch: 10 [53800/60000 (90%)] Loss: 0.00022073769650887698
Train Epoch: 10 [59800/60000 (100%)]  Loss: 0.002098886761814356

Test set: Average loss: 0.0360, Accuracy: 9916/10000 (99.16%)
```
#### 4、预测
使用如下代码进行预测
```
python mnist_conv_net.py predict
```
可以看到输出结果：
```
实际结果: [tensor(8), tensor(7), tensor(4), tensor(7)]
推理结果: [tensor(8), tensor(7), tensor(4), tensor(7)]
```
对应的图片显示
![](http://blog.1nongfu.com/16850290270582.jpg)

#### 5、总结
这是一个很简单的分类网络，数据集也很简单，网络的架构有点像LeNet-5网络，不过99.16%的结果只能算一般般，还有优化的空间，目前网上准确率最高的有99.79%,所以还有提供的空间。
