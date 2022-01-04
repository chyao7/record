# AlexNet

### 提出relu激活函数，速度更快。

### 局部响应归一化
用激活函数将神经元的输出做一个非线性映射，
但是tanh和sigmoid这些传统的激活函数的值域都是有范围的，
但是ReLU激活函数得到的值域没有一个区间，
所以要对ReLU得到的结果进行归一化。

### 覆盖的池化操作

一般的池化层因为没有重叠，所以pool_size 和 stride一般是相等的，例如8 × 8的一个图像，如果池化层的尺寸是2 × 2 ，那么经过池化后的操作得到的图像是 4 × 4 大小，这种设置叫做不覆盖的池化操作，如果 stride < pool_size, 那么就会产生覆盖的池化操作，这种有点类似于convolutional化的操作，这样可以得到更准确的结果。在top-1，和top-5中使用覆盖的池化操作分别将error rate降低了0.4%和0.3%。论文中说，在训练模型过程中，覆盖的池化层更不容易过拟合

### dropout 
Dropout是在全连接层中去掉了一些神经节点，达到防止过拟合的目的，我们可以看上面的图在第六层和第七层都设置了Dropout。


```python
#model.py

import torch.nn as nn
import torch


class AlexNet(nn.Module):
    def __init__(self, num_classes=1000, init_weights=False):   
        super(AlexNet, self).__init__()
        self.features = nn.Sequential(  #打包
            nn.Conv2d(3, 48, kernel_size=11, stride=4, padding=2),  # input[3, 224, 224]  output[48, 55, 55] 自动舍去小数点后
            nn.ReLU(inplace=True), #inplace 可以载入更大模型
            nn.MaxPool2d(kernel_size=3, stride=2),                  # output[48, 27, 27] kernel_num为原论文一半
            nn.Conv2d(48, 128, kernel_size=5, padding=2),           # output[128, 27, 27]
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),                  # output[128, 13, 13]
            nn.Conv2d(128, 192, kernel_size=3, padding=1),          # output[192, 13, 13]
            nn.ReLU(inplace=True),
            nn.Conv2d(192, 192, kernel_size=3, padding=1),          # output[192, 13, 13]
            nn.ReLU(inplace=True),
            nn.Conv2d(192, 128, kernel_size=3, padding=1),          # output[128, 13, 13]
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),                  # output[128, 6, 6]
        )
        self.classifier = nn.Sequential(
            nn.Dropout(p=0.5),
            #全链接
            nn.Linear(128 * 6 * 6, 2048),
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(2048, 2048),
            nn.ReLU(inplace=True),
            nn.Linear(2048, num_classes),
        )
        if init_weights:
            self._initialize_weights()

    def forward(self, x):
        x = self.features(x)
        x = torch.flatten(x, start_dim=1) #展平   或者view()
        x = self.classifier(x)
        return x

    def _initialize_weights(self):
        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu') #何教授方法
                if m.bias is not None:
                    nn.init.constant_(m.bias, 0)
            elif isinstance(m, nn.Linear):
                nn.init.normal_(m.weight, 0, 0.01)  #正态分布赋值
                nn.init.constant_(m.bias, 0)

```