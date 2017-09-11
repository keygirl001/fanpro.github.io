---
layout:    post
title:     "React源码学习"
subtitle:   " \"重点react diff和生命周期\""
date:       2017/08/21
author:     "CaoFan"
header-img: "img/react-article-bg.png"
catalog: true
tags:
  - react

---
>"react"


## 前言
对于react来说，虽然接触的时间比较长，但是没有系统的总结过，今天顺带着复习和重新能发现新知识的心态来总结一下react的知识。

## 正文

### diff算法     
React中最重要的是虚拟Dom机制，它使得我们可以不用大量操作dom，因为对于Web UI界面来说就是对于dom的操作，所以它会提高性能，优化代码。虚拟Dom就是利用了与diff算法的完美结合，以前我也只是了解到这里，今天我想更加深入的了解一下。       

传统的diff算法：计算一棵树形结构转换成另一棵树形结构的最少操作，通过循环递归对节点进行一次对比，效率比较低，复杂度为O(n^3),其中n是树中节点的总数。       
**React将复杂度变为O(n)**，react是如何让复杂度变为O(n)的呢？      
> 做了两种假设：   
  两个相同组件产生类似的DOM结构，不同的组件产生不同的DOM结构。        
  对于同一层级的子节点，它们可以通过唯一的id进行区分。      

1 **对于两棵树的比较只会对同一层的节点进行比较，彻底简化了复杂度**。如下图，只会循环一次对每一层进行比较，如果发现节点不存在，则会把该节点和其子节点完全删除。 
![img](/img/in-post/react-base-article/diff-first.png)    

    updateChildren: function(nextNestedChildrenElements, transaction, context) {
      //通过updateDepth来表示树的层级
      updateDepth++;
      var errorThrown = true;
      try {
        this._updateChildren(nextNestedChildrenElements, transaction, context);
        errorThrown = false;
      } finally {
        updateDepth--;
        if (!updateDepth) {
          if (errorThrown) {
            clearQueue();
          } else {
            processQueue();
          }
        }
      }
    }

 
2 **不同类型的节点：当树中的同一位置前后输出了不同类型的节点，React直接完全删除前面的节点以及它的子节点，然后创建并插入新的节点。**     
对于component来说也是同样的道理：
![img](/img/in-post/react-base-article/diff-second.png)   
通过逐层比较发现，Dcomponent变为了Gcomponent，即使这两个component结构相似但是如果判断为不同类型的componet，React就会直接删除Dcomponent，重新创建Gcomponent以及它的子节点。   

3 **相同类型，不同属性的节点：这个就比较简单了即重新设置属性即可。对于虚拟DOM来说，style的属性值不是简单的字符串而是一个对象。**      

