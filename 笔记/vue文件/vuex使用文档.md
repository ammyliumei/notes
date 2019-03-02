##基础：核心概念

### State

#####对应使用

	state:{data:""} 	==>  store.state.data
							==>computed 中  setState()返回的是一個對象

#####辅助函数mapState实例

```
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  }),
  
  computed: mapState([
  // 映射相同名字 this.count 为 store.state.count
  'count'
]),
...mapState({
    // 根据以上定义
  })
}



```

###Getter （计算属性）

##### 用法

```
getter:{doneTodos:(state[, getters])=>{ return state.xxx+'xxx'}}
===> store.getters.doneTodos

外部传参(通过内部返回一个带参数函数)
 getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
  ===> store.getters.getTodoById(2)
```

##### 辅助函数mapGetters

​	类同mapState

### Mutation （异步操作，操作state）

##### 用法

```
mutations ：{name（state[,params]）{}} 
			==> 	store.commit('name'[,params])；
            ==>        store.commit({
                          type: 'increment',
                          amount: 10
                        })
```

##### 辅助函数setMutations

​	类同mapState

###### 注意

```
1.最好提前在你的 store 中初始化好所有所需属性。

2.当需要在对象上添加新属性时，你应该

使用 Vue.set(obj, 'newProp', 123), 或 state.obj = { ...state.obj, newProp: 123 }

3.Mutation 必须是同步函数
```

###Action（异步操作，操作mutation）

##### 用法

```
actions: {
    incrementAsync (context[,params]) {
     context 是包含一个与 store 实例具有相同方法和属性的 context 对象
  		//context.commit('xxxx')
  		//context.dispatch('xxxx')
  		 //context.state 和 context.getters
    }
  }
  ===>  store.dispatch('incrementAsync'[,params])
  ===>  store.dispatch({
              type: 'incrementAsync',
              amount: 10
            })
```

##### 辅助方法mapActions

​	类同mapState

## 多个模块module

####基本用法

```
moudules:{a,b,c} ===> 外部通过   store.state.a  的路径访问

每个子module内部中 函数 都比之前多了一个参数 rootState 对象

如：actions ==>参数context中含有 rootState属性 ==> content.rootState
	getter ==> 第三个参数 rootState ==> (state,getter,rootState)
```

####命名空间（根据嵌套路径访问）

```
namespaced: true ;
```

#####访问store中modules中的模块属性与方法

```
 state访问不变 ===> store.state.a.xxx;
 getter ===> getters['account/isAdmin']
 action ===> dispatch(path)
 mutation ===>commit(path)
 注 ===> path = ("有命名属性模块名嵌套路径" ||"没有命名空间的模块继承父模块的路径") + 方法名
```

#####访问root模块属性与方法

```
getter ==>action中==> { dispatch, commit, getters, rootGetters } ;
		 getter中 ==>(state, getters, rootState, rootGetters)
action ==> dispatch('someOtherAction', null, { root: true }) 
mutation ==> commit('someMutation', null, { root: true }) // -> 'someMutation'
```

##### 局部模块想注册全局函数

```
function === >object
{root:true,handler(namespaceContext,payload){}}
```

#####辅助函数

######基本用法 ===> 类同mapState（加正确嵌套路径）;

```
 ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  })
    ...mapActions([
    'some/nested/module/foo',
    'some/nested/module/bar'
  ])
```



######简化版 ===>mapState('嵌套路径'+‘module’)

```
 ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
  ...mapActions('some/nested/module', [
    'foo',
    'bar'
  ])  
```

######createNamespacedHelpers(创建基于某个命名空间辅助函数 )

```
import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
```

# vuex插件（使用store中subscribe函数）

#### 基本用法

```
const myPlugin = store => {
  // 当 store 初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}

==>

const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

###传参用法

```
export default function createWebSocketPlugin (socket) {
  return store => {
    socket.on('data', data => {
      store.commit('receiveData', data)
    })
    store.subscribe(mutation => {
      if (mutation.type === 'UPDATE_DATA') {
        socket.emit('update', mutation.payload)
      }
    })
  }
}

===>

const plugin = createWebSocketPlugin(socket)
const store = new Vuex.Store({
  state,
  mutations,
  plugins: [plugin]
})
```