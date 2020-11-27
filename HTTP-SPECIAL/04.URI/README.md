
# URI 的格式
统一资源标识符（**U**niform **R**esource **I**dentifier），包含 URL 和 URN 连个部分。在 HTTP 里的网址实际上是 URL —— 统一资源定位符（**U**niform **R**esource **L**ocator）。

URI 本质上是一个字符串，这个字符串的作用是**唯一地标记资源的位置或者名字。**
![URI格式](https://static001.geekbang.org/resource/image/46/2a/46581d7e1058558d8e12c1bf37d30d2a.png)

**scheme** 表示协议名，表示资源应该使用哪种协议来访问。如：http、https、ftp、ldap、file等。
**host:port**表示资源所在的主机名
**path** 标记资源所在位置的 path
**query** 表示查询参数

# URI的编码
对于 URI 中 ASCII 码以外的字符集和特殊字符进行转义，RFC 规范里称为 “escape” 和 “unescape”

## escape、encodeURI 和 encodeURIComponent
首先 escape 是对字符串进行编码，而encodeURI 和 encodeURIComponent 是针对URI。因此，当需要对 URI 进行编码时，不用这个方法。

其次，encodeURI 和 encodeURIComponent，相同点是都是编码URI，区别在于编码的字符范围，其中，encodeURI 不会对 ASCII 字母、数字、 `~!@#$&*()=:/,;?+'` 进行编码，encodeURIComponent 不会对下列字符编码 ASCII字母、数字、`~!*()'`进行编码。

**encodeURI**
  ```javascript

  encodeURI("http://www.cnblogs.com/season-huang/some other7@");
  ```
编码后结果为：
```javascript
"http://www.cnblogs.com/season-huang/some%20other7@"
```
**encodeURIComponent**
```javascript

encodeURIComponent("http://www.cnblogs.com/season-huang/some other7@")
```
编码后结果为：
```javascript
"http%3A%2F%2Fwww.cnblogs.com%2Fseason-huang%2Fsome%20other7%40"
```

## 总结
1. 如果只是编码字符串，用escape
2. 如果需要编码整个 URI，然后使用这个 URI，那么用 encodeURI。
3. 如果需要编码 URI 中的参数，使用 encodeURIComponent