4 **列节点的比较：允许开发者对同一层级的子节点，通过唯一的key值进行区分，大大提高了性能。**        
diff算法对于列节点提供了三种操作：插入、移动、删除。    


    //插入
    function enqueueInsertMarkup(parentInst, markup, toIndex) {
      updateQueue.push({
        parentInst: parentInst,
        parentNode: null,
        type: ReactMultiChildUpdateTypes.INSERT_MARKUP,
        markupIndex: markupQueue.push(markup) - 1,
        content: null,
        fromIndex: null,
        toIndex: toIndex,
      });
    }        
    //移动  
    function enqueueMove(parentInst, fromIndex, toIndex) {
      updateQueue.push({
        parentInst: parentInst,
        parentNode: null,
        type: ReactMultiChildUpdateTypes.MOVE_EXISTING,
        markupIndex: null,
        content: null,
        fromIndex: fromIndex,
        toIndex: toIndex,
      });
    }       
    //删除  
    function enqueueRemove(parentInst, fromIndex) {
      updateQueue.push({
        parentInst: parentInst,
        parentNode: null,
        type: ReactMultiChildUpdateTypes.REMOVE_NODE,
        markupIndex: null,
        content: null,
        fromIndex: fromIndex,
        toIndex: null,
      });
    }         


    分析一下源码：


    _updateChildren: function(nextNestedChildrenElements, transaction, context) {
      var prevChildren = this._renderedChildren;  //老集合
      var nextChildren = this._reconcilerUpdateChildren(
        prevChildren, nextNestedChildrenElements, transaction, context
      );     //新集合
      if (!nextChildren && !prevChildren) {
        return;
      }
      var name;
      var lastIndex = 0;
      var nextIndex = 0;
      for (name in nextChildren) {              //遍历新集合中的节点
        if (!nextChildren.hasOwnProperty(name)) {
          continue;
        }     //过滤掉原型上的属性
        var prevChild = prevChildren && prevChildren[name];
        var nextChild = nextChildren[name];
        if (prevChild === nextChild) {            //如果存在相同的结点
          // 移动节点
          this.moveChild(prevChild, nextIndex, lastIndex);
          lastIndex = Math.max(prevChild._mountIndex, lastIndex);    //更新lastIndex
          prevChild._mountIndex = nextIndex;                        //更新节点在新集合中的位置
        } else {                            //如果节点不同的话
          if (prevChild) {                  
            lastIndex = Math.max(prevChild._mountIndex, lastIndex);
            // 删除节点
            this._unmountChild(prevChild);
          }
          // 初始化并创建节点
          this._mountChildAtIndex(
            nextChild, nextIndex, transaction, context
          );
        }
        nextIndex++;                       //进入下一个节点的判断
      }
      for (name in prevChildren) {        //遍历老集合中的节点
        if (prevChildren.hasOwnProperty(name) &&
            !(nextChildren && nextChildren.hasOwnProperty(name))) {      //判断一下如果老集合中有，新集合中没有，则删除节点
          this._unmountChild(prevChildren[name]);
        }
      }
      this._renderedChildren = nextChildren;
    },
    // 移动节点
    moveChild: function(child, toIndex, lastIndex) {
      if (child._mountIndex < lastIndex) {
        this.prepareToManageChildren();
        enqueueMove(this, child._mountIndex, toIndex);
      }
    },         
    // 创建节点 
    createChild: function(child, mountImage) {
      this.prepareToManageChildren();
      enqueueInsertMarkup(this, mountImage, child._mountIndex);
    },          
    // 删除节点
    removeChild: function(child) {
      this.prepareToManageChildren();
      enqueueRemove(this, child._mountIndex);
    },          
    _unmountChild: function(child) {
      this.removeChild(child);
      child._mountIndex = null;
    },
    _mountChildAtIndex: function(
      child,
      index,
      transaction,
      context) {
      var mountImage = ReactReconciler.mountComponent(
        child,
        transaction,
        this,
        this._nativeContainerInfo,
        context
      );
      child._mountIndex = index;
      this.createChild(child, mountImage);
    },         

实例：       
![img](/img/in-post/react-base-article/diff-third.png)   
根据源码我们要先遍历新的集合       
+ 遍历到B，判断一下老集合中也有相同的B节点，有相同的B节点，我们要执行moveChild，在老集合中B.\_mounthIndex = 1，此时的lastIndex = 0，不满足条件，更新lastIndex = 1，更新B在新集合中的位置B._mountIndex = 0，nextIndex = 1判断下一个节点;     


+ 遍历到E，判断老集合中没有E节点，执行this.\_mountChildAtIndex，创建新节点E，更新E在新集合中的位置E._mountIndex = 1，nextIndex = 2判断下一个节点;       


+ 遍历到C，判断一下老集合中也存在C节点，执行moveChild，在老集合中C.\_mounthIndex = 2，lastInde = 1，不满足，更新lastIndex = 2，更新C在新集合中的位置C.\_mounthIndex = 2，nextIndex = 3判断下一个节点；     


+ 遍历到A，判断一下老集合中也存在A节点，执行moveChild，在老集合中A.\_mounthIndex = 0，lastIndex = 2，满足A.\_mounthIndex < lastIndex，所以要对A进行移动操作，更新lastIndex = 2，更新A在新集合中的位置A.\_mounthIndex = 3，nextIndex = 4进入下一个节点；        


+ 最后要遍历老集合，判断是否存在在新集合中没有但在老集合中存在的节点，发现D节点，所以接下来删除D节点，diff算法完成。

