#### @media媒体查询响应式布局
##### 1. 设置meta标签
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

##### 2. 使用@media查询
###### 1) 在.css文件中使用
```css
@media screen and (max-width: 375px) {
  #box {
    background: #bfa;
    width: 200px;
    height: 200px;
  }
}
```

###### 2) 通过@import引入
```css
@import './responseLayout375.css' screen and (max-width: 375px);
```

###### 3) 通过\<link\>标签引入
```html
<link rel="stylesheet" type="text\css" href="./responseLayout375.css" media="screen and (max-width: 375px)">
```