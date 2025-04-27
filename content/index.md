`ES6`的`Class`元素可通过以下三个方面来描述：
- 种类：`getter`、`setter`、`method`、`field`
- 位置：`static`、`instance`
- 可见性：`public` 、`private`
总共有16种组合方式。私有属性包含以下几种类型：

-   `private fields`
-   `private methods`
-   `private getters`
-   `private setters`
-   `private static fields`
-   `private static methods`
-   `private static getters`
-   `private static setters`

 
## 求值顺序

Class中各成员的求值顺序大致为：
1. 如果有`extends`子句的话，它会被先求值；
2. 提取`constructor`方法，如果不存在，则会替换成默认实现；
3. 按照声明顺序对类元素的属性`key`求值；
4. 按照声明顺序安装`methods`和`accessors`。`(public)instance methods`和`(public)instance accessors`会被安装到类的`prototype`属性上；`static methods(public & private)`和`static accessors(public & private)`会被安装到类本身；`private instance methods`和`private instance accessors`会在后续直接安装到实例本身。
5. 类元素的`value`按照声明顺序求值：
  - `instance fields(public & private)`，其初始化器表达式会被保存。在创建实例时，在constructor的开头（对于父类）或在`super()`返回前（对于子类）对初始化器进行求值；`private instance methods`和`private instance accessors`会在`instance fields`安装之前立即被安装，且仅在实例上可用，而不能通过类的`prototype`访问。
  - `static fields(public & private)`，其初始化器在求值时，this指向类本身，而且属性会被创建到类上；
  - `static initialization block`在被求值时，this指向类本身

S6的Class私有属性（`private properties`）私有属性只能在当前类的内部访问，不能被子类继承。

## 私有属性



私有属性通过`#` + `属性名`方式定义：
```
Class ClassWithPrivate {
  // private fields
  #privateField;
  #privateFieldWithInitializer = 42;

  // private methods
  #privateMethod() {
    // ...
  }

  // private getters
  get #privateGetter() {
    // ...
  }

  // private setters
  set #privateSetter(value) {
    // ...
  }

  // private static fields
  static #privateStaticField;
  static #privateStaticFieldWithInitializer = 42;

  // private static methods
  static #privateStaticMethod() {
    // ...
  }

  // private static getters
  static get #privateStaticGetter() {
    // ...
  }

  // private static setters
  static set #privateStaticSetter(value) {
    // ...
  }
}
```

> 需要注意的是，私有属性的标识符声明必须唯一，实例属性和静态属性共享命名空间。

这里简单探讨一下`private fields`和`private static fields`在继承中的