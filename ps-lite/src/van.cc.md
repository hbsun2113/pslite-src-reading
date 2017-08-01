# Receiving

```
void Van::Receiving()
```

在source insight中没有看到调用者, 应该是用作回调函数了.

调用ZMQVan::RecvMsg来接收消息, 并解析消息.

# PackMeta和UnpackMeta

用Protobuf来序列化\反序列化Meta数据.

