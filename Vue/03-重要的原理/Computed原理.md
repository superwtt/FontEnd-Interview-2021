#### Computed

Vue中的计算属性

---

##### 基本使用

```javascript
computed:{
  // 作为方法使用
  message(){
    return "这是message经过计算后的值"  
  },
  // 作为对象使用
  assign:{
    set: function(){
       this.testData = "这是testData取值时候的值" 
    },
    get:function(){
       return "get的时候拿上这个值"+this.testData 
    }
  }    
}
```

解析：

1. 定义一个计算属性有两种