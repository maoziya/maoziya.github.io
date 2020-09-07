---
title: jquery on，绑定动态元素的事件，与data属性传值的格式
urlname: kixgq7
date: 2020-07-02 10:01:57 +0800
tags: []
categories: []
---

**官方语法： **
\$(selector).on(event,childSelector,data,function)
\*\*
**属性介绍：**
event              必需 规定要从被选元素添加的一个或多个事件或命名空间。由空格分隔多个事件值，也可以是数                          组，必须是有效的事件。
childSelector 可选 规定只能添加到指定的子元素上的事件处理程序（且不是选择器本身，比如已废弃的                                  delegate() 方法）。
data                 可选 规定传递到函数的额外数据。
function         可选 规定当事件发生时运行的函数。

官方上百度有很多例子了，说下这个 date 属性的传值吧：

**示例：**
对.divmjl 元素下的.mjlbut 集合绑定点击事件，并传值
\$('.divmjl').on('click', '.mjlbut', {msg: mjlid},this.remove);

对返回的 [object Object] 推断出传入的需要是一个{xx: xx}格式的参数。
remove(event){
           alert(event.data.msg); //取值需要这样取
 }

**结语：**
然后最后有意思的事，这个方法，可以用在 vue 里面，主要本人还是太菜了，哈哈，才刚刚接触 vue，不是很会用，最后没办法，为了快点完成，只能先怎么写了，简单粗暴的直接给动态生成的 div 绑定一个事件。
