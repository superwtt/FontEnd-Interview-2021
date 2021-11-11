#### diff算法与patch的过程

数据更新时，渲染得到新的`Virtual DOM`，与上一次得到的`Virtual DOM`进行diff，记录下所有需要在DOM上进行的变更然后在**`patch`**的过程中应用到DOM上，实现UI的同步更新

---

##### 定义

1. diff算法：一种对比新、旧虚拟节点的算法
2. patch：打补丁，把新旧节点的差异应用到DOM上

---

##### 分析diff算法

1. diff的比较只会在同级进行，不会跨层级比较

   ![](https://raw.githubusercontent.com/aooy/blog/master/images/issues-2/diff.png)

---

##### 源码分析

diff的过程就是调用`patch函数`，像打补丁一样修改真实dom

patch函数有两个参数，vnode和oldNode，也就是新旧两个虚拟节点。

① 先看下`patch函数`长什么样子

```javascript
function patch (oldVnode, vnode) {
	if (sameVnode(oldVnode, vnode)) {
		patchVnode(oldVnode, vnode)
	} else {
		const oEl = oldVnode.el
		let parentEle = api.parentNode(oEl)
		createEle(vnode)
		if (parentEle !== null) {
			api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
			api.removeChild(parentEle, oldVnode.el)
			oldVnode = null
		}
	}
	return vnode
}

```

解析：

+ `patch函数`有两个参数，`vnode`和`oldNode`，也就是新旧两个虚拟节点。

+ 一个完整的vnode属性如下：

  ```javascript
  // body下的 <div id="v" class="classA"><div> 对应的 oldVnode 就是
  
  {
    el:  div  //对真实的节点的引用，本例中就是document.querySelector('#id.classA')
    tagName: 'DIV',   //节点的标签
    sel: 'div#v.classA'  //节点的选择器
    data: null,       // 一个存储节点属性的对象，对应节点的el[prop]属性，例如onclick , style
    children: [], //存储子节点的数组，每个子节点也是vnode结构
    text: null,    //如果是文本节点，对应文本节点的textContent，否则为null
  }
  ```

---

② `patch`的第一部分 `if分支`

```javascript
if (sameVnode(oldVnode, vnode)) {
	patchVnode(oldVnode, vnode)
}

function sameVnode(oldVnode, vnode){
	return vnode.key === oldVnode.key && vnode.sel === oldVnode.sel
}
```

解析：

+ `sameNode`函数就是看这两个节点是否值得比较，<font style="color:red">**两个vnode节点的key和sel相同才去比较它们**</font>。
  + 如果值得比较进入`patchNode函数` 稍后详细讲解
  + 如果不值得比较进入`else分支`

---

③ `patch`的第二部分 `else分支`

```javascript
else {
		const oEl = oldVnode.el
		let parentEle = api.parentNode(oEl)
		createEle(vnode)
		if (parentEle !== null) {
			api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
			api.removeChild(parentEle, oldVnode.el)
			oldVnode = null
		}
	}

```

解析：

+ 取得`oldVnode.el`的父节点
+ `createEle(vnode)`会为vnode创建它的真实dom
+ `parentEle`将新的dom插入，移除旧的dom

<font style="color:red">**当不值得比较时，新节点直接把老节点整个替换了**</font>

---

④  `patch`的最后

```javascript
return vnode
```

至此完成一个patch过程

---

##### patchVnode

当两个节点值得比较时，会调用`patchVnode函数`

```javascript
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el
    let i, oldCh = oldVnode.children, ch = vnode.children
    if (oldVnode === vnode) return
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        api.setTextContent(el, vnode.text)
    }else {
        updateEle(el, vnode, oldVnode)
    	if (oldCh && ch && oldCh !== ch) {
	    	updateChildren(el, oldCh, ch)
	    }else if (ch){
	    	createEle(vnode) //create el's children dom
	    }else if (oldCh){
	    	api.removeChildren(el)
	    }
    }
}
```

解析：节点的比较有五种情况：

1）`if(oldNode === vnode)`，他们的引用一致，可以认为没有变化

2）`if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text)`，文本节点的比较，若需要修改，则会调用`Node.textContent = vnode.text`去修改

3）`if( oldCh && ch && oldCh !== ch )`，两个节点都有子节点，并且他们不一样，这样会调用`updateChildren`函数比较子节点，这是`diff的核心`，后面会降到

4）`else if (ch)`，只有新的节点有子节点，调用`createEle(vnode)`，`vnode.el`已经引用了老的dom节点，`createEle`函数会在老dom节点上添加子节点

5）`else if (oldCh)`，新节点没有子节点，老节点有子节点，直接删除老节点。

---

##### updateChildren

`updateChildren`是Vue diff的核心，其过程可以概括为：

- `旧children`和`新children`各有两个头尾的变量`StartIdx`和`EndIdx`，它们的2个变量相互比较，一共有4种比较方式。
- 如果4种比较都没匹配，如果设置了key，就会用key进行比较，在比较的过程中，变量会往中间靠，一旦`StartIdx>EndIdx`表明`旧children`和`新children`至少有一个已经遍历完了，就会结束比较。













---

#### 具体的diff分析

设置key和不设置key的区别：

不设置key，newCh和oldCh只会进行头尾两端的相互比较。设置了key后，除了头尾两端的比较外，还会从用key生成的对象`oldKeyToIdx`中查找匹配的节点，所以为节点设置key可以更高效的利用dom

1. 不设置key：

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/不设置key.png)



2. 设置key，b节点得到了复用

   ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/设置key.png)

---

#### 总结

1. Vue2.0中diff的流程就是调用patch的流程：

   + 先使用key看看新旧两个vnode值不值得比较，如果不值得比较，直接删除旧的节点创建新的节点。如果值得比较，就进入patchNode打补丁的环节。
   + 如果值得比较，就会调用`updateChildren`。`updateChildren`是Vue diff算法的核心：首尾两端比较，一共有4种比较方式，直到startIndex>endIndex才结束比较。将比较的结果直接打补丁patchNode在最终的DOM上

   



https://github.com/chanshiyucx/blog/issues/71

https://zhuanlan.zhihu.com/p/76384873

https://segmentfault.com/a/1190000021896771

https://jishuin.proginn.com/p/763bfbd54ab9