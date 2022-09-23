### Vue组件基础

#### 1、组件定义

将 Vue 组件定义在一个单独的 .vue 文件中，这被叫做单文件组件 (简称 SFC)

```html
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>
```

#### 2、组件使用

父组件中导入，components 选项上注册，组件将会以其注册时的名字作为模板中的标签名

```html
<script>
import ButtonCounter from './ButtonCounter.vue'

export default {
  components: {
    ButtonCounter
  }
}
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

#### 3、props传递

相同的视觉布局，但有不同的内容，此时需要向组件传递数据，例如每篇文章标题和内容，这就会使用到 props

Props 是一种特别的 attributes，你可以在组件上声明注册。要传递给博客文章组件一个标题，我们必须在组件的 props 列表上声明它。这里要用到 props 选项

```html
<!-- BlogPost.vue -->
<script>
export default {
  props:['title']
}
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

使用时：

```html
<BlogPost title="My journey with Vue" />
<BlogPost title="Blogging with Vue" />
<BlogPost title="Why Vue is so fun" />
```

数组情况：

```html
<BlogPost
  v-for="post in posts"
  :key="post.id"
  :title="post.title"
 />
```

#### 4、事件监听

利用该组件与父组件交互，父组件@或者v-on监听事件，子组件$emit抛出事件

父组件：

```html
<!--App.vue-->
<script>
import BlogPost from './BlogPost.vue'
  
export default {
  components: {
    BlogPost
  },
  data() {
    return {
      posts: [
        { id: 1, title: 'My journey with Vue' },
        { id: 2, title: 'Blogging with Vue' },
        { id: 3, title: 'Why Vue is so fun' }
      ],
      postFontSize: 1
    }
  }
}
</script>

<template>
  <div :style="{ fontSize: postFontSize + 'em' }">
    <!--父组件可以通过 v-on 或 @ 来选择性地监听子组件上抛的事件-->
    <BlogPost
      v-for="post in posts"
      :key="post.id"
      :title="post.title"
      @enlarge-text="postFontSize += 0.1"
    ></BlogPost>
  </div>
</template>
```

子组件：

```html
<!--BlogPost.vue-->
<script>
export default {
  props: ['title'],
  //通过 emits 选项来声明需要抛出的事件
  emits: ['enlarge-text']
}
</script>

<template>
  <div class="blog-post">
	  <h4>{{ title }}</h4>
      <!--子组件可以通过调用内置的 $emit 方法-->
	  <button @click="$emit('enlarge-text')">Enlarge text</button>
      <!--插槽，可以像HTML元素一样向组件中传递内容-->
      <slot />
  </div>
</template>
```

