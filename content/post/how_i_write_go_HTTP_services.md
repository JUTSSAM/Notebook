---
title: "[译]我是如何用go写HTTP服务的"
date: 2018-08-03T15:46:03+08:00
draft: false
---

自从go pre1.0版本，我就在使用go来构建HTTP API/Service。在我的Machin
eBox项目中，需要构建大量的API来使之易用。

这些年来，我构建服务的方式一直在变化，我想和大家分享一下现在所用的模式，希望能够帮到大家。

## 服务类型结构体

所有的组件都有一个单独的`server`结构如下：

```
type server struct{
	db      *someDatabase
	router  *someRouter
	email 	 EmailSender
}
```

结构体中部分字段是他们共享的依赖(或者说是配置)。

## routes.go

在每个组件中都有一个文件routes.go包含所有的路由：

```
package app

func (s *server) routes(){
	s.router.HandleFunc("/api/",s.handleAPI())
	s.router.HandleFunc("/about",s.handleAbout())
	s.router.HandleFunc("/",s.handleIndex())
}
```
方便代码维护，在此文件中可以很快找到代码所在位置。

## Handlers接受服务

```
func (s *server) handleSomething() http.HandleFunc {...}
```

Handler可以通过s变量获取相关依赖(配置)。

## 返回Handler

handler不处理请求，而是返回一个处理请求的函数。

函数在闭包中运行。

```
func (s *server) handleSomething() http.HandlerFunc {
	thing := prepareThing()
	return func(w http.ResponseWrier,r *http.Request){
		// use thing
	}
}
```

prepareThing只被调用一次，所以可用用来做初始化，然后将thing用于handler。

如果要修改共用文件，记得需要互斥量或者其他机制来保护。

## 获取handler特定的配置

handler需要特殊的配置时，可以将其作为一个参数。


```
func (s *server) handleGreeting(format string) http.HandleFunc(){
	return func(w http.ResponseWriter, r *http.Request){
		fmt.Fprintf(w,format,"world")
	}
}
```

## HandlerFunc 而不是 Handler

我更喜欢使用http.HandlerFunc 而不是 http.Handler

```
func (s *server) handleSomething() http.HandlerFunc {
	return func(w http.ResponseWrier,r *http.Request){

	}
}
```

他们或多或少是可以互换的，所以选择一个更适合阅读的，我选择前者。

## 中间件只是go函数

中间件函数接收`http.HandlerFunc`并返回一个新的该类型函数--它可以在前者运行之前或之后插入一些代码，或者可以让原来的函数完全不运行。


```
func (s *server) adminOnly(h http.HanlerFunc) http.HandlerFunc{
	return func(w http.ResponseWriter, r *http.Request){
		if !currentUser(r).IsAdmin {
			http.NotFound(w,r)
			return
		}
		h(w,r)
	}
}
```

handler 内部的逻辑可以决定是否运行原来的handler，在上例中，如果`IsAdmin`是`false`，handler会返回一个404状态，并且结束运行，作为参数传入的`h`handler并没有被调用。

相反，则`h`会被调用。

通常中间件会被列在routes.go文件中：

```
...

func (s *server) routes(){
	...
	s.router.HandleFunc("/admin",s.adminOnly(s.handleAdminIndex))
}

```

## 请求、响应自定义格式

如果API端点有自己的请求和响应格式，一般只适用于特定的handler。

需要在函数中对格式定义。


```
func (s *server) handleSometing() http.HandleFunc{
	type request struct{
		Name string
	}

	type response struct{
		Greeting String `json:"greeting"`
	}
	return func(w http.ResponseWriter, r *http.Request){
		...
	}

}
```

这样的写法会让包空间更加干净，并且允许使用相同的类型名称，不需要考虑特定处理函数的版本。

在测试代码中，只需把这些类型复制到测试函数中即可。

## 测试类型可以帮助构建

如果请求/响应的格式类型隐藏在handle中，只需要在测试代码中定义新的类型。

在这里可以为将来的人理解代码做一些帮助。

例如如果有一个Person类型，可以在多个端点使用，greet端点只关心其中的name，就可以写如下测试代码：

```
func TestGreet(t *testing.T){
	is := is.New(t)
	p := struct{
		Name string `json:"name"`
	}{
		Name : "Mat",
	}

	var buf bytes.Buffer
	err := json.NewEncoder(&buf).Encode(p)
	is.NoErr(err)
	req,err := http.NewRequest(http.MethodPost,"/greet",&buf)
	is.NoErr(err)
}

```

可以清楚知道，我们只关心person的name。

## sync.Once

在准备handler时，如果必须要做一些代价昂贵的事情，则在第一次被调用的时候defer。（放在栈底）

这会提升启动时间。


```
func (s *server) handleTemplate(files string...) http.HandleFunc{
	var (
		init sync.Once
		tpl  *template.Template
		err  error
	)
	return func(w http.ResponseWriter, r *http.Request){
		init.Do(func(){
			tpl,err = template.ParseFiles(files...)
		})
		if err != nil{
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}
		//use tpl
	}
}
```

`sync.Once`保证代码只执行一次，处理其他请求时则会保持资源直到所有请求都完成。

- error检查在init函数外边，所以有错误时不会丢失掉
- 如果handler没有被调用，则代价昂贵的事务永远不会执行

## server是可测试的

server类型是可测试的

``` 
func TestHandleAbout(t *testing/.T){

	is := is.New(t)
	
	srv := server{
		db:    mockDatabase,
		email: mockEmailSender,
	}

	srv.routes()
	req ,err := http.NewRequest("GET","/about",nil)
	is.NoErr(err)
	w := httptest.NewRecorder()
	srv.ServeHTTP(w, r)
	is.Equal(w.StatusCode, http.StatusOK)

}

```

- 在每个测试中创建一个服务器实例，就算是很大组件也不会耗时太长
- 通过测试ServeHTTP，可以测试整套系统
- 使用httptest.NewRecorder记录handler的行为

## 原文链接

[How I write Go HTTP services after seven years](https://medium.com/statuscode/how-i-write-go-http-services-after-seven-years-37c208122831)