### 组件的生命周期（Component Lifecycle）      
#### **初始化阶段**（顺序执行）        
_getDeafultProps()_:获取实例的默认属性，`react.createClass()`初始化组件，自定义组件的入口方法，负责管理`getDeafultProps()`,并且只执行一次。                 

    createClass: function (spec) {
      var Constructor = function (props, context, updater) {
        // 触发自动绑定
        if (this.__reactAutoBindPairs.length) {
          bindAutoBindMethods(this);
        }

        // 初始化参数
        this.props = props;
        this.context = context;
        this.refs = emptyObject;  // 本组件对象的引用，可以利用它来调用组件的方法
        this.updater = updater || ReactNoopUpdateQueue;

        // 调用getInitialState()来初始化state变量
        this.state = null;
        var initialState = this.getInitialState ? this.getInitialState() : null;
        this.state = initialState;
      };

      // 继承父类
      Constructor.prototype = new ReactClassComponent();
      Constructor.prototype.constructor = Constructor;
      Constructor.prototype.__reactAutoBindPairs = [];

      injectedMixins.forEach(mixSpecIntoComponent.bind(null, Constructor));

      mixSpecIntoComponent(Constructor, spec);

      // 调用getDefaultProps，并挂载到组件类上。defaultProps是类变量，使用ES6写法时更清晰
      if (Constructor.getDefaultProps) {
        Constructor.defaultProps = Constructor.getDefaultProps();
      }

      // React中暴露给应用调用的方法，如render componentWillMount。
      // 如果应用未设置，则将他们设为null
      for (var methodName in ReactClassInterface) {
        if (!Constructor.prototype[methodName]) {
          Constructor.prototype[methodName] = null;
        }
      }

      return Constructor;
    },
    
其中`getDefaultProps()`是通过`Constructor`管理的，所以它在react整个生命周期中只会执行一次，并且初始化的实例都会共享`defaultProps`。 


_getInitialState()_：创建组件实例对象的时候调用它来获取初始化的state。 
_componentWillMount()_：执行它时不会重新触发render函数，但是可以调用setState来修改state的值，并进行合并。        
_render()_：生成虚拟DOM节点，只能访问，不能修改状态和DOM。        
_componentDidMount()_：真实Dom被渲染之后调用，如发送ajax，定时器设置。      

    var ReactCompositeComponent = {
      /**
      * 组件初始化,渲染、注册事件
      * @param {ReactReconcileTransaction|ReactServerRenderingTransaction} transaction
      * @param {?object} hostParent
      * @param {?object} hostContainerInfo
      * @param {?object} context
      * @return {?string} 返回的markup会被插入DOM中.
      * @final
      * @internal
      */
      mountComponent: function (transaction, nativeParent, nativeContainerInfo, context) {
          // 当前元素对应的上下文
          this._context = context;
          this._mountOrder = nextMountID++;
          this._nativeParent = nativeParent;
          this._nativeContainerInfo = nativeContainerInfo;

          var publicProps = this._processProps(this._currentElement.props);
          var publicContext = this._processContext(context);

          var Component = this._currentElement.type;

          // 初始化公共类
          var inst = this._constructComponent(publicProps, publicContext);
          var renderedElement;

          // 用于判断组件是否为 stateless(无状态组件)，无状态组件没有状态更新队列，它只专注于渲染，它本质上只是返回JSX函数，轻量级的React组件。
          if (!shouldConstruct(Component) && (inst == null || inst.render == null)) {
              renderedElement = inst;
              warnIfInvalidElement(Component, renderedElement);
              inst = new StatelessComponent(Component);
          }

          // 这些初始化参数本应该在构造函数中设置，在此设置是为了便于进行简单的类抽象
          inst.props = publicProps;
          inst.context = publicContext;
          inst.refs = emptyObject;
          inst.updater = ReactUpdateQueue;

          this._instance = inst;

          // 将实例存储为一个引用，方便查找
          ReactInstanceMap.set(inst, this);

          // 初始化 state
          var initialState = inst.state;
          if (initialState === undefined) {
              inst.state = initialState = null;
          }

          // 初始化更新队列
          this._pendingStateQueue = null;
          this._pendingReplaceState = false;
          this._pendingForceUpdate = false;

          var markup;
          // 如果挂载时出现错误，进行错误处理，然后performInitialMount，初始化挂载
          if (inst.unstable_handleError) {
              markup = this.performInitialMountWithErrorHandling(renderedElement, nativeParent,
                  nativeContainerInfo, transaction, context);
          } else {
              // 执行初始化挂载
              markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction,
                  context);
          }

          // 如果存在 componentDidMount，则调用
          if (inst.componentDidMount) {
              transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
          }

          return markup;
      },

      performInitialMountWithErrorHandling: function (renderedElement, nativeParent, nativeContainerInfo,
                                                      transaction, context) {
          var markup;
          var checkpoint = transaction.checkpoint();

          try {
              // 捕捉错误，如果没有错误，则初始化挂载
              markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction,
                  context);
          } catch (e) {
              // 发现错误，卸载组件，然后重新performInitialMount初始化挂载
              transaction.rollback(checkpoint);
              this._instance.unstable_handleError(e);
              if (this._pendingStateQueue) {
                  this._instance.state = this._processPendingState(this._instance.props, this._instance.context);
              }
              checkpoint = transaction.checkpoint();
              this._renderedComponent.unmountComponent(true);
              transaction.rollback(checkpoint);

              markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction,
                  context);
          }
          return markup;
      },

      performInitialMount: function (renderedElement, nativeParent, nativeContainerInfo, transaction,
                                    context) {
          var inst = this._instance;
          // render前，如果存在 componentWillMount，则调用
          if (inst.componentWillMount) {
              inst.componentWillMount();
              // componentWillMount 调用 setState 时，不会触发 re-render 而是自动提前合并
              if (this._pendingStateQueue) {
                  inst.state = this._processPendingState(inst.props, inst.context);
              }
          }

          // 如果不是无状态组件，即可以开始render
          if (renderedElement === undefined) {
              renderedElement = this._renderValidatedComponent();
          }

          this._renderedNodeType = ReactNodeTypes.getType(renderedElement);
          // 得到 _currentElement 对应的 component 类实例
          this._renderedComponent = this._instantiateReactComponent(
              renderedElement
          );
          // render 递归渲染，渲染子组件
          var markup = ReactReconciler.mountComponent(this._renderedComponent, transaction, nativeParent,
              nativeContainerInfo, this._processChildContext(context));

          return markup;
      }
    }

