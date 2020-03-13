---
title: D3 动画与插值
date: 2020-3-013 20:15:24
author:  "Vinecnt Ko"
tags:
    - d3
    - 可视化
---

## D3 动画

D3.js提供了多种工具支持数据可视化的交互，其中`d3.transition`让简单而高效的为图像添加动画成为了可能。

单单从API来讲，`d3.transition`非常简单，用法类似Jquery。 但是想要设计出理想的动画效果，就不得不提到D3绘制图形的一个核心概念`General Update Pattern`. D3的数据驱动特性的核心和实现就是依靠这个Pattern，而动画和交互自然要从它说起了。

> 并不是所有图形都必须遵循Update Pattern，比如一次性绘图，无交互的静态图形等。但如果涉及到了动态数据，这个Update Pattern不仅利于写出易于维护的代码，也能更好的发挥D3强大的功能。

## General Update Pattern

![image-20200308211328582](/../images/update-pattern.png)

D3的数据驱动模式如上图所示，当使用`d3.data()`将数据`Array`与DOM元素绑定的时，数据与元素之间有着三个阶段，即

- Enter 已有数据，但页面还未有与之对应的DOM
- Update 数据元素与DOM元素相绑定
- Exit 数据元素已经被删除，但DOM元素还存在，即失去了绑定元素的DOM

关于这个点，这里不做详细赘述，可参考文档。这里直接对V4和V5版本的`General Update Pattern`进行介绍。举一个简单的例子：

> 假设目前已有数据['a', 'b', 'c'....]等字母序列，现在希望通过D3,使用SVG将其呈现在页面上

### V4

通过`selection.enter()`, `selection.exit()`与 `selection`（update）分别指定相应逻辑，内容如下：

```js
const d3Pattern = (dataSet) => {
  const text = g.selectAll('text').data(dataSet)  // 数据绑定
  
  text.enter()     // enter() 返回绑定数据但是还未生成dom元素的部分
      .append('text')
      .attr('class', 'new')
      .text(d => d)
      .merge(text)  // merge后面的代码，将会分别应用于enter于update两个部分，省略写法
  			.attr('x', (d, i) => 18 * i)
  
  text.attr('class', 'update')  // text 本身是update部分

  text.exit().remove()  // exit() 返回数据已经被删除，但是还存在dom的元素
}
```

### V5

d3 V5.8.0 引入了一个新的API， `selection.join` 

这个API的优势在于，对于一些比较简单、不需要特殊定义enter\exit过程的动作的d3图形，可以简化代码，以上的代码，使用V5的版本写，即

```js
const d3Pattern = (dataSet) => {
  const text = g.selectAll('text').data(dataSet)  // 数据绑定
  
  text.join(
    enter => enter.append('text')
    	.attr('calss', 'new')
    	.text(d => d),
    update => update.attr('class', 'update')
  )
    .attr('x', (d, i) => 18 * i)
  
  // join的exit默认就是 exit().remove()，因此可以免去
}
```

可以发现，使用`selection.join()`, 并不需要再手动写`selection.exit().remove()`，这是因为`selection.join()`这个函数默认的`exit()`函数已经帮你写好了，该API下，d3的Update Pattern可以写为

```js
selection.join(
    enter => // enter.. ,
    update => // update.. ,
    exit => // exit.. 
  )
  // 注意，enter，update等函数一定要return，这样可以对selection继续链式调用
```

当然，这个API的好处在于，一般的使用场景下（不需要在enter、exit等加特殊的动画或操作），完全可以简写，比如：

```js
svg.selectAll("circle")
  .data(data)
  .join("circle")
    .attr("fill", "none")
    .attr("stroke", "black");
```

以上的写法，完全等价于

```js
svg.selectAll("circle")
  .data(data)
  .join(
    enter => enter.append("circle"),
    update => update,
    exit => exit.remove()
  )
    .attr("fill", "none")
    .attr("stroke", "black");
```

等价于V4版本的

```js
circles = svg.selectAll('circle')
	.data(data)

circles.enter()
	.append('circle')
	.merge('circle')
		.attr('fill', 'none')
		.attr('stroke', 'black')

circles.exit().remove()
```

当然，V5完全兼容V4的update Pattern，无论是V4还是V5的新版API，这种Update Pattern的本质没有变，D3仍然是数据绑定，enter/update/exit的工作模式。

### Pattern中的key

当使用`d3.data()`绑定数据和dom时，相对应的关系，可能第一个元素对应第一个dom，第二元素对应第二dom等； 但当`Array`发生变化时，比如重新排序、插入等操作，这时候，数组中元素可能与dom的绑定关系就发生了一些微妙的变化。

最直观的例子就比如动态改变字符的例子

