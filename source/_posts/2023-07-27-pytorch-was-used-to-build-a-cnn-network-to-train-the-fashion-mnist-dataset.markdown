---
layout: post
title: "使用PyTorch构建CNN网络训练FashionMNIST数据集"
date: 2023-07-27 16:31:49 +0800
comments: true
categories: 
---
#### 前言
本文记录了使用PyTorch构建一个简单的CNN网络，并使用Fahion-MNIST数据集进行训练和测试。记录整个推理过程
<!--more-->

#### 1、Fashion-MNIST数据集
Fashion-MNIST是一个流行的图像分类数据集，用于机器学习和计算机视觉任务。数据集包含了来自10个不同类别的70,000个灰度图像。每个图像的分辨率为28x28像素，并包含单件服装或配饰的图像。这些类别包括：
```
1、T恤
2、裤子
3、毛衣
4、裙子
5、外套
6、凉鞋
7、衬衫
8、运动鞋
9、包
10、短靴
```
Fashion-MNIST数据集的目标是训练机器学习模型来准确地识别图像中的服装类别。由于这些图像是灰度图像，每个像素的值介于0到255之间。

该数据集通常用于测试图像分类算法的性能，以及在实践中验证新的计算机视觉技术。它与MNIST数据集类似，但更具挑战性，因为服装图像的变化更加复杂。

#### 2、构建卷积神经网络
由于Fashion-MINST数据集跟MINST数据集一样，都是28*28的灰度图片，所以他们的输入是一样的，我们可以直接按照上次训练MINST的卷积神经网络来训练Fashion-MINST，上一讲中训练MINST的网络结构如下(具体可参考上一篇文章)：
Conv1->Relu->pool->Conv2->Relu->pool->Linear1->Relu->Linear2->Sofrmax
使用上一讲的卷积神经网络训练，训练10个epoch后的准确率在86%左右，可以看出，准确率不是很高，现在改一下神经网络，来提高其准确率，主要是增加卷积层，上一讲中只有两个卷积层，现在我们可以再增加一个卷积层，来看下训练效果，代码如下：
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
EPOCHS = 5  # 总共训练批次
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")  # 让torch判断是否使用GPU，建议使用GPU环境，因为会快很多

train_loader = torch.utils.data.DataLoader(
    datasets.FashionMNIST('../datasets/data', train=True, download=True,
                   transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize(mean=[0.5],std=[0.5])  # transforms.Normalize()将数据进行归一化处理，
                   ])),
    batch_size=BATCH_SIZE, shuffle=True)

test_loader = torch.utils.data.DataLoader(
    datasets.FashionMNIST('../datasets/data', train=False, transform=transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.5],std=[0.5])
    ])),
    batch_size=BATCH_SIZE, shuffle=True)

# Class labels
classes = ('T恤', '牛仔裤', '毛衣', '裙子', '外套',
        '凉鞋', '衬衫', '运动鞋', '包', '短靴')

# 网络结构 conv->Relu->pool->conv->Relu->pool->Linear->Relu->Linear->Sofrmax
class MNISTConvNet(nn.Module):
  def __init__(self):
    super().__init__()
    # 公式：OH=(H + 2P - FH)/S + 1; OW= (W + 2P - FW)/S + 1
    self.conv1 = nn.Conv2d(in_channels=1, out_channels=24, kernel_size=3, stride=1, padding=0) # 12, 26*26
    self.conv2 = nn.Conv2d(in_channels=24, out_channels=48, kernel_size=3, stride=1, padding=0) # 24, 24*24
    self.conv3 = nn.Conv2d(in_channels=48, out_channels=96, kernel_size=3, stride=1, padding=0) # 48, 10*10
    self.fc1 = nn.Linear(in_features=96*5*5, out_features=400)
    self.fc2 = nn.Linear(in_features=400, out_features=10)
    
  def forward(self, x):
    out = self.conv1(x)
    out = F.relu(out)
    # out = F.max_pool2d(input=out, kernel_size=(2,2), stride=2) #12 13*13
    
    out = self.conv2(out)
    out = F.relu(out)
    out = F.max_pool2d(input=out, kernel_size=(2,2), stride=2) #24 12*12
    
    out = self.conv3(out)
    out = F.relu(out)
    out = F.max_pool2d(input=out, kernel_size=(2,2), stride=2) #48 5*5
    
    out = out.view(-1,96*5*5)
    out = self.fc1(out)
    out = F.relu(out)
    
    out = self.fc2(out)
    
    out = F.log_softmax(out, dim=1)
    return out

