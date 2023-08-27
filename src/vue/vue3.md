# Vue3.0

* 更好的性能：
* Tree-shaking支持；
* 组合API
* Fragment/Teleport/Susponse
* 更好的ts支持
* 暴露了自定义渲染API

## 1. 特性
### 更好的性能
compiler优化细节：
#### 1. 靶向更新

$$
靶向更新
\begin{cases}
BlockTree \\
PatchFlags
\end{cases}
$$

思路：跳过静态内容，只对比动态内容。

```vue
<div>
    <span />
    <span>{{msg}}</span>
</div>
```
编译成：
```js
import {createVNode as_createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock} from "vue";

export function render(_ctx, _code) {
    return (_openBlock(), _createBlock("div", null, [
        _createVNode("span", null, "static"),
        _createVNode("span", null, _toDisplayString(_ctx.msg), 1 /* Text */)
    ]));

}
```
在运行时会生成number（>0）的PatchFlag, 用作标记。
仅带PatchFlag标记节点被追踪，动态节点直接与Block根节点绑定。无需再遍历静态节点。

```js
9 /* Text, Props */ 既有Text变化，又有props变化
```

##### Block 
VNode, 带有dynamicChildren，避免传统diff(一层一层遍历)，更新准确知道该节点应用哪些更新动作（靶向更新）。

1. v-if的元素是作为Block
Block（DIV）
    --Block(Section v-if)
    --Block(Section v-else)


```js
const block = {
    tag: 'div',
    dynamicChildren: [
        {   tag: 'section', {key: 0},
            dynamicChildren: [...]
        },
        {
            tag: 'section', {key: 1},
            dynamicChildren: [...]
        }   
    ]
}
```
BlockTree => Dom结构不稳定；

2. v-for的元素作为Block
使用一个Fragment充当Block角色。但是仍面临不稳定，回退到传统diff。当v-for的表达式是常量，Fragment是稳定的。

#### 2. 提升静态节点
举个栗子

```vue
<div>
    <p>text</p>
</div>
```
```js
const hoist1 = createVNode('p', null, 'text');
function render() {
    return (openBlock(), createBlock('div', null, [hoist1]));
}
```


#### 3. 预字符串化
静态节点序列化为字符串并生成一个static类型的VNode。

```js
const hoistStatic = createStaticVNode('<p></p><p>...</p>');

render() {
    return(openBlock(), createBlock('div', null, [
        hoistStatic
    ]));
}
```
优点：
* 生成代码体积减少；
* 减少创建VNode开销；
* 减少内存占用；





#### 4. 事件监听缓存：cacheHandlers

```vue
<div>
    <span @click="onClick">
        {{msg}}
    </span>
</div>
```
开启cacheHandlers后:

```js
export function render(_ctx, _cache) {
    return (_openBlock(), _creaateBlock("div", null, [
        _createVNode("span", {
            onClick: _cache[1] || (_cache[1] = $event => (_ctx.onClick($event)))
        }, _toDisplayString(_ctx.msg), 1 /* Text */);
    ]));
}
```
自动生成并缓存一个内联函数，变为静态属性。


### PatchFlag
定义：

```js
export const enum PatchFlags {
    TEXT = 1, // 动态textContent
    CLASS = 1 << 1, // 动态class元素
    STYLE = 1<<2, // 动态样式
    PROPS = 1 << 3, // 动态非类
    FULL_PROS = 1<< 4, 
    HYDRATE_EVENTS = 1 << 5, // 带事件监听
    ...
}
```

### Fragment
render可返回数组了
 

### Teleport
类似之前的`<portal>`


### Composition API

$$
API
\begin{cases}
reactive \\
watchEffect \\
computed \\
ref\\
toRefs\\
hooks
\end{cases}
$$

举个栗子：
```js
import {reactive, computed} from 'vue';
export default {
    setup() {
        const state = reactive({
            a: 0
        });
        function increate() {
            state: a++
        }

        return {
            state,
            increate
        }
    }
}

//...
const double = computed(() => state.a + 3);
```

### 生命周期钩子对比


| vue2.x | vue3.x |
| -------|--------|
|beforeCreate|setup|
|created|setup|
|beforeMount|onBeforeMount|
|mounted|onMounted|
|beforeUpdate|onBeforeUpdate|
|updated|onUpdated|
|beforeDestory|onBeforeUnmount|
|destroyed|onUnmounted|
|errCaptured|onErrorCaptured|


