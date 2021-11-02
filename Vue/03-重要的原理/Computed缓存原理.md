+ 调用计算属性时会触发`Object.defineProperty`的get访问器函数
+ get访问器内部会继续调用`watcher.depend()`方法，向自身的消息订阅器dep的subs中添加其他属性的watcher
+ 继续调用`watcher.evaluate()`，执行getter求值函数