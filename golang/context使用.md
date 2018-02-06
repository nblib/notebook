# context
golang 中的创建一个新的 goroutine , 并不会返回像c语言类似的pid，所有我们不能从外部杀死某个goroutine，所有我就得让它自己结束，之前我们用 channel ＋ select 的方式，来解决这个问题，但是有些场景实现起来比较麻烦，例如由一个请求衍生出的各个 goroutine 之间需要满足一定的约束关系，以实现一些诸如有效期，中止routine树，传递请求全局变量之类的功能。于是google 就为我们提供一个解决方案，开源了 context 包。使用 context 实现上下文功能约定需要在你的方法的传入参数的第一个传入一个 context.Context 类型的变量。
## 用法
### 设置结束时间,时间到后,结束parent和child
```go
//如果child的超时时间比parent晚,那么以父类的为准,父类结束,都结束
func TestDeadline(t *testing.T) {
	ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(5*time.Second))

	child, _ := context.WithDeadline(ctx, time.Now().Add(10*time.Second))
	go func(ctx context.Context) {
	PROCESS:
		for {
			time.Sleep(2 * time.Second)
			select {
			case <-ctx.Done():
				fmt.Println(ctx.Err())
				break PROCESS
			default:
				fmt.Println("this is from child")
			}
		}
	}(child)

	time.Sleep(20 * time.Second)
	//如果已经超时,这个不起作用
	cancel()
	<-ctx.Done()
	<-child.Done()
	fmt.Println(ctx.Err())
	fmt.Println(child.Err())
}
```
### 设置超时时间
```go
//内部调用了DeadLine,和Deadline一样
```
### 取消context,父类调用cancle后,子类也会结束
```go
func TestCancelChild(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())

	child, _ := context.WithCancel(ctx)
	go func(ctx context.Context) {
		for {
			time.Sleep(2 * time.Second)
			select {
			case <-ctx.Done():
				fmt.Println(ctx.Err())
			default:
				fmt.Println("this is from child")
			}
		}
	}(child)

	time.Sleep(5 * time.Second)
	cancel()
	time.Sleep(5 * time.Second)
}
```