## 简介

Apache Shiro是Java安全框架，提供了认证、授权、加密和会话管理等功能。

Shiro提供了记住密码的功能（rememberMe），有记住密码的功能的网站很可能使用了这个框架。

## 组件检测

抓包重放后如果回包显示rememberMe=deleteMe，证明使用了Shiro。

检测该组件是否存在漏洞时是没有回显的，因此需要用dnslog配合检测。