## 2. 响应式原理

### reactive
Proxy：
1. 对属性的添加、删除动作的监听；
2. 对数组基于下标的修改，length修改的检测；
3. 对Map、Set、WeakMap、WeakSet支持；

建立响应式reactive:

$$
reactive 
\begin{cases}
reactive, 返回proxy对象，有递归 \\
shalllowReactive, 只一层响应 \\
readonly: 不可修改 \\
shallowReadonly
\end{cases}
$$

部分源码：
```js
export function reactive(target: object) {
    if (readonlyToRaw.has(target)) {
        return target;
    }
    return createReactiveObject({
        target,
        rawToReactive,
        reactiveToRaw,
        mutableHandlers, // 处理基于类型和引用类型，
        mutableCollectionHandlers // 处理set、map等类型
    });
}

```

```js
const collectionTypes = new Set<Function>([Set, Map, WeakMap, WeakSet]);

function createReactiveObject() {
    target: unknow,
    toProxy: WeakMap<any, any>,
    toRaw: WeakMap<any, any>,
    baseHandlers: ProxyHandler<any>,
    collectionLHandlers: ProxyHander<any>
} {
    // 目标对象是否被effect
    let observed = toProxy.get(target);
    if (observed !== void 0) {
        return observed;
    }
    if (toRaw.has(target)) {
        return target;
    }
    const handlers = collectionTypes.has(target.constructor) ? collectionHanders: baseHanders;
    // 创建响应式对象
    observed = new Proxy(target, handlers);
    toProxy.set(target, observed);
    toRaw.set(observed, target);
    return observed;
}
```

```mermaid
graph TB

    shallowReactive---CreateReactiveObject
    reactive---CreateReactiveObject
    readonly---CreateReactiveObject
    shallowReadonly---CreateReactiveObject

    CreateReactiveObject---s{是否已proxy}
    s--是-->返回observed
    s--否-->isSet{"是否是Set、Map、WeakSet、WeakMap类型"}

    isSet--"是(collection)"-->Proxy[New Proxy]
    isSet--"否(baseHandlers)"--->Proxy

    Proxy-->isRead{是否只读}
    isRead--是-->A[rawToReadonly\n readonlyToRaw\n 收集]
    isRead--否-->B["rawReactive \n reactiveToRaw \n 收集"]

```

### effect
watcher(2.0) => effect(3.0)

```js
const mountComponent: MountComponentFn = (
    initialVNode,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    optimized
) => {
    const instance: ComponentIntervalInstance = (initialVNode.component = createComponentInstance(
        initialVNode,
        parentComponent,
        parentSuspense
    ));
    // 建立proxy
    setupComponent(instance);
    // 建立渲染effect，执行effect
    setupRenderEffect(
        instance, // 组件实例
        initialVNode,
        container,
        achor,
        parentSuspense,
        isSVG,
        optimized
    );
}
```

1. 创建component实例；
2. 初始化组件，建立proxy， 得到render函数；
3. 建立一个effect，执行effect；

```js
// 构建渲染effect
const setupRenderEffect: setupRenderEffectFn = (
    instance,
    initialVNode,
    container,
    achor,
    parentSuspense,
    isSVG,
    optimized
) => {
    instance.update = effect(function componentEffect() {
        // ...
    }, {scheduler, queueJob});
}
```
创建一个effect，赋给组件实例的update，作为渲染更新视图用。

```js
export function effect<T = ant>(
    fn: v => T,
    options: ReactiveEffectOptions = EMPTY_OBJ
): ReactiveEffect<T> {
    const effect = createReactiveEffect(fn, options);
    if (!options.lazy) {
        effect();
    }
    return effect;
}

function createReativeEffect<T = any>(
    fn: (...args: any[]) => T,
    options: ReativeEffectOptions
): ReactiveEffect<T> {
    const effect = function reativeEffect(...args: unknow[]): unkonw {
        try {
            enableTracking();
            effectStack.push(effect);
            activeEffect = effect;
            return fn(...args);
        } finally {
            effectStack.pop();
            resetTracking();
            // 将activeEffect还原
            activeEffect = effectStack[effectStack.length - 1];
        } as ReactiveEffect
        // 初始化参数
        effect.id = uid++;
        effect._isEffect = true;
        effect.active = true;
        effect.raw = fn;
        effect.deps = []; // 收集相关依赖
        effect.options = options;
        return effect;
    }
}
```
组件初始化阶段
```mermaid
graph TB;
初始化mountComponent-->createComponentInstance创建组件实例-->setupC["setupComponent创建组件，调用composition-api\n 处理options（构建响应式），编译template"]-->setupR[setupRenderEffect创建一个渲染effect]---r23["reactiveEffect: effect初始化创建一个reactiveEffect"]-->A["不是lazy情况,立即执行effect函数 \n 将当前effect赋给activeEffect"];

setupC--reactive产生-->proxy响应式;
setupR--effect创建-->renderEffect-->activeEffect;

A--立即执行-->renderEffect;

```


