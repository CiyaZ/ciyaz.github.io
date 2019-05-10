# Less语法

这里介绍一些Less相对于CSS的扩展语法。

## 变量

Less中能够定义和引用变量。

```less
@width:10rem;
@height:10rem;
.app {
  width: @width;
  height: @height;
}
```

## 属性引入

Less中可以在一组属性定义中引入另一组，便于复用。

```less
.btn {
  font-family: "Microsoft YaHei UI", sans-serif;
  font-size: 1.2rem;
  padding: 0.62rem;
  border: none;
  border-radius: 5px;
  outline: none;
}
.btn-danger {
  // 引入通用属性
  .btn();
  // 定制的颜色属性等
  color: #ffffff;
  background-color: #ff6e66;
  transition: background-color 0.5s;
  box-shadow: 0 0 5px 1px #ff6e66;
  &:hover {
    background-color: #ff9185;
  }
}
```

假如我们还有一些`btn-success`，`btn-warn`等定义，使用Less就可以省去重复写很多相同配置的做法了。当然，使用纯CSS，只要要求用户引入两个`class`也能实现相同的效果，但是对于用户来说就比较麻烦了。

## 嵌套

CSS中，可以使用类似`.list li`这种形式，选择`.list`的所有子`<li>`元素，或者使用`.list > li`只选出一代子元素。但如果我们要设置多种子元素，CSS中就要平行的写出很多种类似配置，可读性不好。Less中能够以嵌套的形式编写父子元素配置。

```less
.list {
  .item {
    background-color: aqua;
  }
}
```

如果我们打算只选一代子元素，可以用`>`。

```less
.list {
  > .item {
    background-color: aqua;
  }
}
```

`&`表示父元素，下面写法能嵌套编写`.item`的`hover`效果。

```less
.list {
  .item {
    background-color: aqua;
    &:hover {
      background-color: red;
      transition: 0.5s background-color;
    }
  }
}
```

## 运算符

Less中，数值属性可以支持运算符。

```less
@width: 10px + 10px;
.app {
  width: @width;
}
```