![Kapture 2020-03-08 at 21.56.19](/../images/text-transition.gif)

如图，发现新增的字符总是排在最后，实际上，如果数据一致保持和dom绑定的话，理论随机生成新字符，完全应该有机会出现在中间的。

在数据绑定时，传入一个唯一的key值，即可避免这种情况发生

```js
selection.data(data, d => d.id)
```

完成以上步骤，只要定时的调用函数`d3Pattern`，传入不同的data，即可实现上图的效果

[完整代码](https://codepen.io/vincenttgao/pen/rNVYoYw)

## Transition

好了，前面铺垫了这么多，终于到了主角`d3.transition`了，但实际上，与之相关API屈指可数，想要让d3画出带交互和炫酷过渡效果的，重点还是对Update Pattern理解透彻。

### 基本动画使用

`transition` 的使用，与jquery十分类似，使用时，只需要对选择的元素调用，并指定修改的属性即可，即`selection.transition().attr(...)`

比如现在画布上有一个方块，该元素为`rect`,我想要使其位置从默认的地方，到30位置，并加上动画，代码为

```js
rect.transition()
	.attr('x', 30)  //  设置新位置
```

效果如下

![Kapture 2020-03-08 at 22.17.46](/../images/move-box.gif)

动画的基本使用，就是如此简单，下面简单看下相关的api

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| selection.transition() | this schedules a transition for the selected elements        |
| transition.duration()  | duration specifies the animation duration in milliseconds for each element |
| transition.ease()      | ease specifies the easing function, example: linear, elastic, bounce |
| transition.delay()     | delay specifies the delay in animation in milliseconds for each element |

这里注意d3的api都支持链式调用，因此比如上面的例子，希望将动画时间设置为1s，可以

```js
rect.transition()
	.duration(1000)
	.attr('x', 30)  //  设置新位置
```

同理，ease和delay可以分别设置动画曲线和延迟。

### Update Pattern下的动画

回到最开始的例子，这里用V4版本的Update Pattern举例

因为transition是应用在`selection`上的，所以为了方便使用，我们可以先定义好动画

```js
const t = d3.transtion().duration(750)
```

接下来，我们希望新加入的文字从上面掉下来，且位置更新时，能有一个动画效果，这时候需要设置在`enter()`时，位置有一个从上倒下的过程(transtion)

```js
const d3Pattern = (dataSet) => {
  const t = d3.transtion().duration(750) // 定义动画
  const text = g.selectAll('text').data(dataSet)
  
  text.enter()  
      .append('text')
      .attr('class', 'new')
      .text(d => d)
      .attr('y', -60)
  			.transition(t)
  				.attr('y', 0)
  				.attr('x', (d, i) => 18 * i)
  
  text.attr('class', 'update')
  	.attr('y', 0)
  	.transition(t)
  		.attr('x', (d, i) => 18 * i)

  text.exit()
    .transition(t)
      .attr('y', 60)
      .remove()
}
```

![Kapture 2020-03-08 at 22.40.14](/../images/letter-transition-no-down.gif)

可以看到，原来呆板的样式，已经变的灵动多了。 当然，也可以继续为退出(exit)的文字，加上红色，与掉落的动画，让整体更具有动效，只需要对exit的部分做相应的处理：

```js
text.exit()
	.transition(t)
		.attr('y', 60)
		.remove()
```

![Kapture 2020-03-08 at 22.46.24](/../images/complete-transition.gif)

如图，这是加了向下掉落和透明度变化的动画效果。

[完整代码](https://codepen.io/vincenttgao/pen/XWbzOdY?editors=1111)

### 实战应用

比如现在已经有一个静态的柱状图，希望在鼠标hover的时候，有一些动态效果变化，如下图

![Kapture 2020-03-08 at 23.03.53](/../images/bar-transition.gif)

对于柱状图的实现，这里就不赘述，这里解释下核心代码，思路与上面提到的完全相同：

1. 监听鼠标移入事件

  2. 选择当前的bar，通过transition修改属性
  3. 监听鼠标移出
  4. 选择当前bar，鼠标移出，恢复属性

核心代码如下： 

```js
svgElement
    .on('mouseenter', (actual, i) => {
        d3.select(this)
          .transition()
        	.duration(300)
        	.attr('opacity', 0.6)
        	.attr('x', (a) => xScale(a.language) - 5)
        	.attr('width', xScale.bandwidth() + 10)
    })
    .on('mouseleave’, (actual, i) => {
        d3.select(this)
          .transition()
          .duration(300)
          .attr('opacity', 1)
          .attr('x', (a) => xScale(a.language))
          .attr('width', xScale.bandwidth())
    })
```

这个柱状图的源码与教程出自[D3.js Tutorial: Building Interactive Bar Charts with JavaScript](https://blog.risingstack.com/d3-js-tutorial-bar-charts-with-javascript/)



### 插值动画

对于一些特殊的过渡，比如颜色的变化、数字的跳变等，如果没有插值函数，直接使用`transition().attr()`是无法实现的。

因此，d3提供了插值函数和插值动画的接口用于这类动画实现。当然，对于大多数场景，非差值动画都可满足了。

#### 特殊的插值

对于一些常用的属性插值，d3提供了非常方便入口，分别是`attrTween`(属性插值)/`styleTween`（样式插值）/`textTween` 文字插值

这类插值主要用于比如颜色、线条粗细等“属性”差值，可以使用`attrTween()`和`styleTween`，对于数字变化，连续跳变，可以使用`textTween`他们的用法类似，如下： 

```js
//颜色插值，从红色变为蓝色
transition.attrTween('fill', function() {
  return d3.interpolateRgb("red", "blue");
})

transition.styleTween('color', function() {
  return d3.interpolateRgb("red", "blue");
})
```

插值函数的第一个参数，是要修改的内容或属性，功能类似`transition().attr()`里，attr的内容；第二个参数是返回的插值函数，可以使用d3提供的一些插值函数，当然也可以自定义插值函数。

举个简单的例子，比如想要实现一下效果： 

![Kapture 2020-03-13 at 20.30.23](/../images/tween-letter.gif)

只需要对元素添加鼠标事件，并通过以上的插值函数完成即可

```js
svg.append('text')
	.text('A')
	.on('mouseenter', function() {
  	d3.select(this)
  		.transition()
  		.attrTween('fill', function() {
      	return d3.interpolateRgb("red", "blue");
    	})
	})
  .on('mouseleave', function() {...})
```

接下来说下自定义函数，比如仍然是红色变为蓝色，我们可以在插值函数返回自己定义的函数`func(t)`, 该函数会在动画时间内不断的运行，t为[0, 1]，借助这个思路，以上的效果可以用自定义函数实现如下: 

```js
svg.append('text')
	.text('A')
	.on('mouseenter', function() {
  	d3.select(this)
  		.transition()
  		.attrTween('fill', function() {
      	return function(i) {
          return `rgb(${i * 255}, 0, ${255-(i * 255)})`
        }
    	})
	})
	.on('mouseleave', function() {
  	... // 与上类似
	})
```

以上两种方案，均可以实现动图的效果哦。

可以看到，对于插值动画，核心在于插值内容的产生。d3提供了多款插值，相关的列表如下，比如在使用数字跳变动画时，就可以使用`d3.interpolatorRound(start,end)`来产生整形的数字插值； `d3.interpolateRgb(color, color2)`来产生颜色插值等，具体的插值函数用法可查阅相关API。

- `d3.interpolatNumber`
- `d3.interpolatRound`
- `d3.interpolatString`
- `d3.interpolatRgb`
- `d3.interpolatHsl`
- `d3.interpolatLab`
- `d3.interpolatHcl`
- `d3.interpolatArray`
- `d3.interpolatObject`
- `d3.interpolatTransform`
- `d3.interpolatZoom`

### 通用插值

当然，除了前面提到的API，还有一个更通用的产值函数API，`d3.tween()`

同`attrTween()`等类似，它的第二个参数也是传入插值函数；不同的是，第一个参数，可以传入更通用的想要改变的内容，比如同样是上面的`fill`属性，使用通用插值函数的写法就是:

```js
selection.transition()
	.tween('attr.fill', function() {
    return function(i) {
            return `rgb(${i * 255}, 0, ${255-(i * 255)})`
          }
	})
```

于是我们发现，其实通用API与前面的特殊的三个API用法及其类似，唯一不同的就是通用API的第一个参数可以接受更广泛的变更属性。

这里就不多举例了，关于插值函数的一些参考实例可以在这里[查看](https://observablehq.com/@d3/transition-texttween)



## 参考资料

1. [D3.js Tutorial: Building Interactive Bar Charts with JavaScript](https://blog.risingstack.com/d3-js-tutorial-bar-charts-with-javascript/)
2. [How to work with D3.js’s general update pattern](https://www.freecodecamp.org/news/how-to-work-with-d3-jss-general-update-pattern-8adce8d55418/)
3. [Interaction and Animation: D3 Transitions, Behaviors, and Brushing](http://duspviz.mit.edu/d3-workshop/transitions-animation/)
4. [d3-selection-join](https://observablehq.com/@d3/selection-join)
5. [sortable-bar-chart](https://observablehq.com/@d3/sortable-bar-chart)
6. [Using Basic and Tween Transitions in d3.js](http://4waisenkinder.de/blog/2014/05/11/d3-dot-js-tween-in-detail/)

