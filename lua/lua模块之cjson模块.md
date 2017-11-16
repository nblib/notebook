lua json的解析和编译
===
## 摘要(Synopsis)
```
local cjson = require "cjson"
-- Module instantiation
-- 实例化模块
local cjson2 = cjson.new()
local cjson_safe = require "cjson.safe"

-- Translate Lua value to/from JSON
text = cjson.encode(value)
value = cjson.decode(text)

-- Get and/or set Lua CJSON configuration
setting = cjson.decode_invalid_numbers([setting])
setting = cjson.encode_invalid_numbers([setting])
keep = cjson.encode_keep_buffer([keep])
depth = cjson.encode_max_depth([depth])
depth = cjson.decode_max_depth([depth])
convert, ratio, safe = cjson.encode_sparse_array([convert[, ratio[, safe]]])
```
## 模块实例化(Module Instantiation)
```
local cjson = require "cjson"
local cjson2 = cjson.new()
local cjson_safe = require "cjson.safe"
```
* `cjson`解码/编码的时候遇到非法数据会抛出错误
* `cjson_safe`和`cjson`一样,但是遇到非法数据时,不会抛错误,而是返回`nil`和`error`.
* `cjson.new()`会创建一个实例,拥有自己独立的编码缓冲区和默认设置.
## 解析json字符串(decode)
```
value = cjson.decode(json_text)
```
* UTF-16和UTF-32不支持
```
json_text = '[ true, { "foo": "bar" } ]'
value = cjson.decode(json_text)
-- Returns: { true, { foo = "bar" } }
```
## 解码不可用的数字(decode_invalid_numbers)
```
setting = cjson.decode_invalid_numbers([setting])
-- "setting" must be a boolean. Default: true.
```
* 尝试解析不被json规范支持的数字时,会报错,比如:
    * infinity
    * not-a-number(NaN)
    * hexadecimal
* 可用的设置:
    * `true`: 会解析不可用的数字,默认设置
    * `false`: 当遇到不可用的数字会抛错误

这个方法调用后会返回当前的设置值.
```
local in = cjson.decode_invalid_numbers(false)
--in is false
```
## json字符串最大解析深度(decode_max_depth)
```
depth = cjson.decode_max_depth([depth])
-- "depth" must be a positive integer. Default: 1000.
```
* 设置json字符串最大内嵌深度,当超过这个深度,会抛出错误,防止栈溢出
* 默认深度为1000.
## 编码(encode)
```
json_text = cjson.encode(value)
```
* 支持如下类型转化为json串:
    * boolean
    * nil
    * number
    * string
    * table
    * lightuserdata(null value only)
* 剩下的会报错:
    * function
    * lightuserdata(not null value)
    * thread
    * userdata
* 默认数字保留十四位精度
* 遇到下面的字符串会被转义:
    * 控制字符
    * 双引号
    * 斜杠和反斜杠
    * 删除符号
* 遇到下面的情况会报错:
    * 不符合json规范的数字(infinity,NaN)
    * 表嵌套超过1000层
    * 过度稀疏lua数组
## 编码设置
1. encode_invalid_numbers
    * true : 允许编码不可用的值.
    * null : 生成为空值
    * false : 报错
2. encode_keep_buffer
    * true : 缓存编码内容
    * false : 不缓存
3. encode_max_depth
    * depth: 数字,编码最大嵌套深度,默认1000
4. encode_number_precision
    * 数字编码精度: 1-14,默认14
5. encode_sparse_array
    ```
    cjson.encode({ [3] = "data" })
    -- Returns: '[null,null,"data"]'
    ```
    ```
    cjson.encode_sparse_array(true)
    cjson.encode({ [1000] = "excessively sparse" })
    -- Returns: '{"1000":"excessively sparse"}'
    ```