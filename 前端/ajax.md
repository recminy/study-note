# Ajax

## fetch

现代浏览器原生支持

```javascript
fetch("URL").then(resolve => {}).catch(e => {})
let res = await fetch("url")
let {data/json/...} = res.text()/json()/blob()
## blob二进制图片
let res = await fetch("./images/file.png");
let blobData = await res.blob();
url = URL.createObjectURL(blobData)

imageObject.src = url
```

## jsonp

古老的项目用于解决ajax跨域问题，现在基本不用了

```javascript
function callBack() {
  //TODO cb
  console.log({...data})
  //<script src="http://sp0.baidu.com/api/jsonp?kw=xxx&cb=callBack"
}
```

jquery的jsonp

```javascript
$.ajax({
  url : "url",
  ...
  dataType: "jsonp"
}).then((resolve) => {
  	//TODO
}, rejected => {
  	//TODO
})
```



## ajax2.0

​	增加了表单提交的formData，仅此而已

```javascript
let formData = new FormData()
...
formData.append(field, value)
...

//jquery
let form = $("#form")
$.ajax({
  url : "",
  ..
  data: new FormData(form),
  processData: false,	//以为formData方式提交不转化
  contentType: false	//禁用
}).then(res => {
  //TODO
}, err => {
  //TODO 
})
# PS 注意阻止表单提交
```



## websocket

