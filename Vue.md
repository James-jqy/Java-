# el属性

**1.Vue实例的作用范围？**

在el命中的元素内部是可以继续使用{{message}}来渲染数据的。但是在命中的元素外部就不行了。

![image-20201206204541988](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201206204541988.png)

**2.是否可以使用其它选择器？**

可以使用各种选择器。

**3.是否可以使用其它元素**

可以使用任何标签。（所有双标签都支持，除了body）



![image-20201206204834649](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201206204834649.png)



# data 数据对象

![image-20201206205243578](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201206205243578.png)

我们在{{}}中写的属性名字就是Vue实例里面data属性里面的属性名字。不一定要message。就比如说，Vue的data里面有一个数组叫campus。那么在页面上可以直接写{{campus[0]}}。

在差值表达式中可以写一个表达式，但是只能写一个。也可以写一个函数。

# v-text

```html
<div class="app">
    <h2 v-text="message"></h2>
    <h2>上海 {{message}}</h2>
</div>

<script>
	var app = new Vue({
        el:"#app",
        data:{
            message:"黑马程序员"
        }
    })
</script>
```

就像上面那段文本，v-text和直接写差值表达式的区别就是，v-text指令会将标签里面的内容全部替换。而插值表达式只会替换它所存在的地方。

# v-html

和上面的v-text都差不多。只不过当值是html结构的时候，会被解析成html的页面。普通文本没有显示的差异。

# v-on

需要注意的一个地方。就是v-on:可以替换为@。

![image-20201207110447870](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201207110447870.png)

Vue的一大特点就是双向数据绑定。我们不再需要去修改dom节点，查找dom节点。而是专注于数据层面的修改就行了。因为数据修改的时候，dom节点也会跟着修改。

在对象的某个方法上，修改同一个对象中的属性的时候可以使用this关键字。

![image-20201207111211569](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201207111211569.png)

# v-on（补充）

![image-20201207182215622](D:\Desktop\笔记\images\image-20201207182215622.png)

![image-20201207182227643](D:\Desktop\笔记\images\image-20201207182227643.png)

# v-show

![image-20201207111347879](C:\Users\Jieqiyue\AppData\Roaming\Typora\typora-user-images\image-20201207111347879.png)

```html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        window.onload = function(){
            var app = new Vue({
                el:"#app",
                isShow:false,
                data:{
                    isShow:true
                }
            })    
        }
    </script>   
</head>
<body>
    <div id="app">
        <img v-show="isShow" src="./1.jpg">
    </div>
</body>
</html>
```

v-show会把后面那个属性看做bool值来进行解析。可以在Vue实例中定义一个属性来作为v-show指令的判断依据。

# v-if

和v-show差不多。不过这个v-if操作的是dom树。当需要频繁添加或者删除的话，建议用v-show。

# v-bind

![image-20201207120715474](D:\Desktop\note\images\image-20201207120715474.png)

![image-20201207120732426](D:\Desktop\note\images\image-20201207120732426.png)

# v-for

![image-20201207164037718](D:\Desktop\笔记\images\image-20201207164037718.png)

当直接用item作为对象的输出时，会把对象所有的都输出。如果需要将对象中某一个属性输出。需要用点语法。

![image-20201207164507867](D:\Desktop\笔记\images\image-20201207164507867.png)

# v-model

双向数据绑定。可以十分便捷的通过这个指令，将表单中的值和Vue实例中的某个值进行绑定。两个当中的任意一个改变后，两个都会改变。

![image-20201207183927436](D:\Desktop\笔记\images\image-20201207183927436.png)

要注意某一个变量定义的作用域范围。在某一个函数中定义的变量，在函数外面不可以访问。函数中可以访问到全局变量。当一个函数被定义在全局作用域的时候，它可以访问所有在全局作用域中定义的变量。

# Vue网络应用

# axios





# 组件

注册一个组件可以用下面的代码

