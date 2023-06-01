# axios

基于 `Promise` 的异步请求框架

## 基本用法

axios(config)

别名

- axios.get()
- axios.post()
- axios.put()
- axios.delete()





# 特殊操作

预览pdf文件

```js
this.$axios
    .post('http://localhost:8088/provision/pdf', 
          // 请求数据
          pdfModel, 
          // 需要指定 responseType 为 blob, 否则转换失败, 预览为空白
          {responseType:"blob"})
    .then(res => {
	// 构造 blob 对象, 并在新窗口打开
    let blob = new Blob([res.data], {
        type: `application/pdf`
    });
    let fileURL = URL.createObjectURL(blob);
    window.open(fileURL)
})
```

