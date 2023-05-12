# 第1课 课后作业

## [ZKP 三色演示](https://zkshanghai.xyz/notes/exercise1.html#zkp-%E4%B8%89%E8%89%B2%E6%BC%94%E7%A4%BA)

### 练习1

如果可以选择任意节点对来检查，证明就不是零知识的了。验证者可以通过以下方式逆向着色方案：

1. 选择节点对，其中一点固定选择节点A，另一点遍历节点A之外的所有节点，这样能获取一个信息：即A之外的其他节点是否与A同色；
2. 继续选择节点对，其中一点固定在与节点A不同色的一点B，另一点遍历节点B之外的所有节点，这样能获取一个信息：即B之外的其他节点是否与B同色；
3. 假设节点A的颜色是1，节点B的颜色是2，剩下的一个颜色是3，那么，与节点A同色的节点颜色是1，与节点B同色的节点颜色是2，与节点A、B均不同色的节点颜色是3；

至此，着色方案被逆向破解

### 练习2

这是正确的等式。这是一个零知识证明，验证者需要在不知道实际着色方案的情况下检查着色是否正确，这可能是为什么没有提供先验的原因。

## zkmessage.xyz

- 这里的秘密是一个私钥，zk电路中会用这个私钥生成证明，同时这个证明不会暴露私钥本身。这个私钥在功能中代表了一个账户身份本身，泄露的话别人就能用这个账户发消息，这个账户就不是原有者私有的了，所以需要自己保存且保密；
- 证明一条信息是一组用户中的某个人发出的，但是不暴露是具体哪一个人；
- 使用用户名/密码模式的话，就需要在服务端保存和验证密码（hash），同时服务端也可以作恶，比如冒充用户发一条消息，同时用户无法否认这一条消息。而用秘密+zk这种形式，服务端只保存了公钥和twitter的对应关系，不触及用户的私钥，那么服务端即使冒充用户A发送了一条消息，其他用户也可以发现这条信息是冒充的，因为服务端没有A的私钥它是无法生成正确的证明的。