vue2.0, 响应式在初始化就深层递归处理；  
vue3.0, 获取上一级get之后才触发下一级的深度响应；

### track
track, 找到与当前proxy和key对应的dep，dep与当前activeEffect建立里联系，收集依赖。

依赖收集器：
* targetMap：proxy：depsMapproxy，存放依赖dep的map映射；
* depsMap: deps存放effect的set数据类型

<font color="red">依赖收集，get做了什么？</font>

```mermaid
flowchart TB;
start["Reflect.get获取属性"]-->isShallow{是否浅响应};

isShallow--是-->isShallowRead{是否只读};
isShallow--否-->isRead{是否只读};

isShallowRead--是-->返回res;
isShallowRead--否-->track响应式-->返回res了;

isRead--是-->isObject{是否引用类型};
isRead--否---t[tranck响应式]-->isObject;

isObject--是-->readonly{是否只读属性};
isObject--否-->C[返回res];

readonly--是-->A["深度响应式readonly(res)"];
readonly--否-->B["深度响应式reactive(res)"];


```



**依赖收集阶段**

```mermaid
graph TD;

    reactive["reactive()"]-->proxy;
    proxy-->set;
    proxy-->get-->track--通过target和key2层映射建立dep-->dep[找到对应dep];

    effect["effect()"]-->renderEffect-->rt[renderComponentRoot \n 触发render方法]-->D[解析表达式，替换真实data \n 下的属性，触发get];

    D--触发get-->get;
    renderEffect<--"建立双向关联\n 将当前renderEffect存入dep中\n 将dep存入当前effect的deps中"-->dep;


```





1. 执行renderEffect，赋值给activeEffect，调用renderComponentRoot，触发render函数；
2. 根据render函数，解析经过compile，语法树处理后模板表达式，访问真实data属性，触发get；
3. get方法首先经过之前不同reactive，通过track方法进行依赖收集；
4. track方法通过当前proxy对象target、key找对应的dep；
5. 将dep与当前reactiveEffect建立起联系，将activeEffect压入dep数组中，dep中已含有当前组件的渲染effect，触发set，能在数组中找到对应effect，依次执行。


#### set触发更新流程

```mermaid
flowchart TB
    start["this[key]=value \n 改变属性"]--触发-->set

    Proxy-->get
    Proxy-->set-->trigger-->fen[分离effect和\n computed]--依次-->isScheduler{需要scheduler}

    isScheduler--是-->放入scheduler调度
    isScheduler--否-->执行fn

    subgraph targetMaps
        subgraph depMaps
            deps
        end
    end
    

    deps-->fen
    trigger--"通过proxy和key \n 找到对应deps"-->deps


```

```js
export function trigger(
    target: object,
    type: TrrigerOpTypes,
    key?:unknow,
    newValue?:unknow,
    oldValue?: unknow,
    oldTarget ?: Map<unknow, unknow> | Set<unkoow>
) {
    const depsMap = targetMap.get(target);
    // 没有经过依赖收集的，直接返回
    if (!depsMap) {
        return;
    }
    // 钩子队列
    const effects = new Set<ReactiveEffect>(); 
    // 计算属性队列
    const computedRunners = new Set<ReactiveEffect>();
    const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
        if (effectsToAdd) {
            effectsToAdd.forEach(effect=> {
                if (effect !== activeEffect || !shouldTrack) {
                    if (effect.options.computed) {
                        computedRunners.add(effect);
                    } else {
                        effects.add(effect);
                    }
                }
            });
        }
    }
    add(depsMap.get(key));
    const run = (effect: ReactiveEffect) => {
        if (effect.options.scheduler) {
            effect.options.scheduler(effect);
        } else {
            effect();
        }
    }
    // 必须首先运行计算属性更新
    computedRunners.forEach(run); // 依次执行回调
    effects.forEach(run);

}
```