`mountComponent`负责管理生命周期中的`getInitial`、`componentWillMount`、`render`、`componentDidMount`; `ReactCompositeComponent`自定义组件类。     
首先将实例对象的`props`、`state`、队列等初始化，然后调用`performInitialMount`挂载组件，这里有个容错，如果存在`componentWillMount`，则执行(如果此时在`componentWillMount`中调用`setSate`,是不会重新触发`render`，而是自动合并`state`)，完成挂载后调用`componentDidMount`。       

#### **更新执行阶段**(顺序执行)       
_componentWillReceiveProps(nextProps, nextContext)_：组件接收到新的props时调用。           
_shouldComponentUpdate(nextProps, nextState, nextContext)_：接受到新的props和state，判断是否需要渲染，渲染前调用。        
_componentWillUpdate(nextProps, nextState, nextContext)_：接受到新的props和state，渲染前调用。    
_render()_。                             
_componentDidUpdate(prevProps, prevState, prevContext)_：组件渲染后，更新到Dom之后调用。       

    var ReactCompositeComponent = {
      /**
      * 更新已经渲染的组件。componentWillReceiveProps和shouldComponentUpdate方法将会被调用
      * 然后,(更新的过程没有被省略)，其余的更新阶段的生命周期都会被调用，对应的DOM会被更新。
      * @param {ReactReconcileTransaction} transaction
      * @param {ReactElement} prevParentElement
      * @param {ReactElement} nextParentElement
      * @internal
      * @overridable
      */

      updateComponent: function (transaction, prevParentElement, nextParentElement, prevUnmaskedContext, nextUnmaskedContext) {
          var inst = this._instance;
          var willReceive = false;
          var nextContext;
          var nextProps;

          // context对象如果有变化，则检查propTYPES,在开发阶段可以报错提醒
          if (this._context === nextUnmaskedContext) {
              nextContext = inst.context;
          } else {
              nextContext = this._processContext(nextUnmaskedContext);
              willReceive = true;
          }

          var prevProps = prevParentElement.props;
          var nextProps = nextParentElement.props;

          // 如果父元素类型相同，则跳过类型检查
          if (prevParentElement !== nextParentElement) {
              willReceive = true;
          }

          // 如果存在 componentWillReceiveProps，通过setState进入的updateComponent，则没有这一步
          if (willReceive && inst.componentWillReceiveProps) {
              inst.componentWillReceiveProps(nextProps, nextContext);
          }

          // 将新的 state 合并到更新队列中，此时 nextState 为最新的 state；
          // componentWillReceiveProps中调用setState不会重新渲染
          var nextState = this._processPendingState(nextProps, nextContext);

          // 根据更新队列和 shouldComponentUpdate 的状态来判断是否需要更新组件
          var shouldUpdate =
              this._pendingForceUpdate || !inst.shouldComponentUpdate ||
              inst.shouldComponentUpdate(nextProps, nextState, nextContext);

          if (shouldUpdate) {
              // 重置更新队列
              this._pendingForceUpdate = false;
              // 即将更新 this.props、this.state 和 this.context
              this._performComponentUpdate(nextParentElement, nextProps, nextState, nextContext, transaction,
                  nextUnmaskedContext);
          } else {
              // 如果确定组件不更新，仍然要设置 props 和 state
              this._currentElement = nextParentElement;
              this._context = nextUnmaskedContext;
              inst.props = nextProps;
              inst.state = nextState;
              inst.context = nextContext;
          }
      },

      //当确定组件需要更新时，则调用
      _performComponentUpdate: function (nextElement, nextProps, nextState, nextContext, transaction, unmaskedContext) {
          var inst = this._instance;
          var hasComponentDidUpdate = Boolean(inst.componentDidUpdate);
          var prevProps;
          var prevState;
          var prevContext;

          // 如果存在 componentDidUpdate，则将当前的 props、state 和 context 保存一份
          if (hasComponentDidUpdate) {
              prevProps = inst.props;
              prevState = inst.state;
              prevContext = inst.context;
          }

          // 如果存在 componentWillUpdate，则调用
          if (inst.componentWillUpdate) {
              inst.componentWillUpdate(nextProps, nextState, nextContext);
          }

          this._currentElement = nextElement;
          this._context = unmaskedContext;

          // 更新 this.props、this.state 和 this.context
          inst.props = nextProps;
          inst.state = nextState;
          inst.context = nextContext;

          // 递归调用 render 渲染组件
          this._updateRenderedComponent(transaction, unmaskedContext);

          // 当组件完成更新后，如果存在 componentDidUpdate，则调用
          if (hasComponentDidUpdate) {
              transaction.getReactMountReady().enqueue(
                  inst.componentDidUpdate.bind(inst, prevProps, prevState, prevContext),
                  inst
              );
          }
      },

      //调用render渲染组件
      _updateRenderedComponent: function(transaction, context) {
          var prevComponentInstance = this._renderedComponent;
          var prevRenderedElement = prevComponentInstance._currentElement;

          // _renderValidatedComponent内部会调用render,得到ReactElement
          var nextRenderedElement = this._renderValidatedComponent();

          // 判断是否做DOM diff。React为了简化递归diff,认为组件层级不变,且type和key不变(key用于listView等组件,很多时候我们没有设置type)才update,否则先unmount再重新mount
          if (shouldUpdateReactComponent(prevRenderedElement, nextRenderedElement)) {
            // 递归updateComponent,更新子组件的Virtual DOM
            ReactReconciler.receiveComponent(prevComponentInstance, nextRenderedElement, transaction, this._processChildContext(context));
          } else {
            var oldNativeNode = ReactReconciler.getNativeNode(prevComponentInstance);

            // 不做DOM diff,则先卸载掉,然后再加载。也就是先unMountComponent,再mountComponent
            ReactReconciler.unmountComponent(prevComponentInstance, false);

            this._renderedNodeType = ReactNodeTypes.getType(nextRenderedElement);

            // 由ReactElement创建ReactComponent
            this._renderedComponent = this._instantiateReactComponent(nextRenderedElement);

            // mountComponent挂载组件,得到组件对应HTML
            var nextMarkup = ReactReconciler.mountComponent(this._renderedComponent, transaction, this._nativeParent, this._nativeContainerInfo, this._processChildContext(context));

            // 将HTML插入DOM中
            this._replaceNodeWithMarkup(oldNativeNode, nextMarkup, prevComponentInstance);
          }
      },

      _renderValidatedComponent: function() {
          var renderedComponent;
          ReactCurrentOwner.current = this;
          try {
            renderedComponent =
              this._renderValidatedComponentWithoutOwnerOrContext();
          } finally {
            ReactCurrentOwner.current = null;
          }

          return renderedComponent;
      },

      _renderValidatedComponentWithoutOwnerOrContext: function() {
          var inst = this._instance;
          // 执行render，渲染
          var renderedComponent = inst.render();

          return renderedComponent;
      },
    }

