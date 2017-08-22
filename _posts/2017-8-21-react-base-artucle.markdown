---
layout:    post
title:     "React"
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

1.对于两棵树的比较只会对同一层的节点进行比较，彻底简化了复杂度。如下图，只会循环一次对每一层进行比较，如果发现节点不存在，则会把该节点和其子节点完全删除。 
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

 
2. **不同类型的节点：当树中的同一位置前后输出了不同类型的节点，React直接完全删除前面的节点以及它的子节点，然后创建并插入新的节点。**     
对于component来说也是同样的道理：
![img](/img/in-post/react-base-article/diff-second.png)   
通过逐层比较发现，Dcomponent变为了Gcomponent，即使这两个component结构相似但是如果判断为不同类型的componet，React就会直接删除Dcomponent，重新创建Gcomponent以及它的子节点。   

* **相同类型，不同属性的节点：这个就比较简单了即重新设置属性即可。对于虚拟DOM来说，style的属性值不是简单的字符串而是一个对象。**         

* **列节点的比较：允许开发者对同一层级的子节点，通过唯一的key值进行区分，大大提高了性能。**       
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


