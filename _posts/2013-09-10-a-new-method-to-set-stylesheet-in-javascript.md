---
layout: post
title: javascript设置CSS样式的新方法
date: 2013-09-10 16:50:04.000000000 +08:00
categories:
- 前端技术
tags: [javascript, 前端]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '2'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
  first_name: ''
  last_name: ''
---

在用JS写各种效果的时候，用的最多的应该就是改变对象的CSS样式来达到自己想要的效果。 
最常见的比如更换背景，更换字体颜色，图片切换等等。这些无非就是改变元素的背景，字体颜色， 
left,top 等属性。通过在JS执行过程中动态设置元素的各种属性，就能做出很漂亮的效果， 
最常见莫过于图片切换，下拉菜单以及各种友好的提示框等等。 


我们通常设置CSS的方法就是获取对象，然后通过改变JS与CSS对应的属性达到更改目的。 
比如JS中的backgroundColor对应CSS中的background-color，采用驼峰规则改写， 
每一个横杠后面的第一个字母都要大写，不过有几个特殊的，大伙可以自己查查。 

假如网页中有这样一个元素： 

	<div  id='wrapper'> Hello JS.</div> 

通过  

	var _wrp = document.getElementById( "wrapper"  );


获取这个这个元素，如果要把_wrp的背景颜色，宽度分别改成黑色，和300像素，我们可以一般 

会直接通过下面两句代码设置： 

*   第一种方法

    _wrp.style.backgroundColor = "#000"; 
    _wrp.style.width = "300px"; 

这是我们最常用的方法，大家再熟悉不过了。 

*   第二种方法 

这几天没事敲了一个播放连续图片以达到看电影效果的一个图片切换，做到预加载进度条的时候， 
在设置进度条样式的时候遇到了一个麻烦，因为我是让用户自己传入一个CSS字符串，格式就是正常的 
CSS，传进来之后，再对它进行分割到一个数组里面，达到 '属性名'=>'属性值'的程度：

比如从"width:300px;background-color:#000"  

分割成{ "width" => "300px", "background-color" => "#000"  }

然后通过一个循环设置那个元素的样式，这样就遇到一个问题，CSS里面的属性名需要改成JS中的属 
性名，这就是一个麻烦事了，如果只是一个横杠比如background-color还好说，关键是还有几个的 
以及一些特殊的，这就更麻烦了，不可能在设置之前弄一个很大的转换过程，效率太低了。 
然后我突然想到，_wrp.style如果是一个数组，如果是数组那直接通过_wrp.style['属性名']赋值即可， 
我alert了一下_wrp.style['width']，它很惊喜的弹出了前面设置的300px！！！ 

这样我设置CSS就简单的多了， 

直接通过 

{% highlight javascript %}

_wrp.style['width'] = '300px';
_wrp.style['background-color'] = '#000'; 

{% endhighlight %}

设置即可，在谷歌和火狐测试下都通过了。 

这种方法至少有一下两个优点： 

*   不需要记着JS里面那些对应的属性名，更不用说特殊的了。 
*   如果你有一个数组，里面都是'css属性名：属性值', 这时，我们只需要通过一个循环就能达到目的，高效方便。 

使用如下：
	{% highlight javascript %}

var _style_str = "width:300px;background-color:#000"; //CSS样式字符串,注意空格问题，如果有空格后面要把空格去掉 
var style_arr =  _style_str.split( ";"  ); //通过 ; 分割成 {"width:300px", "background-color:#000"} 
var _wrp = document.getElementById( "wrapper"  ); //要被改变样式的元素 
 
for( var i = 0; i < style_arr.length; i++  ){ 
    arr = style_arr[i].split(":"); //再次分割 
    _wrp.style[arr[0].trim()] = arr[1]; //比如 _wrp.style['width'] = "300px" 
}
	{% endhighlight %}
这样一个元素的样式就很好设置了！