`updateComponent`负责管理生命周期中的`componentWillReceiveProps`、`shouldComponentUpdate`、`componentWillUpdate`、`render`和`componentDidUpdate`。          
首先判断前后元素不一致，`context`发生变化时，若存在`componentWillReceiveProps`，执行(如果此时`componentWillReceiveProps`中国调用`setState`，是不会重新触发`render`，而是自动合并`state`,这时我们得不到新的`state`)，我们会发现`parentElement`发生变化时会触发`componentWillReceiveProps`，通过`parentElement`储存的信息我们得出：如果父组件的`props`等属性发生改变时，即使这个改变的属性没有传入到子组件，但是也会调用`componentWillReceiveProps`的执行；更新`state`。     

    // parentElement
    {
        $$typeof:Symbol(react.element),
        key:null,
        props:Object,
        ref:null,
        type: function Example(props),
        _owner:ReactCompositeComponentWrapper,
        _store:Object,
        _self:App,
        _source:Object,
        __proto__:Object
    }

如果存在`componentDidUpdate`则需要将`state`、`props`和`context`进行复制备份。
调用`shouldComponentUpdate`判断组件是否需要更新，如果需要更新，存在`componentWillUpdate`，执行，递归调用；如果返回false，组件中的`props`和`state`也都会被更新(如果调用了`forceUpdate`函数的话，会跳过`shouldComponnetUpdate`的判断过程)；`render`之后，若存在`componnetDidUpdate`，则调用执行，并将更新前的数据当作参数返回出来。    

