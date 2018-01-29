# 概述
golang 在导入一个包后,没有使用,会报错,如果希望执行一个包中的`init`函数,但用不到别的东西,这时可以通过`_`解决
# demo
```go
package string
import _ "os"
```
这样,会执行`os`中的`init`函数,但不会报`unused import`的错误