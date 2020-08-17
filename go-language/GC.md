# GO GC

并发标记清除算法with写屏障，增量线程、无分代、无压缩

### GC流程

1. GC 清扫结束阶段
   1. STW，所有P到达一个GC安全点
   2. 清扫未清扫的spans，（在预期时间前强制触发GC时会有未清扫spans）
2. GC 标记阶段
   1. 准备标记，设置`gcphace=_GCmark (from _GCoff)`，开启写屏障，开启辅助线程，将root对象入队，等待所有P开启写屏障完成STW
   2. StartTheWorld，开始标记，写屏障：将被覆盖的指针和新增的指针标记为灰色，新分配的对象直接标记为黑色（避免重新扫描栈）
   3. 标记，扫描所有根对象（栈、全局对象、指向堆的堆外运行时数据结构），扫描栈对象时会停止对应goroutine，标记所有指针为灰色，然后继续goroutine
   4. 扫描队列中所有灰色对象，标记为黑色，并将对象中指针标记为灰色入队
   5. 直到扫描结束（采用分布式结束算法检测是否标记结束）
3. GC 标记结束阶段
   1. STW（旧版本由于写屏障未对新增对象处理，需要重新扫描栈）
   2. 设置`gcphace=_GCmarktermination`，停止标记线程
   3. 打扫现场，如清空mcache
4. GC清扫阶段
   1. 设置`gcphace=_GCoff`，关闭写屏障
   2. StartTheWorld，在分配内存时清扫span，新增对象为白色
   3. 后台并发清扫
5. 分配对象增多时重新开始GC

#### 并发清扫

逐span 同时进行lazily清扫（用到时才清扫）和后台并发清扫（有线程逐span清扫）

#### 触发GC

1. 手动 runtime.GC()
2. GC rate，内存占用达到某个比例
3. 频繁分配内存

### 写屏障

结合Yuasa删除屏障（将删除指针的对象标记为灰色）和Dijkstra插入屏障（将插入的对象标记为灰色）的混合写屏障

```go
writePointer(slot, ptr):
	shade(*slot) //slot会被删除，将其指向的对象标记为灰色
	if current stack is grey:
		shade(ptr) //如果ptr的对象为白色会被清除，将其标记为灰色
	*slot = ptr
```



