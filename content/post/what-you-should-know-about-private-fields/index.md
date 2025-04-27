+++
slug = 'what-you-should-know-about-private-properties'
title = '你该了解的私有属性'
date = 2025-04-26T15:54:22+08:00
draft = false
tags = ['ES6', 'Class', 'Private properties']
categories = ['学习笔记']
image = 'cover.png'
+++

`ES6` 引入了私有属性和私有方法的标准写法，进一步增强了代码的封装性。然而私有属性的一些特性经常让人困惑，本文主要针对几种典型使用场景进行分析。


## 私有属性

`ES6` 中 `Class` 的元素可通过以下三个方面来描述：
-   种类：`getter`、`setter`、`method`、`field`
-   位置：`static`、`instance`
-   可见性：`public` 、`private`

总共有16种组合方式。

私有属性( `private properties` )包含以下几种类型：

-   `private fields`
-   `private methods`
-   `private getters`
-   `private setters`
-   `private static fields`
-   `private static methods`
-   `private static getters`
-   `private static setters`

私有属性通过添加 `#` 前缀来创建:

```js
    class ClassWithPrivate {
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
类中所有声明的私有标识符都必须是唯一的，实例属性和静态属性共享命名空间， 唯一的特例是相同名称的**getter**和**setter**。

私有属性只能在**定义它的类中**通过`点表示法`访问，**不会被子类继承**。




## 求值顺序

我们先大致了解一下当**class声明**或**class表达式**被求值时，class中各组件的求值顺序，这对理解后面的示例会有所帮助：

1.  如果有 `extends` 子句的话，子句会被先求值；
2.  提取 `constructor` 方法，如果不存在，则会替换成默认实现；
3.  按照声明顺序对类元素的属性 `key` 求值；
4.  按照声明顺序安装 `methods` 和 `accessors`。
    - `public instance methods` 和 `public instance accessors` 会被安装到类的 `prototype` 属性上；
    - `public & private static methods` 和 `public & private static accessors`则会被安装到类本身；
    - `private instance methods` 和 `private instance accessors` 会在**后续**（详见步骤5）直接安装到实例本身。
5.  类元素的 `value` 按照声明顺序求值：
    -   `public & private instance fields`，其初始化器表达式会被保存。在创建实例时，在 `constructor` 的开头（对于父类）或在`super()` return之后（对于子类）对初始化器进行求值；
    `private instance methods` 和 `private instance accessors` 则会在 `instance fields` 安装之前立即被安装，且**仅在实例上可用，而不能通过类的`prototype`访问**。
    -   `public & private static fields`，其初始化器在求值时，this指向类本身，而且属性会被创建到类上；
    -   `static initialization block` 在被求值时，this指向类本身


## 示例

### 在super()中访问public instance fields
{{< highlight js "linenos=table,hl_lines=5" >}}
class Parent {
  p = 'parent';

  constructor() {
    console.log(this.getP()); // 'parent'
  }
}

class Child extends Parent {
  p = 'child';

  constructor() {
    super();
  }

  getP() {
    return this.p;
  }
}

const child = new Child(); 
{{< /highlight >}}

在调用 `super()` 前，Child类的**实例方法**和**原型链**就已经初始化好了，而 `super()` 中的 `this` 指向**子类实例**，所以在 `super()` 中可以通过 `this` 访问Child类的`getP()` 方法。由于子类的**实例字段**要等到 `super()` 结束后才会初始化，此时Child类的实例字段 `p` 还不可用，而父类的实例字段是在 `constructor` 开头初始化的，所以此时访问到的 `p` 是父类的属性，因此打印的是`'parent'` 。

### 在super()之后访问public instance fields
{{< highlight js "linenos=table,hl_lines=14" >}}
class Parent {
  p = 'parent';

  constructor() {
    console.log(this.getP()); // 'parent'
  }
}

class Child extends Parent {
  p = 'child';

  constructor() {
    super();
    console.log(this.getP()); // 'child'
  }

  getP() {
    return this.p;
  }
}

const child = new Child(); 
{{< /highlight >}}

将前一个例子稍微修改一下，在调用 `super()` 完成之后再访问 `this.p` ,此时子类的实例字段已经初始化完成，所以可以访问到。

### 在super()中访问private instance fields
#### 调用子类方法访问
{{< highlight js "linenos=table,hl_lines=5" >}}
class Parent {
  #p = 'parent';

  constructor() {
    console.log(this.getP()); // Uncaught TypeError: Cannot read private member #p from an object whose class did not declare it
  }
}

class Child extends Parent {
  #p = 'child';

  getP() {
    return this.#p;
  }
}

new Child();
{{< /highlight >}}

执行会报错，但实际上并不是Child类没有定义该私有属性，而是当父类的 `constructor` 还在执行时，子类的私有实例属性还未初始化好，所以无法访问（**私有属性不可继承，只能在定义它的类内部访问， 访问不存在的私有属性会报错**）。

#### 调用父类方法访问
而当我们把 `getP()` 方法移到Parent类中，情况又不一样：
{{< highlight js "linenos=table,hl_lines=5 8-10" >}}
class Parent {
  #p = 'parent';

  constructor() {
    console.log(this.getP()); // 'parent'
  }

  getP() {
    return this.#p;
  }
}

class Child extends Parent {
  #p = 'child';
}

new Child();
{{< /highlight >}}

按照上面的思路分析，应该也报错才对，但是实际却打印了父类的私有属性。原因在于私有属性的特殊性：
1. **私有字段与定义它的类绑定，不能被继承或覆盖**。（可以将其看成附到实例上的由类管理的
外部元数据。）
2. **方法与定义它的类中的私有属性绑定**。即使父类方法被子类继承，通过子类实例调用方法时依然访问的是父类（方法定义时的类）的私有属性。

上面例子中的`getP()` 方法是在父类声明的，所以绑定了父类的私有属性。子类在实例化时已经将父类的私有属性初始化到自身了，所以`getP()` 能够访问到父类的那部分私有属性，故而打印`'parent'`。同时最终生成的实例对象中既包含父类的`#p`,也包含子类的`#p`,这两个私有属性虽然同名，但是却是相互隔离的两个属性。