* 注意：
> 我们不能在 `shouldComponent` 和 `componentWillUpdate` 中调用 `setState` ,如果调用了则会导致再次调用 `updateComponent` ，会导致循环调用，直到浏览器内存崩溃。        

#### **组件卸载**      
_componnetWillUnmount()_：销毁组件。    

    unmountComponent: function(safely) {
      if (!this._renderedComponent) {
        return;
      }
      var inst = this._instance;

      // 调用componentWillUnmount
      if (inst.componentWillUnmount && !inst._calledComponentWillUnmount) {
        inst._calledComponentWillUnmount = true;
        // 安全模式下，将componentWillUnmount放在try-catch中。否则直接componentWillUnmount
        if (safely) {
          var name = this.getName() + '.componentWillUnmount()';
          ReactErrorUtils.invokeGuardedCallback(name, inst.componentWillUnmount.bind(inst));
        } else {
          inst.componentWillUnmount();
        }
      }

      // 递归调用unMountComponent来销毁子组件
      if (this._renderedComponent) {
        ReactReconciler.unmountComponent(this._renderedComponent, safely);
        this._renderedNodeType = null;
        this._renderedComponent = null;
        this._instance = null;
      }

      // reset等待队列和其他等待状态
      this._pendingStateQueue = null;
      this._pendingReplaceState = false;
      this._pendingForceUpdate = false;
      this._pendingCallbacks = null;
      this._pendingElement = null;

      // reset内部变量,防止内存泄漏
      this._context = null;
      this._rootNodeID = null;
      this._topLevelWrapper = null;

      // 将组件从map中移除,还记得我们在mountComponent中将它加入了map中的吧
      ReactInstanceMap.remove(inst);
    },

`unmountComponent`负责管理生命周期中的`componentWillUnmount`。如果在`componentWillUnmount`中调用`setState`，是不会重新触发render的。

### react组件实例的创建、组件插入dom       
#### react组件实例的创建     
_React.createClass_方法用于生成一个组件的实例。在前面的组件生命周期里面`getDefaultProps`就在`createClass`里面出现，我们简答介绍一下源码的部分内容：             
+ 定义了构造函数Constructor，构造函数的prototype指向了一个ReactClassComponent的实例。                   
    Constructor.prototype = new ReactClassComponent();          
