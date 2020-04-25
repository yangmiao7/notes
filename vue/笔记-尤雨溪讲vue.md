## 响应式原理

> Reactivity：响应式编程，在vue中意味着当改变状态时，状态将在从整个系统的更新中反应出来（如反应到DMO中的变化）；

 react 中手动调用setState强制触发状态改变；

angular中直接操作状态，angular中使用脏检查实现；

vue中使用ES5的```Object.defineProperty```API 将状态对象转换为响应式的（MobX中也是这样进行的依赖追踪）;



### 数据监听（Data Observer）

通过使用```Object.defineProperty```

#### ```Object.defineProperty(obj, prop, descriptor)```

参数：

```obj```要进行定义的对象；

```prop```要定义或修改的属性名称；

```descriptor```要定义或修改的**属性描述**；

**数据属性：**

1. ```[[Configurable]]```:标识属性能否被删除，能否修改属性描述；默认为```false```；
2. ```[[Enumerable]]```:标识是否为可枚举属性（能否通过for-in循环）；默认为```false```；
3. ```[[Value]]```:属性的数据值；默认为```undefined```;
4. ```[[Writable]]```:能否修改属性的值（上面的```[[value]]```），默认为```false```；在严格模式下为```false```时，赋值操作会抛出异常；非严格模式下会忽略赋值操作，静默错误；

*直接在对象上定义的属性，属性的```[[Configurable]]```、```[[Enumerable]]```、```[[Writable]]```特性都被设置为```true```；*

**访问器**

1. ```[[Get]]```：在读取属性时调用的函数。只指定```getter```意味着属性不能写入；
2. ```[[Set]]```：在楔入属性时调用的函数。只指定```setter```意味着属性不能读取，在非严格模式下为```undefined```；

*```getter``` ``` setter```不能与```value``` ```writable```同时设置*



## 双向数据绑定

> 数据传递的两个方向

1. 从数据（model层）到视图（view层），数据发生变化时，视图也随之变化；
2. 从视图（view层）到数据；用户在界面进行操作时（输入、选择等），触发数据的变化，视图也会发生改变；