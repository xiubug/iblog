---
title: 设计模式
date: 2018-04-19 20:23:16
tags: 
  - note
categories:
  - design
---

设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。设计模式是一种思想，常见的生活例子如：盖房子的设计图纸，古代战争的孙子兵法。设计模式（Design pattern）代表了最佳的实践，通常被有经验的面向对象的软件开发人员所采用。设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

**设计模式的目的：**
> 设计模式是优秀的使用案例，使用设计模式可提高代码的重用性、让代码更容易被他人理解、保证代码可靠性。

java基本模式有23种：单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式....，下面分别介绍一下设计模式：

## 单例模式Singleton

保证整个应用中某个实例有且只有一个。常见的生活例子有：古代皇帝有且只有一个、中国的一妻一夫制... 开发中，有些对象我们只需要一个，比如：配置文件、工具类、线程池、缓存、日志对象等，如果创造出多个实例，就会导致许多问题，比如占用过多资源，不一致的结果等。

**应用场合：**有些对象只需要一个就足够了，如古代皇帝、老婆
**作用：**保证整个应用程序中某个实例有且只有一个
**类型：**饿汉模式、懒汉模式

### 单例模式的饿汉式实现

单例类：
```javascript
package com.sosout;
/**
* 饿汉模式
*/
public class Singleton {
  // 1、将构造方法私有化，不允许外部直接创建对象
  private Singleton() {
  }

  // 2、创建类的唯一实例，使用private static修饰
  private static Singleton instance = new Singleton();

  // 3、提供一个用于获取实例的方法，使用public static 修饰
  public static Singleton getInstance() {
    return instance;
  }
}
```

测试类：
```javascript
package com.sosout;
public class Test {
  public static void main(String[] args) {
    Singleton s1 = Singleton.getInstance();
    Singleton s2 = Singleton.getInstance();
    if (s1 == s2) {
      System.out.println("s1和s2是同一个实例");
    } else {
      System.out.println("s1和s2不是同一个实例");
    }
  }
}
```

### 单例模式的懒汉式实现

单例类：
```javascript
package com.sosout;
/**
* 懒汉模式
*/
public class Singleton2 {
  // 1、将构造方法私有化，不允许外部直接创建对象
  private Singleton2() {
  }

  // 2、创建类的唯一实例，使用private static修饰
  private static Singleton2 instance;

  // 3、提供一个用于获取实例的方法，使用public static 修饰
  public static Singleton2 getInstance() {
    if (instance == null) {
      instance = new Singleton2();
    }
    return instance;
  }
}
```

测试类：
```javascript
package com.sosout;
public class Test {
  public static void main(String[] args) {
    // 饿汉模式
    Singleton s1 = Singleton.getInstance();
    Singleton s2 = Singleton.getInstance();
    if (s1 == s2) {
      System.out.println("s1和s2是同一个实例");
    } else {
      System.out.println("s1和s2不是同一个实例");
    }

    // 懒汉模式
    Singleton2 s3 = Singleton2.getInstance();
    Singleton2 s4 = Singleton2.getInstance();
    if (s3 == s4) {
      System.out.println("s3和s4是同一个实例");
    } else {
      System.out.println("s3和s4不是同一个实例");
    }
  }
}
```

### 饿汉模式和懒汉模式区别

饿汉模式的特点是加载类时比较慢，但运行时获取对象的速度比较快，线程安全的，懒汉模式的特点是加载类时比较快，但运行时获取对象的速度比较慢，线程不安全的。

**在JavaScript里，单例作为一个命名空间提供者，从全局命名空间里提供一个唯一的访问点来访问该对象。**

**在JavaScript里，实现单例的方式有很多种，其中最简单的一个方式是使用对象字面量的方法，其字面量里可以包含大量的属性和方法：**
```javascript
var mySingleton = {
  property1: "something",
  property2: "something else",
  method1: function () {
    console.log('hello world');
  }
};
```

**如果以后要扩展该对象，你可以添加自己的私有成员和方法，然后使用闭包在其内部封装这些变量和函数声明。只暴露你想暴露的public成员和方法，样例代码如下：**
```javascript
var mySingleton = function () {

  /* 这里声明私有变量和方法 */
  var privateVariable = 'something private';
  function showPrivate() {
    console.log(privateVariable);
  }

  /* 公有变量和方法（可以访问私有变量和方法） */
  return {
    publicMethod: function () {
      showPrivate();
    },
    publicVar: 'the public can see this!'
  };
};

var single = mySingleton();
single.publicMethod();  // 输出 'something private'
console.log(single.publicVar); // 输出 'the public can see this!'
```
**上面的代码很不错了，但如果我们想做到只有在使用的时候才初始化，那该如何做呢？为了节约资源的目的，我们可以另外一个构造函数里来初始化这些代码，如下：**
```javascript
var Singleton = (function () {
    var instantiated;
    function init() {
        /*这里定义单例代码*/
        return {
            publicMethod: function () {
                console.log('hello world');
            },
            publicProperty: 'test'
        };
    }

    return {
        getInstance: function () {
            if (!instantiated) {
                instantiated = init();
            }
            return instantiated;
        }
    };
})();

/*调用公有的方法来获取实例:*/
Singleton.getInstance().publicMethod();
```

