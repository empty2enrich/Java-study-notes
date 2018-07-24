# Java 基础总结

 + [1、enum](#1)
 + [2、==](#2)
 + 

## <p id=1>1、enum</p>

enum class 中的构造方法必须私有 private

```java
public enum Color {
    RED("红色"), //RED 是一个 Color 类实例
    BLACK("黑色");
    
    private Color(String desc){};
    private   Color(){};

}
```

## <p id=2>2、==</p>

 + Object == Object ： 只要指向同一个对象,返回 true , 否则 false