回顾[前一个例子](#调用子类方法访问)中的 `getP()` 由于是在子类中声明的，所以绑定了子类的私有属性，而这部分私有数据在`getP()` 被调用时还未初始化，所以访问报错。



### 访问private static fields
```js
class Parent {
  static #p = 42;

  static getP() {
    return this.#p;
  }
}

class Child extends Parent {}

console.log(Child.getP()); // Uncaught TypeError: Cannot read private member #p from an object whose class did not declare it

```
因为`getP()` 方法是在父类中声明的，会和父类的私有属性绑定，通过`Child.getP()` 调用时 `this` 指向子类 `Child` , 在`Child` 类上无法找到`Parent` 相关私有属性而报错。

因此推荐使用 **类名** 而不是`this` 访问静态私有属性。

{{< highlight js "linenos=table,hl_lines=5" >}}

  class Parent {
    static #p = 42;

    static getP() {
      return Parent.#p;
    }
  }
{{< /highlight >}}


### 访问private instance methods & accessors

```js
class C {
  #method() {}

  get #x() { return 'getter' ;}

  static getMethod(obj) {
    return obj.#method;
  }

  static getGetter(obj) {
    return obj.#x;
  }
}

const c = new C();
console.log(C.getMethod(c)); // ƒ #method() {}
console.log(C.getGetter(c)); // 'getter'
console.log(C.getMethod(C.prototype)); // TypeError
console.log(C.getGetter(C.prototype)); // TypeError

```

`private instance methods` 和 `private instance accessors` 仅在实例上可用，而不是在类的 `prototype` 上。

## 总结
1. 公有属性可以被继承，私有属性不能被继承, 私有属性只能在定义它的类中使用`点表示法`访问， 外部无法访问；
2. 私有属性和定义它的类绑定，类的方法和定义它的类中的私有属性绑定，方法中访问的私有属性是该方法被定义时的类中的私有属性，如果访问不了就会报错；
3. 推荐使用**类名**而不是 `this` 访问类的静态私有属性

## 参考
- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)