+ 在Constructor原型上定义了一个`\_reactAutoBindPairs`的空数组，我们自定义的方法按照[key,content,key,content,...]的方法存在了`\_reactAutoBindPairs`里面，为了组件实例化的时候直接绑定到实例上，并且避免了react官方制定的方法也被绑定到实例上。       
+ 调用了`getDefaultProps`，并且把值放到defaultProps类变量上。                
+ 我们创建一个组件自定义的方法都在`ReactClassInterface`上有，当前未定义的方法设置为空。     

    for (var methodName in ReactClassInterface) {
        if (!Constructor.prototype[methodName]) {
          Constructor.prototype[methodName] = null;
        }
      }    
+ 在构造函数里主要做了两件事，一就是把__reactAutoBindPairs的内容取出来，自动绑定到实例上；二就是初始化一些参数，调用`getInitialState()`，返回sate的值。        

    var Constructor = function (props, context, updater) {
      // 触发自动绑定
      if (this.__reactAutoBindPairs.length) {
        bindAutoBindMethods(this);
      }

      // 初始化参数
      this.props = props;
      this.context = context;
      this.refs = emptyObject;  // 本组件对象的引用，可以利用它来调用组件的方法
      this.updater = updater || ReactNoopUpdateQueue;

      // 调用getInitialState()来初始化state变量
      this.state = null;
      var initialState = this.getInitialState ? this.getInitialState() : null;
      this.state = initialState;
    };

#### 组件插入dom
_ReactDom.render()_方法用于插入dom元素。            

    ReactDom.render(
      <p>插入dom元素</p>,
      document.getElementById('root')         //id为root的为一个根节点
    ); 

上述代码经过babel转义过来就是：

    ReactDom.render(
      // React.createElement(type, config, children)
      React.createElement('p', '插入dom元素'),
      document.getElementById('root')
    )

**ReactElement()** `ReactElement`就是我们常说的虚拟dom。首先我们来看一下`ReactElement`的源码：       

    var ReactElement = function(type, key, ref, self, source, owner, props) {
      var element = {
        // This tag allow us to uniquely identify this as a React Element
        $$typeof: REACT_ELEMENT_TYPE,
        // Built-in properties that belong on the element
        type: type,
        key: key,
        ref: ref,
        props: props,
        // Record the component responsible for creating this element.
        _owner: owner,
      };
      return element;
    };

在mountComponent()挂载组件中，会进行组件渲染，调用到`instantiateReactComponent()`方法。`instantiateReactComponent()`会根据`ReactElement`中不同type，创建不同类型的组件对象。           

    // 初始化组件对象，node是一个ReactElement对象，是节点元素在React中的表示
    function instantiateReactComponent(node) {
      var instance;

      var isEmpty = node === null || node === false;
      if (isEmpty) {
        // 空对象
        instance = ReactEmptyComponent.create(instantiateReactComponent);
      } else if (typeof node === 'object') {
        // 组件对象，包括DOM原生的和React自定义组件
        var element = node;

        // 根据ReactElement中的type字段区分
        if (typeof element.type === 'string') {
          // type为string则表示DOM原生对象，比如div span等。
          instance = ReactNativeComponent.createInternalComponent(element);
        } else if (isInternalComponentType(element.type)) {
          // 保留给以后版本使用，此处暂时不会涉及到
          instance = new element.type(element);
        } else {
          // React自定义组件
          instance = new ReactCompositeComponentWrapper(element);
        }
      } else if (typeof node === 'string' || typeof node === 'number') {
        // 元素是一个string时，对应的比如<span>123</span> 中的123
        // 本质上它不是一个ReactElement，但为了统一，也按照同样流程处理，称为ReactDOMTextComponent
        instance = ReactNativeComponent.createInstanceForText(node);
      } else {
        // 无处理
      }

      // 初始化参数，这两个参数是DOM diff时用到的
      instance._mountIndex = 0;
      instance._mountImage = null;

      return instance;
    }