## 工厂模式

### 什么是工厂模式？

实例化对象，用工厂方法代替new操作。
工厂模式包括工厂方法模式和抽象工厂模式。
抽象工厂模式是工厂方法模式的扩展。

## 策略模式

### 定义
定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

### 优点
**1、**策略模式利用组合，委托等技术和思想，有效的避免很多if条件语句。
**2、**策略模式提供了开放-封闭原则，使代码更容易理解和扩展。
**3、**策略模式中的代码可以复用。

### 示例：使用策略模式计算奖金
比如公司的年终奖是根据员工的工资和绩效来考核的，绩效为A的人，年终奖为工资的4倍，绩效为B的人，年终奖为工资的3倍，绩效为C的人，年终奖为工资的2倍；现在我们使用一般的编码方式会如下这样编写代码：

#### 一般方式
``` js
var calculateBouns = function(salary,level) {
  if(level === 'A') {
    return salary * 4;
  }
  if(level === 'B') {
    return salary * 3;
  }
  if(level === 'C') {
    return salary * 2;
  }
};
// 调用如下：
console.log(calculateBouns(4000,'A')); // 16000
console.log(calculateBouns(2500,'B')); // 7500
```
第一个参数为薪资，第二个参数为等级；代码缺点如下：
**1、**calculateBouns 函数包含了很多if-else语句。
**2、**calculateBouns 函数缺乏弹性，假如还有D等级的话，那么我们需要在calculateBouns 函数内添加判断等级D的if语句；
**3、**算法复用性差，如果在其他的地方也有类似这样的算法的话，但是规则不一样，我们这些代码不能通用。

#### 组合函数
组合函数是把各种算法封装到一个个的小函数里面，比如等级A的话，封装一个小函数，等级为B的话，也封装一个小函数，以此类推；如下代码：
``` js
var performanceA = function(salary) {
  return salary * 4;
};
var performanceB = function(salary) {
  return salary * 3;
};
        
var performanceC = function(salary) {
  return salary * 2;
};
var calculateBouns = function(level,salary) {
  if(level === 'A') {
    return performanceA(salary);
  }
  if(level === 'B') {
    return performanceB(salary);
  }
  if(level === 'C') {
    return performanceC(salary);
  }
};
// 调用如下
console.log(calculateBouns('A',4500)); // 18000
```
代码看起来有点改善，但是还是有如下缺点：
calculateBouns 函数有可能会越来越大，比如增加D等级的时候，而且缺乏弹性。

#### 策略模式
##### 传统面向对象
策略模式指的是 定义一系列的算法，把它们一个个封装起来，将不变的部分和变化的部分隔开，实际就是将算法的使用和实现分离出来；算法的使用方式是不变的，都是根据某个算法取得计算后的奖金数，而算法的实现是根据绩效对应不同的绩效规则；
一个基于策略模式的程序至少由2部分组成，第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。第二个部分是环境类Context，该Context接收客户端的请求，随后把请求委托给某一个策略类。我们先使用传统面向对象来实现；
如下代码：
``` js
var performanceA = function(){};
performanceA.prototype.calculate = function(salary) {
    return salary * 4;
};
        
var performanceB = function(){};
performanceB.prototype.calculate = function(salary) {
    return salary * 3;
};

var performanceC = function(){};
performanceC.prototype.calculate = function(salary) {
    return salary * 2;
};
// 奖金类
var Bouns = function(){
    this.salary = null;    // 原始工资
    this.levelObj = null;  // 绩效等级对应的策略对象
};
Bouns.prototype.setSalary = function(salary) {
    this.salary = salary;  // 保存员工的原始工资
};
Bouns.prototype.setlevelObj = function(levelObj){
    this.levelObj = levelObj;  // 设置员工绩效等级对应的策略对象
};
// 取得奖金数
Bouns.prototype.getBouns = function(){
    // 把计算奖金的操作委托给对应的策略对象
    return this.levelObj.calculate(this.salary);
};
var bouns = new Bouns();
bouns.setSalary(10000);
bouns.setlevelObj(new performanceA()); // 设置策略对象
console.log(bouns.getBouns());  // 40000
        
bouns.setlevelObj(new performanceB()); // 设置策略对象
console.log(bouns.getBouns());  // 30000
```
如上代码使用策略模式重构代码，可以看到代码职责更新分明，代码变得更加清晰。

