### vue深层组件嵌套传值



使用场景

在父组件A 里 引入了子组件B , 但同时B组件又是子组件C的父组件,C又是子组件D的父组件

那么, A组件分别向B ,C ,D传值是如何实现的?

D组件又是如何分别向C B A 传值的

这里就提到 **$attrs** 和 **$listeners**

#### [$attrs官方文档](https://cn.vuejs.org/v2/api/#vm-attrs)

> $attrs概念： 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。
>
> 当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，
>
> 并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。



[**$listeners官方文档**](https://cn.vuejs.org/v2/api/#vm-listeners)

>$listeners概念：包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。
>
>它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。



[**inheritAttrs**](https://cn.vuejs.org/v2/api/#inheritAttrs)

>inheritAttrs概念：默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。
>
>当撰写包裹一个目标元素或另一个组件的组件时，这可能不会总是符合预期行为。
>
>通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。
>
>而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上。



简单来说, **C传值给D 通过props** ; **B传值给C 通过props** ;**A 传值给 D 也是通过props** 

但是 我们不想在B C 组件里写props 时,而直接在D里面写props 在中间组件B和C 用 $attrs 进行传递比较好

#### 主要解决的问题

>vue多级组件的传参：A-B-C-D 四个组件嵌套，
>
>参数需要从A->D 或者A-C，D组件想要改变A组件的内容等。

**A组件**

```vue
<template>
  <div>
      <BComponent
      :B='B'
      :C='C'
      :D='D'
      @DClick='DClick'
      @CClick='CClick'
      @BClick='BClick'
      >
      </BComponent>
  </div>
</template>

<script>
import BComponent from './BComponent';

export default {
  components: {
    BComponent,
  },
  data() {
    return {
      B: 'hello B',
      C: 'hello C',
      D: 'hello D',
    };
  },
  methods: {
    DClick() {
      console.log('子组件D操作A组件成功');
    },
    CClick() {
      console.log('子组件C操作A组件成功');
    },
    BClick() {
      console.log('子组件B操作A组件成功');
    },

  },
};
</script>

<style>

</style>

```

**B组件**

```vue
<template>
  <div>
      B组件
      <p>B:{{B}}</p>
      <p>attrs:{{$attrs}}</p>
      <CComponent
      v-bind="$attrs"
      v-on="$listeners"
      @DClick='DClick'
      @CClick='CClick'
      >
      </CComponent>
  </div>
</template>

<script>
import CComponent from './CComponent';

export default {
  components: {
    CComponent,
  },
  props: {
    B: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
    };
  },
  methods: {
    DClick() {
      console.log('子组件D操作B组件成功');
    },
    CClick() {
      console.log('子组件C操作B组件成功');
    },
    mounted() {
      console.log(this.$attrs, 'this.$attrs');
      console.log(this.$listeners, 'this.$listeners');
    },

  },
};
</script>

<style>

</style>

```

**C组件**

```vue

<template>
  <div>
      C组件
      <p>C:{{C}}</p>
      <p>attrs:{{$attrs}}</p>
      <p @click='isClick'>C组件独有的事件</p>
      <DComponent
      v-bind="$attrs"
      v-on="$listeners"
      @DClick='DClick'
      >
      </DComponent>
  </div>
</template>

<script>
import DComponent from './DComponent';

export default {
  components: {
    DComponent,
  },
  props: {
    C: {
      type: String,
      default: '',
    },
  },
  inheritAttrs: false,
  data() {
    return {
    };
  },
  methods: {
    DClick() {
      console.log('子组件D操作C组件成功');
    },
    isClick() {
      this.$emit('CClick');
    },
    mounted() {
      console.log(this.$attrs, 'this.$attrs');
      console.log(this.$listeners, 'this.$listeners');
    },

  },
};
</script>

<style>

</style>

```

**D组件**

```vue
<template>
  <div>
      D组件
      <p>D:{{D}}</p>
      <p @click='DClick'>D组件独有的事件</p>
  </div>
</template>

<script>

export default {
  components: {
  },
  props: {
    D: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
    };
  },
  methods: {
    DClick() {
      this.$emit('DClick');
    },
    mounted() {
    },

  },
};
</script>

<style>

</style>


```

这里详细说一下**inheritAttrs属性**

inheritAttrs到底有啥用？到底用在哪里？看下边代码

```vue
<template>
    <childcom :name="name" :age="age" type="text"></childcom>
</template>
<script>
export default {
    'name':'test',
    props:[],
    data(){
        return {
            'name':'张三',
            'age':'30',
            'sex':'男'
        }
    },
    components:{
        'childcom':{
            props:['name','age'],
            template:`<input type="number" style="border:1px solid blue">`,
        }
    }
}
</script>
```

 父组件传递了type="text"，子组件里input 上type="number"，那渲染到页面会是什么样?

 父组件传递的type="text"覆盖了input 上type="number"，把input数据类型都给改变了，

 需要input 上type="number"类型不变，但是我还是要取到父组件的type="text"的值，那么代码如下：

```vue
<template>
    <childcom :name="name" :age="age" type="text"></childcom>
</template>
<script>
export default {
    'name':'test',
    props:[],
    data(){
        return {
            'name':'张三',
            'age':'30',
            'sex':'男'
        }
    },
    components:{
        'childcom':{
            inheritAttrs:false,
            props:['name','age'],
            template:`<input type="number" style="border:1px solid blue">`,
            created () {
                console.log(this.$attrs.type)
            }
        }
    }
}
</script>
```

默认情况下vue会把父作用域的不被认作 props 的特性绑定 且作为普通的 HTML 特性应用在子组件的根元素上。

怕遇到一些特殊的，就比如上面的input的情况，这个时候inheritAttrs:false的作用就出来啦。



**$listeners属性**详细说明

父组件-子组件-孙子组件....现在我要你在孙子组件里改变父组件的值，怎么改？有很多方法，但是$listeners给我们提供了一个新的思路  



```vue
<template>
    <div>
        <childcom :name="name" :age="age" :sex="sex" @testChangeName="changeName"></childcom>
    </div>
</template>
<script>
export default {
    'name':'test',
    props:[],
    data(){
        return {
            'name':'张三',
            'age':'30',
            'sex':'男'
        }
    },
    components:{
        'childcom':{
            props:['name'],
            template:`<div>
                <div>我是子组件   {{name}}</div>
                <grandcom v-bind="$attrs" v-on="$listeners"></grandcom>
            </div>`,
           
            components: {
                'grandcom':{
                    template:`<div>我是孙子组件-------<button @click="grandChangeName">改变名字</button></div>`,
                    methods:{
                        grandChangeName(){
                           this.$emit('testChangeName','kkkkkk')
                        }
                    }
                }
            }
        }
    },
    methods:{
        changeName(val){
            this.name = val
        }
    }
}
</script>
```

