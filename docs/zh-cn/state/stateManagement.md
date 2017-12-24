> 以react为例子 最开始学习 React 的时候，并不需要额外的工具来做状态管理
使用 React 的单项数据流 props + 组件内部的 state 更新来做 UI 以及组件状态的变化管理
只要 props 或是 state 变化，React 就会重新 render 相应的组件，这种方式对于处理一些极其简单的项目是完全够用的.
但是对于复杂的项目

1: 组件之间通信的成本增高
2: 数据流变得模糊
3: 组件变得臃肿

>基于以上需要引入状态管理
通过把所有 state 提升到 global store，然后通过 react-redux 提供的 Provider，
让 components 能够获得、修改、更新 state，整个层级体系一下子清晰了。
所以说，状态管理，说白了还是降低组件间的耦合度。
而耦合度，恰恰是制约大型应用可维护性的至关重要的因素。
所以说一旦代码规模上去了，自然就会需要一个提供状态管理的机制。


redux VS mobx
1.Redux 和 MobX 都是为了解决”状态管理“这个问题而生的

Redux 的核心概念有三个
* global state store
* dispatch actions
* pure reducers

 Redux 只有一个 store，并且解耦了 reducer 和 state。与此同时它也采用了很多函数式编程的范式，
 比如说 redux 强调的是输入 (action) 和输出 (new state) ；比如说 immutable，
 这可能是 Redux 自始至终最为强调的一点，reducer 全部是纯函数，没有 side effect。
 这意味着每个 action 都会产生一个全新的 state，而不是修改之前的 state，这也带来了调试方面它独有的优势

>而 react-redux 则是通过 Provider 提供给每个组件都能接触 global store 的机会（不需要一直 pass as props），
此外通过 mapStateToProps，使得 state 和 props 同步更新，触发 render，
通过 mapDispatchToProps，使得组件能够 dispatch action，更新 state。

mobx

>在命令式编程中，a := b + c 意味着将 b + c 的值赋给 a，随后 b 和 c 的值的变化都不会再影响 a。
但是在响应式编程中，b 和 c 的变化都会导致 a 的自动更新（并不需要重新执行 a := b + c）。

 MobX 的几个关键点：
 
* @observable: make the state trackable
* @computed: derive everything from the state, automatically
* @action: change state
* reaction: auto-run, side effect


>另外值得一提的是，无论是 Redux 还是 MobX，在主要提供数据和视图解耦的解决方案的同时，都体现了自己的分治思想（Divide and Conquer）。
在 Redux 中，这种思想体现在 reducer 的分解；而在 MobX 中，主要体现在 Multiple Store。