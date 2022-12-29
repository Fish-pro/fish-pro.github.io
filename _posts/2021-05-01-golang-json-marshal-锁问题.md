---
layout: post
title: golang json marshal 锁问题
author: Fish-pro
tags:
- marshal
- go
- golang
date: 2022-05-01 19:26 +0800
---
map操作时，明明已经加了读写锁，为什么还有并发安全问题，以下一个实例，乍一看没有问题，但是运行报错，`fatal error:concurrent map read and map write`

```golang
package main

import (
	"encoding/json"
	"fmt"
	"sync"
)

type obj struct {
	wg     sync.WaitGroup
	mu     sync.RWMutex
	resMap map[string]int
}

func main() {
	obj1 := &obj{
		wg:     sync.WaitGroup{},
		resMap: make(map[string]int),
	}
	obj1.wg.Add(1000)

	for i := 0; i < 500; i++ {
		go func(obj1 *obj, i int) {
			fmt.Println("write", i)
			obj1.set(fmt.Sprintf("%d", i), i)
			obj1.wg.Done()
		}(obj1, i)
	}
	for j := 0; j < 500; j++ {
		go func(obj1 *obj) {
			fmt.Println("read")
			res := obj1.get()
			json.Marshal(res)
			fmt.Println(len(res))
			obj1.wg.Done()
		}(obj1)
	}

	obj1.wg.Wait()
}

func (o *obj) get() map[string]int {
	o.mu.RLock()
	defer o.mu.RUnlock()
	return o.resMap
}

func (o *obj) set(key string, val int) {
	o.mu.Lock()
	defer o.mu.Unlock()
	o.resMap[key] = val
}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c39b5919e0e47808ff1fa079418bd57.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUHJvZmVzc29yIEZpc2g=,size_20,color_FFFFFF,t_70,g_se,x_16)
分析原因：map是引用传递，json.Marshal同样会触发map的读操作，所以会报错
修改后如下，json.Marshal时同样加读锁：
```golang
package main

import (
	"encoding/json"
	"fmt"
	"sync"
)

type obj struct {
	wg     sync.WaitGroup
	mu     sync.RWMutex
	resMap map[string]int
}

func main() {
	obj1 := &obj{
		wg:     sync.WaitGroup{},
		resMap: make(map[string]int),
	}
	obj1.wg.Add(1000)

	for i := 0; i < 500; i++ {
		go func(obj1 *obj, i int) {
			fmt.Println("write", i)
			obj1.set(fmt.Sprintf("%d", i), i)
			obj1.wg.Done()
		}(obj1, i)
	}
	for j := 0; j < 500; j++ {
		go func(obj1 *obj) {
			fmt.Println("read")
			obj1.mu.RLock()
			res := obj1.get()
			json.Marshal(res)
			obj1.mu.RUnlock()
			fmt.Println(len(res))
			obj1.wg.Done()
		}(obj1)
	}

	obj1.wg.Wait()
}

func (o *obj) get() map[string]int {
	return o.resMap
}

func (o *obj) set(key string, val int) {
	o.mu.Lock()
	defer o.mu.Unlock()
	o.resMap[key] = val
}

```
