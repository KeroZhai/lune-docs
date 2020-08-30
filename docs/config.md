# 配置

conf 目录下的 config.ini 文件用于对项目进行一些简单的配置：

``` ini
[Application]
; dev 或 prod
; 默认：dev。
; 如果为 prod，则路由映射关系将会被持久化到文件中，接口有变化时需要删除以便重新生成。
; 如果为 dev，则不会缓存路由映射关系以适应开发阶段接口的频繁变化。
; env = 

; true 或 false
; 默认：true
; 如果为 true, 则会将接口返回结果转为 JSON 格式
; echo-json = 

; 符合`glob()`方法参数规则的路径字符串，用于配置扫描接口文件时采用的路径模式，以`,`分隔
; 默认：/controller/*
; scan-pattern = 

; 首选数据库配置
[Datasource]
; address =
; user =
; password =
; database =

[JSON]
; true 或 false
; 默认：true
; 如果为 true, 则转换为 JSON 时会忽略 null 值
; non-null = 
```