def train(model, optimizer, epoch):
  model.train() #设置模型为训练模式
  train_loss = 0
  correct = 0
  for batch_id, (images, labels) in enumerate(train_loader):
    images = images.to(DEVICE) # 将tensors移动到配置的device
    labels = labels.to(DEVICE)
    optimizer.zero_grad()  # 在反向传播的时候先把梯度记录清0
    output = model(images)
    loss = F.nll_loss(output, labels)
    loss.backward()
    optimizer.step()  # 调用step()方法更新梯度参数
    if (batch_id + 1) % 30 == 0:
        print(f'Train Epoch: {epoch} [{batch_id * len(images)}/{len(train_loader.dataset)} ({ 100. * batch_id / len(train_loader):.0f}%)]\tLoss: {loss.item()}')
    train_loss += loss.item()
    # print(f'train loss: {loss.item()}, train_loss_sum: {train_loss}')
    
    pred = output.max(1, keepdim=True)[1]  # 找到概率最大的下标
    correct += pred.eq(labels.view_as(pred)).sum().item()
  return 100. * train_loss / len(train_loader.dataset), 100. * correct / len(train_loader.dataset)
  
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
  print(f'\nTest set: Average loss: {test_loss:.4f}, Accuracy: {correct}/{len(test_loader.dataset)} ({100. * correct / len(test_loader.dataset)}%)\n')
  return test_loss, 100. * correct / len(test_loader.dataset)

def predict(model):
  val_loader = torch.utils.data.DataLoader(
    datasets.FashionMNIST('../datasets/data', train=False, transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize(mean=[0.5],std=[0.5])
                   ])),batch_size=8, shuffle=True)
  iterator = iter(val_loader)#获得Iterator对象:
  X_val, Y_val = next(iterator) #循环Iterator对象迭代器，一般和上面的iter()一起使用
  inputs = Variable(X_val)
  #Variable()函数用于将一个张量转换为可训练的变量(也称为梯度变量)。使用Variable()函数可以方便地更新神经网络中的参数，从而实现反向传播算法。
  pred = model(inputs)
  _,pred = torch.max(pred, 1) #找到 tensor里最大的值，torch.max(input, dim)，input是softmax函数输出的一个tensor。 dim是max函数索引的维度0/1，0是每列的最大值，1是每行的最大值
  print("实际结果:",[classes[i] for i in Y_val])
  print("推理结果:", [ classes[i] for i in pred.data])

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
  train_loss_list, test_loss_list = [],[]
  train_acc_list, test_acc_list = [],[]
  for epoch in range(1, EPOCHS + 1):
    train_loss, train_acc =  train(model, optimizer, epoch)
    #torch.save(model.state_dict(), 'fashion_mnist_model.pkl') #保存模型
    test_loss, test_acc = test(model)
    train_acc_list.append(train_acc)
    test_acc_list.append(test_acc)
    train_loss_list.append(train_loss)
    test_loss_list.append(test_loss)
    print(f'Epoch: {epoch} Train Loss: {train_loss:.4f}, Train Acc.: {train_acc:.2f}% | Validation Loss: {test_loss:.4f}, Validation Acc.: {test_acc:.2f}%')
  
  plt.plot(range(1, EPOCHS+1), train_loss_list, label='Training loss')
  plt.plot(range(1, EPOCHS+1), test_loss_list, label='Validation loss')
  plt.legend(loc='upper right')
  plt.ylabel('Cross entropy')
  plt.xlabel('Epoch')
  plt.show()
  

def run_predict():
  model = MNISTConvNet().to(DEVICE)
  model.load_state_dict(torch.load('fashion_mnist_model.pkl')) #加载之前训练好的模型
  predict(model)