**createElement()**`createElement()`创建React元素节点    

    ReactElement.createElement = function(type, config, children) {
      var propName;

      // 初始化参数
      var props = {};

      var key = null;
      var ref = null;
      var self = null;
      var source = null;
      //从config中提取出ref，key，self，source等属性
      if (config != null) {
        if (hasValidRef(config)) {
          ref = config.ref;
        }
        if (hasValidKey(config)) {
          key = '' + config.key;
        }

        self = config.__self === undefined ? null : config.__self;
        source = config.__source === undefined ? null : config.__source;
        // 从config中取出属性放到新的props对象中
        for (propName in config) {
          if (
            hasOwnProperty.call(config, propName) &&
            !RESERVED_PROPS.hasOwnProperty(propName)
          ) {
            props[propName] = config[propName];
          }
        }
      }

      // 传过来的children被放在了props的children属性下
      // 传入的参数前两个为type和config，所以要减2
      var childrenLength = arguments.length - 2;
      if (childrenLength === 1) {
        //如果只有一个时，直接放到props.children下
        props.children = children;
      } else if (childrenLength > 1) {
        // 不止一个的时候以array的方式存放到props.children下
        var childArray = Array(childrenLength);
        for (var i = 0; i < childrenLength; i++) {
          childArray[i] = arguments[i + 2];
        }
        props.children = childArray;
      }

      // 静态变量defaultProps，属性设置默认值
      if (type && type.defaultProps) {
        var defaultProps = type.defaultProps;
        for (propName in defaultProps) {
          if (props[propName] === undefined) {
            props[propName] = defaultProps[propName];
          }
        }
      }
      //返回ReactElement对象
      return ReactElement(
        type,
        key,
        ref,
        self,
        source,
        ReactCurrentOwner.current,
        props,
      );
    };

ReactDOM.render()实际上是调用ReactMount.render()；接着调用ReactMount.\_renderSubtreeIntoContainer，在这里面调用\_renderNewRootComponent并生成一个ReactCompositeComponentWrapper并返回。                

`_renderSubtreeIntoContainer()`源码：      

    _renderSubtreeIntoContainer: function (parentComponent, nextElement, container, callback) {
        callback = callback === undefined ? null : callback;
        // 包装ReactElement，将nextElement挂载到wrapper的props属性下，这段代码不是很关键
        var nextWrappedElement = ReactElement(TopLevelWrapper, null, null, null, null, null, nextElement);
        // 获取要插入到的容器的前一次的ReactComponent，这是为了做DOM diff
        // 对于ReactDOM.render()调用，prevComponent为null
        var prevComponent = getTopLevelWrapperInContainer(container);

        if (prevComponent) {
          // 从prevComponent中获取到prevElement这个数据对象。一定要搞清楚ReactElement和ReactComponent的 作用，他们很关键
          var prevWrappedElement = prevComponent._currentElement;
          var prevElement = prevWrappedElement.props;
          // DOM diff精髓，同一层级内，type和key不变时，只用update就行。否则先unmount组件再mount组件
          // 这是React为了避免递归太深，而做的DOM diff前提假设。它只对同一DOM层级，type相同，key(如果有)相同的组件做DOM diff，否则不用比较，直接先unmount再mount。这个假设使得diff算法复杂度从O(n^3)降低为O(n).
          if (shouldUpdateReactComponent(prevElement, nextElement)) {
            var publicInst = prevComponent._renderedComponent.getPublicInstance();
            var updatedCallback = callback && function () {
              callback.call(publicInst);
            };
            // 只需要update，调用_updateRootComponent，然后直接return了
            ReactMount._updateRootComponent(prevComponent, nextWrappedElement, container, updatedCallback);
            return publicInst;
          } else {
            // 不做update，直接先卸载再挂载。即unmountComponent,再mountComponent。mountComponent在后面代码中进行
            ReactMount.unmountComponentAtNode(container);
          }
        }

        var reactRootElement = getReactRootElementInContainer(container);
        var containerHasReactMarkup = reactRootElement && !!internalGetID(reactRootElement);
        var containerHasNonRootReactChild = hasNonRootReactChild(container);

        var shouldReuseMarkup = containerHasReactMarkup && !prevComponent && !containerHasNonRootReactChild;
        // 初始化，渲染组件，然后插入到DOM中。_renderNewRootComponent很关键，后面详细分析
        var component = ReactMount._renderNewRootComponent(nextWrappedElement, container, shouldReuseMarkup, parentComponent != null ? parentComponent._reactInternalInstance._processChildContext(parentComponent._reactInternalInstance._context) : emptyObject)._renderedComponent.getPublicInstance();
        // render方法中带入的回调，ReactDOM.render()调用时一般不传入
        if (callback) {
          callback.call(component);
        }
        return component;
      },
    }