# 使用ChatGPT处理excel

```diff
```

不要盲目信任答案，有时候会存在bug，重要数据需要提前备份

```
```



\`#ff0000`

```markdown
The background color is `sadfasdfasdf` for light mode `#0969DA` xxxx `#ff0000`
```

## 1.表格数据生成

这里使用Chatbox提问。

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518222007.png)

然后等待返回答复：
![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518222105.png)

然后直接复制出现的内容到excel里面后调整格式即可。

## 2.生成excel函数

1. 求和

   提问：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518222645.png)

   返回答案：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518222709.png)

   经过在excel中测试正确。

2. 查询具体的值

   提问：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518223138.png)

   返回的答复：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518223222.png)

   经过在excel中测试正确。

## 3.高级技巧

1. 背景颜色处理

   提问：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518224253.png)

   返回的答案：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518224324.png)

   顺便问下怎么打开VBA编辑器：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518224357.png)

   在excel上操作后，结果正确。

2. 图形

   提问：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518225026.png)

   返回答案：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230518225053.png)

   将代码在excel中执行报错，在尝试了多次后仍然无法给出正确答案，因此切换到new bing来实现（有时候需要清理edge的cache才能正确访问new bing）：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230519000609.png)

   将合并后的代码在excel里面运行：

   ![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230519000537.png)

   可以看出结果是正确的，Y轴可能需要微调。

   推测可能是由于excel是office 365的版本，比较新，但是chatbot使用的ChatGPT是3.5的api，但是new bing已经是ChatGPT-4了，所以结果更匹配一些。
