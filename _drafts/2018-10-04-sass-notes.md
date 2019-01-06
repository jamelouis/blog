---
layout: post
title: Sass Notes
categories: web
---

## Preprocessing

Sass添加了一些如*Variable*,*nesting*,*mixins*,*inheritance*特性，便于管理和配置CSS。

## Variables

```
$variable-name
```

## Nesting

嵌套特性，便于为某个特定标签下的其他标签设置属性。
```
tag one {
    tag two {

    }
}
```

## Partials

_filename.scss表示该文件不是一个完整的预处理的sass文件。
因此不会生成css，但可以导入到其他的scss文件中。

## import

```
@import "filename.scss"
```

## Mixins

因为CSS3的一些特性，在不同的浏览器做了不同的支持，导致了一些特性
有特定的前缀，为了避免反复的设置这些前缀的问题，引入了Mixins的语法，
来批量处理。


## Extend/Inheritance

扩展便于为不同的标签设置相同的特性。

## Operators

提供了数学运算符，便于标签盒型空间计算。

## References

* [Sass](http://sass-lang.com/)
* [Sass Basic](http://sass-lang.com/guide)
