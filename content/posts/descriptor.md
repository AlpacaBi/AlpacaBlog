+++
title = "装饰器Descriptor：一个可能是未来JavaScript的新语法"
date = "2020-12-15T10:16:33+08:00"
author = "Alpaca Bi"
cover = ""
tags = ["TypeScript", "JavaScript"]
categories = ["前端"]
keywords = ["", ""]
description = "`Decorator`是ES7的一个新语法，目前仍处于[第2阶段提案](https://github.com/tc39/proposal-decorators)中,不过在TypeScript中已经实装。它能实现前端的`AOP`思想"
showFullContent = false
+++

首先要说一点，目前`Decorator`已经在在TypeScript中实装，所以下面的代码基本都是TypeScript代码，为了照顾没了解过TypeScript的小伙伴，例子里面关掉了ts类型检查，所以全是AnyScript,所以你基本上不会看到TypeScript的各种语法。

# <span style="color:#F90">1.脑子先别带东西，先看看下面的场景 </span>

我一般不喜欢一上来就说定义啊、用法之类的东西，我喜欢先从实际场景开始。

先看看下面的代码

```ts
class Human {
    constructor(name){
        this.name = name
    }
    name
    said() {
        console.log(`hi, my name is ${this.name}!!!\n`)
    }
}

class Cat {
    meow() {
        console.log(`meow!!!!`)
    }
}

let jack = new Human("jack")
jack.said() // hi, my name is jack!!!

let tom = new Cat()
tom.meow() //  meow!!!!
```
基本上都知道是什么意思吧，就是有一个`Human`类和`Cat`类，`Human`类具有`name`属性和`said`方法，而`Cat`类只有`meow`方法。

而现在有一个需求，就是要给`Human`类和`Cat`类加上一个设置年龄和显示年龄的业务逻辑，仔细点说就是加上一个`age`属性、`setAge`方法和一个`getAge`方法，该怎么办呢？？？


#### 1.1.直接手动给这两个类添加

最直观的方法当然是，直接手动给这两个类添加，如下：
```ts
class Human {
    constructor(name){
        this.name = name
    }

    name
    said() {
        console.log(`hi, my name is ${this.name}!!!`)
    }

    // 向Human手动添加age、setAge和getAge逻辑
    age
    setAge(age) {
        this.age = age
    }
    getAge() {
        console.log(`I am ${this.age} years old now!!!\n`)
    }
}

class Cat {
    meow() {
        console.log(`meow!!!!`)
    }

    // 向Cat手动添加age、setAge和getAge逻辑
    age
    setAge(age) {
        this.age = age
    }
    getAge() {
        console.log(`I am ${this.age} years old now!!!\n`)
    }
}


let jack = new Human("jack")
jack.said() // hi, my name is jack!!!
jack.setAge(22)
jack.getAge() // I am 22 years old now!!!

let tom = new Cat()
tom.meow() // meow!!!!
tom.setAge(3)
tom.getAge() // I am 3 years old now!!!
```

这样当然能实现需求了，但是这么搞技术含量就有点低了，因为`age`、`setAge`和`getAge`的逻辑和联动都是一模一样的，一样的东西在两个类里都写一遍，未免会觉得代码臃肿而且没有组织性。


#### 1.2.通过class的继承来抽出公共部分

我们可以使用面向对象的继承思想来组织代码，把公用的东西抽出来。

所以我们要做的就是造一个父类`Animal`（虽然把`Human`归进`Animal`有点怪怪的），把`age`、`setAge`和`getAge`这套公用的逻辑塞进父类`Animal`里，然后`Human`类和`Cat`类都继承于父类`Animal`，如下。

```ts
class Animal {
    // 把age、setAge和getAge逻辑抽出来，塞进父类Animal
    age
    setAge(age) {
        this.age = age
    }
    getAge() {
        console.log(`hi, I am ${this.age} years old now!!!\n`)
    }
}

class Human extends Animal {
    constructor(name){
        super()
        this.name = name
    }

    name;
    said() {
        console.log(`hi, my name is ${this.name}!!!`)
    }
}

class Cat extends Animal {
    meow() {
        console.log(`meow!!!!`)
    }
}


let jack = new Human("jack")
jack.said() // hi, my name is jack!!!
jack.setAge(22)
jack.getAge() // I am 22 years old now!!!

let tom = new Cat()
tom.meow() // meow!!!!
tom.setAge(3)
tom.getAge() // I am 3 years old now!!!
```

事实上，很多业务里面都是采用上面的写法的，这也是目前大多数场景解决方案。

#### 1.3.使用装饰器`Decorator`来注入公共部分

好了，上面铺垫了这么多，现在，就给你看看用装饰器`Decorator`是怎么实现上面的需求的（看就行了，不要求你去理解）

```ts
let addAge = target => {
    Object.assign(target.prototype, {
        age: 0,
        setAge: function(age){
            this.age = age
        },
        getAge: function(){
            console.log(`I am ${this.age} years old now!!!\n`)
        }
    })
}
    
@addAge
class Human {
    constructor(name){
        this.name = name
    }
    
    name;
    said() {
        console.log(`hi, my name is ${this.name}!!!`)
    }
}

@addAge
class Cat {
    meow() {
        console.log(`meow!!!!`)
    }
}


let jack: any = new Human("jack")
jack.said() // hi, my name is jack!!!
jack.setAge(22)
jack.getAge() // I am 22 years old now!!!

let tom: any = new Cat()
tom.meow() // meow!!!!
tom.setAge(3)
tom.getAge() // I am 3 years old now!!!
```

可能你会想这个`@addAge`是个什么鬼语法，addAge函数里面的`target`什么什么玩意，先别急，这个后面会慢慢说，只是现在你观察一下，这个方案和之前的两个方案有什么不一样？？？？

你会发现，第3个方案**并没有去改`Human`和`Cat`类的代码**。

为了满足往`Human`和`Cat`类加上`age`、`setAge`和`getAge`这三个玩意，第1个方案直接往`Human`和`Cat`里面加了那3个玩意，第2个方案则是强行让`Human`和`Cat`去extends Animal来认爹。

而第3个方案并没有去改`Human`和`Cat`的代码，而是直接往他们的头上扣一个`@addAge`,这样生成的实例就已经有那3个玩意了（所以装饰器的名字就是这么来的）。

所以你会发现，这是一种`无侵入性`、`可插拔性高`的抽取公共逻辑的方案。

当然了，`Decorator`的功能还很多，下面就会慢慢说。


# <span style="color:#F90">2.接下来就开始嗑定义啊、用法、语法什么的 </span>
待续。。。



