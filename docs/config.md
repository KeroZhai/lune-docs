# 配置

配置文件的各个参数和含义如下：

```
[Application]
; 环境配置，dev（开发环境）或 prod（生产环境），默认为 dev。
; 如果为 prod，则路由映射关系将会被持久化到文件中，接口有变化时需要删除以便重新生成。
; 如果为 dev，则不会缓存路由映射关系以适应开发阶段接口的频繁变化。
env = dev
; 如果为 true, 则会返回 JSON 格式的结果，默认为 true。
echo-json = true

; 首选数据库配置
[Datasource]
; address =
; user =
; password =
; database =

[JSON]
; 如果为 true, 则转换为 JSON 时会忽略 null 值，默认为 true
non-null = true
```