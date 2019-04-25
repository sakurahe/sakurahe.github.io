---
title: 图解Redux数据流
date: 2019-04-24 15:16:03
tags:
      - flutter
      - redux
---

## 什么是Redux

[官网](https://cn.redux.js.org/)的解释是：

>Redux 是 JavaScript 状态容器，提供可预测化的状态管理。 (如果你需要一个 WordPress 框架，请查看 Redux Framework。)
>可以让你构建一致化的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。不仅于此，它还提供 超爽的开发体验，比如有一个时间旅行调试器可以编辑后实时预览。
>Redux 除了和 React 一起用外，还支持其它界面库。 它体小精悍（只有 2kB，包括依赖）。

没有接触过的人一看肯定是一脸懵逼，那么什么是Redux呢？

一句话概括，**redux只是一个实现了Flux思想的数据流框架**.

简单介绍一下Flux

一个Flux应用包含四个部分：

- Dispatcher，处理动作分发，维持 Store 之间的依赖关系
- Store，负责存储数据和处理数据相关逻辑
- Action，触发 Dispatcher
- View，视图，负责显示用户界面

![Flux数据流](/images/flux_flow.png)

通过上图可以看出来，Flux 的特点就是**单向数据流**：

- 用户在 View 层发起一个 Action 对象给 Dispatcher
- Dispatcher 接收到 Action 并要求 Store 做相应的更新
- Store 做出相对应更新，然后发出一个 change 事件
- View 接收到 change 事件后，更新页面

所以在 Flux 体系下，如果想要驱动界面，只能派发一个 Store。在这种规矩下，如果想要追溯一个应用的逻辑就变得很轻松了。

## 为什么要使用Redux

在我们一开始构建应用的时候，也许很简单。我们有一些状态，直接把他们映射成视图就可以了。这种简单应用可能并不需要状态管理。

![简单状态流](/images/simpleness.jpg)

但是随着功能的增加，你的应用程序将会有几十个甚至上百个状态。这个时候你的应用应该会是这样。

![复杂状态流](/images/complex.jpg)

我们很难再清楚的测试维护我们的状态，因为它看上去实在是太复杂了！而且还会有多个页面共享同一个状态，例如当你进入一个文章点赞，退出到外部缩略展示的时候，外部也需要显示点赞数，这时候就需要同步这两个状态。

## Redux的基本原则

上文提到过，Redux是Flux的一种实现，但是除了“单一数据源”之外，Redux还强调[三大基本原则](https://cn.redux.js.org/docs/introduction/ThreePrinciples.html):

- 单一数据源(Single source of truth)
- State 是只读的(State is read-only)
- 使用纯函数来执行修改(Changes are made with pure functions)

### 唯一的store

在 Flux 中，应用可以拥有多个 Store，但是分成多个 Store 容易造成数据冗余，数据一致性不太好处理，而且 Store 之间可能还会有依赖，增加了应用的复杂度。所以 Redux 对这个问题的解决方法就是：**整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。**

### 保持状态只读

**唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。**这样确保了视图和网络请求都不能直接修改 state，相反它们只能表达想要修改的意图。因为所有的修改都被集中化处理，且严格按照一个接一个的顺序执行。

### 数据改变只能通过纯函数来完成

这里说的纯函数就是[reducers](https://www.redux.org.cn/docs/Glossary.html#reducer)，它接收先前的 state 和 action，并返回新的 state。

## 在Flutter中应用Redux

下面咱们根据例子来了解一下Redux在Flutter中的应用

### 添加依赖

在pubspec.yaml中添加以下依赖

```
  redux: ^3.0.0
  flutter_redux: ^0.5.3
```

### 创建State

状态是由**reducer生成并储存在Store**里面的。Store更新状态的时候，并**不是更改原来的状态对象**，而是直接将reducer生成的新的状态对象**替换**掉老的状态对象。所以，我们的状态应该是immutable的。

```
@immutable
class AppState {
  final int count;

  AppState({
    this.count = 0,
  });
}

```

### 创建Action

action到底是什么？View如何发出action。其实，action只是我们对状态进行操作方法的一个代号而已。在我们这个应用中，唯一的一个功能就是让count的值+1，所以我们这里只有一个action。

```
/*
 * 定义操作该State的全部Action
 * 这里只有增加count一个动作
 */
class IncrementAction {}
```

### 创建Reducer

reducer是我们的状态生成器，它接收一个我们原来的状态，然后接收一个action，再匹配这个action生成一个新的状态。

```
final countReducer = combineReducers<int>({
  TypedReducer<int, IncrementAction>(_increment),
});

int _increment(int state, action) {
  return state + 1;
}
```

### 创建store

Store接收一个reducer，以及初始化State，我们想用Redux管理全局的状态的话，需要将store储存在应用的入口才行。

```
final Store<AppState> store = Store<AppState>(
  appReducer,
  initialState: AppState(),
);
```

### 将Store放入顶层

futter_redux包提供了一个StoreProvider用来接收store。

```
class MyApp extends StatelessWidget {
  final Store<AppState> store = Store<AppState>(
    appReducer,
    initialState: AppState(),
  );

  @override
  Widget build(BuildContext context) {
    return StoreProvider(
      store: store,
      child: MaterialApp(
        title: 'Example',
        debugShowCheckedModeBanner: false,
        home: IncrementContainer(),
      ),
    );
  }
}
```

### 在子页面中获取Store中的state

这里建议大家把实际代码对照下面的解释一起看。

```
static _ViewModel fromStore(Store<AppState> store) {
    return _ViewModel(
      count: store.state.count,
    );
  }
```

```
StoreConnector<AppState, _ViewModel>(
      distinct: true,
      converter: _ViewModel.fromStore,
      builder: (context, vm) => IncrementComponent(
            count: vm.count,
          ),
    );
```

要想获取store我们需要使用StoreConnector<S,ViewModel>。StoreConnector能够通过StoreProvider找到顶层的store。而且能够在state发生变化时rebuilt Widget。

![StoreConnector解释](/images/explain.jpg)

- 首先这里需要强制声明类型，S代表我们需要从store中获取什么类型的state，ViewModel指的是我们使用这个State时的实际类型。
- 然后我们需要声明一个converter<S,ViewModel>，它的作用是将Store转化成实际ViewModel将要使用的信息，比如我们这里实际上要使用的是count，所以这里将count提取出来。
- builder是我们实际根据state创建Widget的地方，它接收一个上下文context，以及刚才我们转化出来的ViewModel，所以我们就只需要把拿到的count放进Text Widget中进行渲染就好了。

### 发出Action

store.dispatch发起一个action，任何中间件都会拦截该操作,在运行中间件后，操作将被发送到给定的reducer生成新的状态，并更新状态树。

```
static _ViewModel fromStore(Store<AppState> store) {
    return _ViewModel(
      onTap: () => store.dispatch(IncrementAction()),
    );
  }
```

```
StoreConnector<AppState, _ViewModel>(
      distinct: true,
      converter: _ViewModel.fromStore,
      builder: (context, vm) => IncrementComponent(
            onTap: vm.onTap,
          ),
    );
```

以上便是在flutter中使用redux共享状态信息的全部内容。

## ViewModel性能优化

我们的StoreConnector能够将store提取出信息并转化成ViewModel，这里其实是有一个性能优化的点的。我们这里的例子非常简单，它的ViewModel就只是一个int的值，当我们ViewModel很复杂的时候，我们可以使用StoreConnector的distinct属性进行性能优化。使用方法很简单：需要我们在ViewModel中重写[==] and [hashCode] 方法，然后把distinct属性设为true。

## 总结

我们已经完成了整个应用，现在大家了解Redux的运行原理了吗？

具体代码可以到 [GitHub](https://github.com/sakurahe/flutter-_redux_count) 查看。
