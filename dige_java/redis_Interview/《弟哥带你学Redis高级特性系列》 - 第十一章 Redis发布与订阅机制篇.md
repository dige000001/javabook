## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

Redis的发布与订阅功能由`PUBLISH`、`SUBSCRIBE`、`PSUBSCRIB`E等命令组成。

通过执行SUBSCRIBE命令，客户端可以订阅一个或多个频道，从而成为这些频道的订阅者(subscriber )：每当有其他客户端向被订阅的频道发送消息(message)时，频道的所有订阅者都会收到这条消息。

举个例子，假设A、B、C三个客户端都执行了命令

SUBSCRIBE "news.it"

那么这三个客户端就是"news.it”频道的订阅者，如图18-1所示。

如果这时某个客户端执行命令

PUBLISH "news.it" "hello"

向"news.it”频道发送消息"hello"，那么"news .it"的三个订阅者都将收到这条消息,如图18-2所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/397b798548c54a56b35038a87cb867cc~tplv-k3u1fbpfcp-zoom-1.image)



除了订阅频道之外，客户端还可以通过执行PSUBSCRIBE命令订阅一个或多个模式，从而成为这些模式的订阅者：每当有其他客户端向某个频道发送消息时，消息不仅会被发送给这个频道的所有订阅者，它还会被发送给所有与这个频道相匹配的模式的订阅者。

举个例子，假设如图18-3所示:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60f27be4ad164365b45467cb4c018083~tplv-k3u1fbpfcp-zoom-1.image)

-   客户端A正在订阅频道"news.it"。
-   客户端B正在订阅频道"news.et"。

<!---->

-   客户端C和客户端D正在订阅与"news.it"频道和"news.et”频道相匹配的模式"news. [ie]t"。

如果这时某个客户端执行命令

PUBLISH "news.it" "hello"

向"news.it"频道发送消息"hello"，那么不仅正在订阅"news.it"频道的客户端A会收到消息，客户端C和客户端D也同样会收到消息，因为这两个客户端正在订阅匹配"news.it”频道的"news. [ie]t”模式，如图18-4所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc7fd3185678471cb9a968d121c1dd5b~tplv-k3u1fbpfcp-zoom-1.image)

与此类似，如果某个客户端执行命令

PUBLISH "news.et""world"

向"news.et"频道发送消息"world"，那么不仅正在订阅"news.et"频道的客户端B会收到消息，客户端C和客户端D也同样会收到消息，因为这两个客户端正在订阅匹配"news.et”频道的"news. [ie]t”模式，如图18-5所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48a84fdb4d4b4804a316acc8cf5258cf~tplv-k3u1fbpfcp-zoom-1.image)

# 频道的订阅与退订

当一个**客户端**执行SUBSCRIBE命令订阅某个或某些频道的时候，这个客户端与被订阅频道之间就建立起了一种订阅关系。

Redis将所有频道的订阅关系都保存在服务器状态的pubsub_channels字典里面，这个字典的键是某个被订阅的频道，而键的值则是一个链表，链表里面记录了所有订阅这个 频道的客户端:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0ecee3035d1459b8cea3c1fad94cbdc~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abb695d0b0e34686a19b4de386c9f26b~tplv-k3u1fbpfcp-zoom-1.image)

## 订阅频道

每当客户端执行`SUBSCRIBE`命令订阅某个或某些频道的时候，服务器都会将客户端与被订阅的频道在pubsub_channels字典中进行关联。

根据频道是否已经有其他订阅者，关联操作分为两种情况执行:

-   -   如果频道已经有其他订阅者，那么它在pubsub_channels字典中必然有相应的订阅者链表，程序唯一要做的就是将客户端添加到订阅者链表的末尾。
    -   如果频道还未有任何订阅者，那么它必然不存在于pubsub_channels字典，程序首先要在pubsub_channels字典中为频道创建一个键，并将这个键的值设置为空链表，然后再将客户端添加到链表，成为链表的第一个元素。

## 退订频道

UNSUBSCRIBE命令的行为和SUBSCRIBE命令的行为正好相反，当一个客户端退订某个或某些频道的时候，服务器将从pubsub_channels 中解除客户端与被退订频道之间的关联:

-   程序会根据被退订频道的名字，在pubsub_channels字典中找到频道对应的订阅者链表，然后从订阅者链表中删除退订客户端的信息。
-   如果删除退订客户端之后，频道的订阅者链表变成了空链表，那么说明这个频道已经没有任何订阅者了，程序将从pubsub_channels字典中删除频道对应的键。

# 模式的订阅与退订

前面说过，服务器将所有频道的订阅关系都保存在服务器状态的pubsub_channels属性里面，与此类似，服务器也将所有模式的订阅关系都保存在服务器状态的pubsub_patterns属性里面:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/326d817dd1014cda9e15f079908440de~tplv-k3u1fbpfcp-zoom-1.image)

pubsub_patterns属性是一个链表，链表中的每个节点都包含着一个pubsubPattern结构，这个结构的pattern属性记录了被订阅的模式，而client属性则记录了订阅模式的客户端：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b206b75702c143268470285dc388ed11~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b03271680f84288a2c2f3299306567a~tplv-k3u1fbpfcp-zoom-1.image)

图18-11展示了一个pubsub_patterns链表示例,这个链表记录了以下信息:

-   客户端client-7正在订阅模式"music.*"。
-   客户端client-8正在订阅模式"book.*"。

<!---->

-   客户端client-9正在订阅模式"news.*"。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c9e5e4fef884bf4b4cec8311f6c42e1~tplv-k3u1fbpfcp-zoom-1.image)

## 订阅模式

每当客户端执行`PSUBSCRIBE`命令订阅某个或某些模式的时候，服务器会对每个被订阅的模式执行以下两个操作:

1.  新建一个pubsubPattern结构，将结构的pattern属性设置为被订阅的模式，client属性设置为订阅模式的客户端。
1.  将pubsubPattern结构添加到pubsub_patterns链表的表尾。

## 退订模式

模式的退订命令`PUNSUBSCRIBE`是 `PSUBSCRIBE`命令的反操作：当一个客户端退订某个或某些模式的时候，服务器将在pubsub_patterns链表中查找并删除那些pattern属性为被退订模式，并且client属性为执行退订命令的客户端的pubsubPattern结构。

# 发送消息

当一个Redis 客户端执行`PUBLISH <channel> <message>`命令将消息message发送给频道channel的时候，服务器需要执行以下两个动作:

1.  将消息message发送给channel频道的所有订阅者。
1.  如果有一个或多个模式pattern与频道channel相匹配，那么将消息message发送给pattern模式的订阅者。

## 将消息发送给频道订阅者

因为服务器状态中的pubsub_channels字典记录了所有频道的订阅关系，所以为了将消息发送给channel频道的所有订阅者，PUBLISH命令要做的就是在pubsub_channels字典里找到频道channel的订阅者名单(一个链表)，然后将消息发送给名单上的所有客户端。

## 将信息发给模式订阅者

因为服务器状态中的pubsub_patterns链表记录了所有模式的订阅关系，所以为了将消息发送给所有与channel频道相匹配的模式的订阅者，PUBLISH命令要做的就是遍历整个pubsub_patterns链表，查找那些与channel频道相匹配的模式，并将消息发送给订阅了这些模式的客户端。



