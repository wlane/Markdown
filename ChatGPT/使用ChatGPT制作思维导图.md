# 使用ChatGPT制作思维导图

```diff
- 不要盲目信任答案，有时候会存在bug，重要数据需要提前备份
```

## 1. 提问

首先尝试对chatgpt提问：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422223312.png)

可以看出这里描述的很笼统，对于一般小白而言，甚至都看不懂，我们可以使用更具体的提问方式：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422224056.png)

这里的使用方式也是仅限于api，需要借助开源的第三方工具来实现，比如我当前使用的这个。实际上chatgpt是可以直接在其官方页面上配置的：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422224106.png)

此时我发现使用的开源[ChuanhuChatGPT](https://github.com/GaiZhenbiao/ChuanhuChatGPT)不能很好的查看markdown格式，于是切换到了桌面版本的[chatbox](https://github.com/Bin-Huang/chatbox)：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422231213.png)

以及:

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422231228.png)



## 2.生成思维导图

有个chatgpt的回答之后，我们首先点击chatbot回答框右上角三个点里的编辑：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422231559.png)

得到以下内容：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422231653.png)

### 第一种方式：使用在线的markmap

复制内容，打开地址[markmap](https://markmap.js.org/repl)，注意删除无意义的开头和结尾：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422232141.png)

这种方式只适合要求比较简单的流程或者不正式的场合临时使用。

### 第二种方式：使用工具Xmind

创建一个md文件，将上面复制的内容复制进去，注意删除无意义的开头和结尾：：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422232719.png)

保存。打开Xmind：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422233338.png)

选择导入上面的文件：

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230422233459.png)

可以看到它自动生成了一个思维导图，之后也可以选择想要的其他模板。