##### Javascript
代码如下：
``` js
var obj = {
  "A": function(salary) {
    return salary * 4;
  },
  "B": function(salary) {
    return salary * 3;
  },
  "C": function(salary) {
    return salary * 2;
  } 
};
var calculateBouns =function(level,salary) {
    return obj[level](salary);
};
console.log(calculateBouns('A',10000)); // 40000
```
可以看到代码更加简单明了；
策略模式指的是定义一系列的算法，并且把它们封装起来，但是策略模式不仅仅只封装算法，我们还可以对用来封装一系列的业务规则，只要这些业务规则目标一致，我们就可以使用策略模式来封装它们；

### 示例：使用策略模式进行表单校验
我们经常来进行表单验证，比如注册登录对话框，我们登录之前要进行验证操作：
**1、**用户名不能为空。
**2、**密码长度不能小于6位。
**3、**手机号码必须符合格式。
。。。

比如HTML代码如下：
```html
<form action = "http://www.baidu.com" id="registerForm" method = "post">
  <p>
      <label>请输入用户名：</label>
      <input type="text" name="userName"/>
  </p>
  <p>
      <label>请输入密码：</label>
      <input type="text" name="password"/>
  </p>
  <p>
      <label>请输入手机号码：</label>
      <input type="text" name="phoneNumber"/>
  </p>
</form>
```
#### 一般方式
我们正常的编写表单验证代码如下：
``` js
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    if(registerForm.userName.value === '') {
        alert('用户名不能为空');
        return;
    }
    if(registerForm.password.value.length < 6) {
        alert("密码的长度不能小于6位");
        return;
    }
    if(!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
        alert("手机号码格式不正确");
        return;
    }
}
```
但是这样编写代码有如下缺点：
**1、**registerForm.onsubmit 函数比较大，代码中包含了很多if语句；
**2、**registerForm.onsubmit 函数缺乏弹性，如果增加了一种新的效验规则，或者想把密码的长度效验从6改成8，我们必须改registerForm.onsubmit 函数内部的代码。违反了开放-封闭原则。
**3、**算法的复用性差，如果在程序中增加了另外一个表单，这个表单也需要进行一些类似的效验，那么我们可能又需要复制代码了；

#### 策略模式
第一步我们先来封装策略对象，如下代码：
``` js
var strategy = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    } 
};
```
接下来我们准备实现Validator类，Validator类在这里作为Context，负责接收用户的请求并委托给strategy 对象，如下代码：
``` js
var Validator = function(){
        this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6 
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};
```
Validator类在这里作为Context，负责接收用户的请求并委托给strategys对象。上面的代码中，我们先创建一个Validator对象，然后通过validator.add方法往validator对象中添加一些效验规则，validator.add方法接收3个参数，如下代码：
**validator.add(registerForm.password,'minLength:6','密码长度不能小于6位')**
**registerForm.password：**校验的input输入框dom节点；
**minLength:6：**是以一个冒号隔开的字符串，冒号前面的minLength代表客户挑选的strategys对象，冒号后面的数字6表示在效验过程中所必须验证的参数，minLength:6的意思是效验 registerForm.password 这个文本输入框的value最小长度为6位；如果字符串中不包含冒号，说明效验过程中不需要额外的效验信息；
**第三个参数：**当效验未通过时返回的错误信息；

当我们往validator对象里添加完一系列的效验规则之后，会调用validator.start()方法来启动效验。如果validator.start()返回了一个errorMsg字符串作为返回值，说明该次效验没有通过，此时需要registerForm.onsubmit方法返回false来阻止表单提交。下面我们来看看初始化代码如下：
``` js
var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```
下面是所有的代码如下：
``` js
var strategys = {
    isNotEmpty: function(value,errorMsg) {
        if(value === '') {
            return errorMsg;
        }
    },
    // 限制最小长度
    minLength: function(value,length,errorMsg) {
        if(value.length < length) {
            return errorMsg;
        }
    },
    // 手机号码格式
    mobileFormat: function(value,errorMsg) {
        if(!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
            return errorMsg;
        }
    } 
};
var Validator = function(){
    this.cache = [];  // 保存效验规则
};
Validator.prototype.add = function(dom,rule,errorMsg) {
    var str = rule.split(":");
    this.cache.push(function(){
        // str 返回的是 minLength:6 
        var strategy = str.shift();
        str.unshift(dom.value); // 把input的value添加进参数列表
        str.push(errorMsg);  // 把errorMsg添加进参数列表
        return strategys[strategy].apply(dom,str);
    });
};
Validator.prototype.start = function(){
    for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]; ) {
        var msg = validatorFunc(); // 开始效验 并取得效验后的返回信息
        if(msg) {
            return msg;
        }
    }
};

var validateFunc = function(){
    var validator = new Validator(); // 创建一个Validator对象
    /* 添加一些效验规则 */
    validator.add(registerForm.userName,'isNotEmpty','用户名不能为空');
    validator.add(registerForm.password,'minLength:6','密码长度不能小于6位');
    validator.add(registerForm.userName,'mobileFormat','手机号码格式不正确');

    var errorMsg = validator.start(); // 获得效验结果
    return errorMsg; // 返回效验结果
};
var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    var errorMsg = validateFunc();
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
};
```