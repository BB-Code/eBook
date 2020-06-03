# **原生**javascript 安全的类型检测

```javascript
Object.prototype.toString.call([1, 2, 5]);
("[object Array]");
Object.prototype.toString.call(function test() {});
("[object Function]");
Object.prototype.toString.call(/.?123/);
("[object RegExp]");
Object.prototype.toString.call(JSON);
("[object JSON]");
```

# 作用域安全的构造函数

```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
}
undefined
var p = new Person('bobocode',23)
undefined
p instanceof Person
true
//忽略了new 操作符， this 晚绑定造成的
var p2 =  Person('fangfang',22)
p2.name
Cannot read property 'name' of undefined
window.name
"fangfang"

//作用域安全的构造函数
function Person(name,age){
    if(this instanceof Person){
        this.name = name;
        this.age = age;
    }else{
        return new Person(name,age);
    }
}
var p3 = new Person('p3',23);
var p4 =  Person('p4',23);
p3.name
"p3"
p4.name
"p4"
```

# 构造函数 call 的问题

## ![扩展](扩展知识.md)

```javascript
function Animal(name){
    if(this instanceof Animal){
        this.name = name;
        this.cry = function(){
            return  this.name + "cry";
        }
    }else{
        return new Animal("tiger");
    }
}

function Eat(eat){
    Animal.call(this,'eat')
    this.eat = eat;
    this.behavior = function(){
        return this.eat;
    }
}
undefined
var e = new Eat('tiger')
undefined
e.behavior()
"tiger"
e.name
undefined

解决：
Eat.prototype = new Animal();
var e = new Eat('tiger')
e.name
"tiger"
```

# 惰性载入函数

```javascript
// 问题  假如浏览器支持XHR,if执行了没必要的判断
function createXHR() {
  if (typeof XMLHttpRequest !== "undefined") {
    return new XMLHttpRequest();
  } else if (typeof ActiveXObject !== "undefined") {
    if (typeof arguments.callee.activeXString != "string") {
      var versions = [
          "MSXML2.XMLHttp.6.0",
          "MSXML2.XMLHttp.3.0",
          "MSXML2.XMLHttp",
        ],
        i,
        len;
      for (i = 0, len = versions.length; i < len; i++) {
        try {
          new ActiveXObject(versions[i]);
          arguments.callee.activeXString = versions[i];
        } catch (ex) {
          //
        }
      }
      return new ActiveXObject(arguments.callee.activeXString);
    } else {
      throw new Error("No XHR object available");
    }
  }
}

//惰性载入函数 分支只执行一次 效率提高 执行时会损失一点性能
//第一种  函数式  | 执行时损失一点性能
function createXHR() {
  if (typeof XMLHttpRequest !== "undefined") {
    createXHR = function () {
      return new XMLHttpRequest();
    };
  } else if (typeof ActiveXObjct !== "undefined") {
    createXHR = function () {
      if (typeof arguments.callee.activeXString != "string") {
        var versions = [
            "MSXML2.XMLHttp.6.0",
            "MSXML2.XMLHttp.3.0",
            "MSXML2.XMLHttp",
          ],
          i,
          len;
        for (i = 0, len = versions.length; i < len; i++) {
          try {
            new ActiveXObject(versions[i]);
            arguments.callee.activeXString = versions[i];
          } catch (ex) {
            //
          }
        }
        return new ActiveXObject(arguments.callee.activeXString);
      } else {
        throw new Error("No XHR object available");
      }
    };
  }else{
        createXHR = function () {
            throw new Error("No XHR object available");
        }
  }
  return createXHR();
}
//第二种  声明式  | 加载时损失一点性能
var  createXHR= (function() {
  if (typeof XMLHttpRequest !== "undefined") {
    return function () {
      return new XMLHttpRequest();
    };
  } else if (typeof ActiveXObjct !== "undefined") {
     return function () {
      if (typeof arguments.callee.activeXString != "string") {
        var versions = [
            "MSXML2.XMLHttp.6.0",
            "MSXML2.XMLHttp.3.0",
            "MSXML2.XMLHttp",
          ],
          i,
          len;
        for (i = 0, len = versions.length; i < len; i++) {
          try {
            new ActiveXObject(versions[i]);
            arguments.callee.activeXString = versions[i];
          } catch (ex) {
            //
          }
        }
        return new ActiveXObject(arguments.callee.activeXString);
      } else {
        throw new Error("No XHR object available");
      }
    };
  }else{
        return function () {
            throw new Error("No XHR object available");
        }
  }
})();
```