def show_image():
  train_loader = torch.utils.data.DataLoader(
    datasets.FashionMNIST('../datasets/data', train=True, download=True,
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
      print("-----")
      run_predict() # 用于预测图片
    elif action == "img":
      show_image() # 只是简单地显示图片
    else:
      print("请输入参数：train|predict|img")
  

```
可以看到与上一讲的MINST网络的主要区别是增加一个卷积层。
##### 3、训练
运行10epoch后的训练结果如下：
```
Train Epoch: 1 [5800/60000 (10%)] Loss: 0.6379690766334534
Train Epoch: 1 [11800/60000 (20%)]  Loss: 0.5845944881439209
Train Epoch: 1 [17800/60000 (30%)]  Loss: 0.4599098265171051
Train Epoch: 1 [23800/60000 (40%)]  Loss: 0.512351393699646
Train Epoch: 1 [29800/60000 (50%)]  Loss: 0.40181490778923035
Train Epoch: 1 [35800/60000 (60%)]  Loss: 0.3760054409503937
Train Epoch: 1 [41800/60000 (70%)]  Loss: 0.36197417974472046
Train Epoch: 1 [47800/60000 (80%)]  Loss: 0.3655475676059723
Train Epoch: 1 [53800/60000 (90%)]  Loss: 0.40860098600387573
Train Epoch: 1 [59800/60000 (100%)] Loss: 0.38368210196495056
Train Epoch: 2 [5800/60000 (10%)] Loss: 0.3502006232738495
Train Epoch: 2 [11800/60000 (20%)]  Loss: 0.2745484411716461
Train Epoch: 2 [17800/60000 (30%)]  Loss: 0.29497388005256653
Train Epoch: 2 [23800/60000 (40%)]  Loss: 0.30289018154144287
Train Epoch: 2 [29800/60000 (50%)]  Loss: 0.3070503771305084
Train Epoch: 2 [35800/60000 (60%)]  Loss: 0.34184956550598145
Train Epoch: 2 [41800/60000 (70%)]  Loss: 0.25298836827278137
Train Epoch: 2 [47800/60000 (80%)]  Loss: 0.2620333731174469
Train Epoch: 2 [53800/60000 (90%)]  Loss: 0.2306962013244629
Train Epoch: 2 [59800/60000 (100%)] Loss: 0.3607088327407837
Train Epoch: 3 [5800/60000 (10%)] Loss: 0.22903680801391602
Train Epoch: 3 [11800/60000 (20%)]  Loss: 0.20266905426979065
Train Epoch: 3 [17800/60000 (30%)]  Loss: 0.25112783908843994
Train Epoch: 3 [23800/60000 (40%)]  Loss: 0.22732162475585938
Train Epoch: 3 [29800/60000 (50%)]  Loss: 0.2040642946958542
Train Epoch: 3 [35800/60000 (60%)]  Loss: 0.20411935448646545
Train Epoch: 3 [41800/60000 (70%)]  Loss: 0.19729731976985931
Train Epoch: 3 [47800/60000 (80%)]  Loss: 0.24248278141021729
Train Epoch: 3 [53800/60000 (90%)]  Loss: 0.21713067591190338
Train Epoch: 3 [59800/60000 (100%)] Loss: 0.21899420022964478
Train Epoch: 4 [5800/60000 (10%)] Loss: 0.1833008974790573
Train Epoch: 4 [11800/60000 (20%)]  Loss: 0.1431916356086731
Train Epoch: 4 [17800/60000 (30%)]  Loss: 0.2693891227245331
Train Epoch: 4 [23800/60000 (40%)]  Loss: 0.21619722247123718
Train Epoch: 4 [29800/60000 (50%)]  Loss: 0.22982168197631836
Train Epoch: 4 [35800/60000 (60%)]  Loss: 0.26103365421295166
Train Epoch: 4 [41800/60000 (70%)]  Loss: 0.23812612891197205
Train Epoch: 4 [47800/60000 (80%)]  Loss: 0.25694453716278076
Train Epoch: 4 [53800/60000 (90%)]  Loss: 0.2516362965106964
Train Epoch: 4 [59800/60000 (100%)] Loss: 0.19944973289966583
Train Epoch: 5 [5800/60000 (10%)] Loss: 0.20240060985088348
Train Epoch: 5 [11800/60000 (20%)]  Loss: 0.17876243591308594
Train Epoch: 5 [17800/60000 (30%)]  Loss: 0.1623563915491104
Train Epoch: 5 [23800/60000 (40%)]  Loss: 0.14842358231544495
Train Epoch: 5 [29800/60000 (50%)]  Loss: 0.1817866861820221
Train Epoch: 5 [35800/60000 (60%)]  Loss: 0.1474132388830185
Train Epoch: 5 [41800/60000 (70%)]  Loss: 0.20310136675834656
Train Epoch: 5 [47800/60000 (80%)]  Loss: 0.2332427054643631
Train Epoch: 5 [53800/60000 (90%)]  Loss: 0.14285261929035187
Train Epoch: 5 [59800/60000 (100%)] Loss: 0.16876384615898132
Train Epoch: 6 [5800/60000 (10%)] Loss: 0.15261796116828918
Train Epoch: 6 [11800/60000 (20%)]  Loss: 0.12889760732650757
Train Epoch: 6 [17800/60000 (30%)]  Loss: 0.17289716005325317
Train Epoch: 6 [23800/60000 (40%)]  Loss: 0.12383649498224258
Train Epoch: 6 [29800/60000 (50%)]  Loss: 0.14670918881893158
Train Epoch: 6 [35800/60000 (60%)]  Loss: 0.20221909880638123
Train Epoch: 6 [41800/60000 (70%)]  Loss: 0.21042735874652863
Train Epoch: 6 [47800/60000 (80%)]  Loss: 0.16009800136089325
Train Epoch: 6 [53800/60000 (90%)]  Loss: 0.23655852675437927
Train Epoch: 6 [59800/60000 (100%)] Loss: 0.1507928967475891
Train Epoch: 7 [5800/60000 (10%)] Loss: 0.16259260475635529
Train Epoch: 7 [11800/60000 (20%)]  Loss: 0.14753393828868866
Train Epoch: 7 [17800/60000 (30%)]  Loss: 0.15315169095993042
Train Epoch: 7 [23800/60000 (40%)]  Loss: 0.1445481777191162
Train Epoch: 7 [29800/60000 (50%)]  Loss: 0.132000133395195
Train Epoch: 7 [35800/60000 (60%)]  Loss: 0.09024682641029358
Train Epoch: 7 [41800/60000 (70%)]  Loss: 0.15115217864513397
Train Epoch: 7 [47800/60000 (80%)]  Loss: 0.1311248242855072
Train Epoch: 7 [53800/60000 (90%)]  Loss: 0.10016605257987976
Train Epoch: 7 [59800/60000 (100%)] Loss: 0.12980304658412933
Train Epoch: 8 [5800/60000 (10%)] Loss: 0.12055137008428574
Train Epoch: 8 [11800/60000 (20%)]  Loss: 0.0916302502155304
Train Epoch: 8 [17800/60000 (30%)]  Loss: 0.14020071923732758
Train Epoch: 8 [23800/60000 (40%)]  Loss: 0.1205180287361145
Train Epoch: 8 [29800/60000 (50%)]  Loss: 0.1326950639486313
Train Epoch: 8 [35800/60000 (60%)]  Loss: 0.19008560478687286
Train Epoch: 8 [41800/60000 (70%)]  Loss: 0.1668577790260315
Train Epoch: 8 [47800/60000 (80%)]  Loss: 0.1530739814043045
Train Epoch: 8 [53800/60000 (90%)]  Loss: 0.13995856046676636
Train Epoch: 8 [59800/60000 (100%)] Loss: 0.05448155477643013
Train Epoch: 9 [5800/60000 (10%)] Loss: 0.09004207700490952
Train Epoch: 9 [11800/60000 (20%)]  Loss: 0.08474709838628769
Train Epoch: 9 [17800/60000 (30%)]  Loss: 0.12305212020874023
Train Epoch: 9 [23800/60000 (40%)]  Loss: 0.08823414891958237
Train Epoch: 9 [29800/60000 (50%)]  Loss: 0.07393839210271835
Train Epoch: 9 [35800/60000 (60%)]  Loss: 0.11944027245044708
Train Epoch: 9 [41800/60000 (70%)]  Loss: 0.086588054895401
Train Epoch: 9 [47800/60000 (80%)]  Loss: 0.08738173544406891
Train Epoch: 9 [53800/60000 (90%)]  Loss: 0.1982622593641281
Train Epoch: 9 [59800/60000 (100%)] Loss: 0.11448238044977188
Train Epoch: 10 [5800/60000 (10%)]  Loss: 0.0850382074713707
Train Epoch: 10 [11800/60000 (20%)] Loss: 0.06795746088027954
Train Epoch: 10 [17800/60000 (30%)] Loss: 0.1085655689239502
Train Epoch: 10 [23800/60000 (40%)] Loss: 0.05534665286540985
Train Epoch: 10 [29800/60000 (50%)] Loss: 0.09382271021604538
Train Epoch: 10 [35800/60000 (60%)] Loss: 0.026057429611682892
Train Epoch: 10 [41800/60000 (70%)] Loss: 0.07035879790782928
Train Epoch: 10 [47800/60000 (80%)] Loss: 0.1626715511083603
Train Epoch: 10 [53800/60000 (90%)] Loss: 0.08846493810415268
Train Epoch: 10 [59800/60000 (100%)]  Loss: 0.07113683223724365

Test set: Average loss: 0.2453, Accuracy: 9204/10000 (92.04%)
```
可以看到现在准确率达到了92.04%，可见适当增加卷积层可以提高识别精度

#### 4、预测
使用命令预测一下
```
 python fashion_mnist_conv_net.py predict
```
![](http://blog.1nongfu.com/16871595198847.jpg)
```
实际结果: ['裙子', '外套', '凉鞋', '包', '包', '毛衣', '牛仔裤', '运动鞋']
推理结果: ['裙子', '外套', '凉鞋', '包', '包', '毛衣', '牛仔裤', '运动鞋']

```


