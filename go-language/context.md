# Context

## 概念

Context是一种在跨API边界或在进程间携带截止时间、取消信号和其他请求域内的值的类型

进入服务的请求应当创建一个Context、服务流出的调用应该接收一个Context*？？* 这之间函数调用链必须传递这个Context，可以用一个由WithCancel, WithDeadline, WithTimeout, or WithValue派生的Context替代。当一个Context取消时，所有由它派生的Context也会取消

 The WithCancel, WithDeadline, and WithTimeout 这些函数接收一个Context（父）作为参数，返回一个派生Context（子）和它的取消函数，调用该取消函数会取消子Context和其子孙，并且删除父Context中对子Context的引用，关闭所有相关计时器。调用取消函数失败会导致子Context和其子孙泄露直到父Context取消或是计时器击发。

## 原则

为了保持包间接口的一致性并使检测context传递的静态分析工具有效，使用Context编程应遵守以下原则：

- 不要在结构体内保存Context，而是向每个需要的函数显式传递Context，应当在第一个参数，命名为ctx
- 不要传递一个nil Context，即使函数允许；当不确定应该用哪个Context时传递context.TODO
- 只在跨进程和API的请求域的数据使用context Values，不在用于向函数传递可选参数时使用
- 同一个Context可能被传递到不同的goroutines中，是并发安全的
- 