```html
   // 注册一个组件
    Vue.component('blog-post', {
        props: ['title', 'keys'],
        template: `
          <div>
              <h2>{{ keys }}  : {{ title }}</h2>
              <h3>{{ title }}</h3>
          </div>
		`
    })


// 改进后的写法
    Vue.component('blog-post', {
        props: ['post'],
        template: `
          <div class="blog-post">
              <h3>{{ post.title }}</h3>
              <div v-html="post.content"></div>
          </div>
        `
    })

<div id="blog-post-demo">
<!--    重构后的,重构后  -->
    <blog-post
            v-for="post in posts"
            v-bind:key="post.id"
            v-bind:post="post"
    ></blog-post>
</div>

new Vue({
        el: '#blog-post-demo',
        data: {
            posts: [
                {id: 1, title: 'My journey with Vue'},
                {id: 2, title: 'Blogging with Vue'},
                {id: 3, title: 'Why Vue is so fun'}
            ]
        }
    })
```

注意到，props可以接收多个参数。并且参数的值可以是任意的。可以是一个对象。当template中需要使用很多个参数的时候，最好改进为下面那种写法（传入对象的方式）。因为，在template中也可以使用对象加点的方式得到对象中的属性。

# 组件事件

子组件可以抛出事件。如下：

```html
Vue.component('blog-post', {
        props: ['post'],
        template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button v-on:click="$emit('enlarge-text',1)">
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
    })
```

<button v-on:click="$emit('enlarge-text',1)"> 就抛出了一个事件。事件名为enlarge-text。那么，在父级组件中可以监听到这个事件的抛出。使用v-on即可监听到子组件抛出的事件。需要指定事件名。还能带有参数。在上面那个事件中，参数值是1。下面接收的是一个叫做onEnlargeText的Vue中的一个函数。

```html
 methods: {
            onEnlargeText: function (enlargeAmount) {
                this.postFontSize += enlargeAmount
            }
        }
```



```html
<div id="blog-posts-events-demo">
        <div :style="{ fontSize: postFontSize + 'em' }">
            <blog-post
                    v-for="post in posts"
                    v-bind:key="post.id"
                    v-bind:post="post"
                    v-on:enlarge-text="onEnlargeText"
            ></blog-post>
        </div>
    </div>


new Vue({
        el: '#blog-posts-events-demo',
        data: {
            posts: [
                {id: 1, title: 'My journey with Vue'},
                {id: 2, title: 'Blogging with Vue'},
                {id: 3, title: 'Why Vue is so fun'}
            ],
            postFontSize: 1
        },
        methods: {
            onEnlargeText: function (enlargeAmount) {
                this.postFontSize += enlargeAmount
            }
        }
    })
```

## [Prop 的大小写 (camelCase vs kebab-case)](https://cn.vuejs.org/v2/guide/components-props.html#Prop-的大小写-camelCase-vs-kebab-case)



# 组件的prop

你可以以对象形式列出 prop，这些 property 的名称和值分别是 prop 各自的名称和类型：

```html
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

## [传递静态或动态 Prop](https://cn.vuejs.org/v2/guide/components-props.html#传递静态或动态-Prop)

像这样，你已经知道了可以像这样给 prop 传入一个静态的值：

```html
<blog-post title="My journey with Vue"></blog-post>
```

你也知道 prop 可以通过 `v-bind` 动态赋值，例如：

```html
<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

在上述两个示例中，我们传入的值都是字符串类型的，但实际上*任何*类型的值都可以传给一个 prop。

**这两种方式有所不同。静态的只能是传入字符串，如果要传入数字，Boolean等就需要动态的传入。**

# computed

计算属性，可以定义在vue实例中。与methods属性不同的是，computed可以缓存结果的值。如果值不变的话，最好使用computed。使用computed时候，不用写括号。不像是调用方法。



# v-bind：key

在循环指令v-for的时候，最好要加入key，key就是一个唯一标识。标识每一个生成的节点都不一样。

















