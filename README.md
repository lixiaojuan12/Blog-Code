# Blog-Code
手写JS代码
## 一、前言
大家好！我是你们的开心小姐姐，愿你在阅读这篇文章吸收知识的同时是开心愉悦的☺️

最近也是看了不少的掘金文章，利用这些资源结合自己平时的代码总结，梳理成为了今天的这篇文章。这里面有一些面试题目和工作中经常用到的一些代码片段，希望对屏幕前的您有帮助😉



同时这写本篇的代码时也做了详细的注释，方便大家阅读与理解，如果仍有不理解的可以在评论区讨论一下。

开始让我们的遨游在代码的世界里吧~~~
## 二、代码篇

### 1. URL处理
> 实现效果：将地址栏中的URL中问号后面的内容变为对象的形式：http://www.baidu.com/search?name=li&age=18#student  
=>  
{HASH: "student", name: "li", age: "18"} 
#### 1.1 使用数组中的方法实现
```javascript
let str = 'http://www.baidu.com/search?name=li&age=18#student';
function urlQuery(str) {
    // 获取？和#的索引
    let askIndex = str.indexOf('?'),
        polIndex = str.indexOf('#'),
        askText = '',
        polText = '';
    //存储最后我们想要的结果    
    let obj = {};
    // 把# ?后面的内容给polText 和 askText
    polIndex != -1 ? polText = str.substring(polIndex + 1) : null;
    askIndex != -1 ? askText = str.substring(askIndex + 1, polIndex) : null;
    //存储到对象中
    polText ? obj['HASH'] = polText : null;
    if (askText) {
         askText.split('&').forEach((item, index) => {
             item = item.split('=');
             obj[item[0]]=item[1];
         });
    }
    return obj;
}
console.log(urlQuery(str));  //=> {HASH: "student", name: "li", age: "18"} 
```
#### 1.2 正则的实现
如果对下面案例中的正则不熟悉，可以查看这个正则篇来学习一下：[正则学习](https://juejin.im/post/5e8efc266fb9a03c550fdcb9#heading-25)
```javascript
let str = 'http://www.baidu.com/search?name=li&age=18#student';
function urlQuery(str) {
    let obj = {};
    //正则匹配=两边的
    str.replace(/([^=?&#]+)=([^=?&#]+)/g,(_,group1,group2)=>{
        obj[group1]=group2;
    });
    //正则匹配#后边的
    str.replace(/#([^=?&#]+)/g,(_,group1)=>{
        obj['HASH']=group1;
    });            
    return obj;
}
console.log(urlQuery(str));  //=> {name: "li", age: "18", HASH: "student"}
```
### 2. 数组去重
> 实现效果：把数组中的重复项去掉。例如：
[1,2,3,4,2,1] => [1,2,3,4]
#### 2.1 时间复杂度为O(n^2)的实现
```javascript
let ary = [1, 2, 2, 3, 4, 2, 1, 3, 1, 3];
function unique(ary) {
    // 双重for循环拿当前项和后面的每一项进行对比
    for (let i = 0; i < ary.length; i++) {
        let item = ary[i];
        for (let j = i + 1; j < ary.length; j++) {
            if (item == ary[j]) {
                //如果当前项和后面的这一项相等，那么末尾一项赋值给后面的这一项，并且长度减一，
                ary[j] = ary[ary.length - 1];
                ary.length--;
                j--;
            }
        }
    }
    return ary;
}
console.log(unique(ary));
```
#### 2.2 时间复杂度为O(n)的实现
```javascript
let ary = [1, 2, 2, 3, 4, 2, 1, 3, 1, 3];
function unique(ary) {
    let obj = {};
    for (let i = 0; i < ary.length; i++) {
        let item = ary[i];
        if (obj[item] !== undefined) {
            ary[i] = ary[ary.length - 1];
            ary.length--;
            //解决数组塌陷
            i--; 
        }
        obj[item] = item;
   }
   return ary;
}
console.log(unique(ary));
```
#### 2.3 使用ES6的Set实现
Set 是ES6中一种没有重复项的数据结构：[阮一峰老师ES6语法的讲解](https://es6.ruanyifeng.com/#docs/set-map)
```javascript
let ary = [1, 2, 2, 3, 4, 2, 1, 3, 1, 3];
ary = Array.from(new Set(ary));
console.log(ary);
```
### 3. 内置new的实现
```javascript
function Fn(x,y){
    this.x=x;
    this.y=y;
}
function _new(func){
    let params=Array.from(arguments).slice(1);
    let obj=Object.create(func.prototype);
    let result=func.apply(obj,params);
    //如果类中有return并且是引入数据类型，那么返回类中的return
    if(result!==null&&(typeof result==='object'||typeof result==='function')){
         return result;
    }
    return obj;
}
let f1=_new(Fn,10,20);
```
### 4. 什么情况下输出'ok'
```javascript
var a = ?;
if (a == 1 && a == 2 && a == 3) {
    console.log('ok');
}
```
```javascript
//方案一 
var a = {
  n:0,
  toString:function(){
    return ++this.n;
  }
}

//方案二
var a = [1,2,3];
a.toString = a.shift;

//方案三
var i = 0;
Object.defineProperty(window,'a',{
  get(){
    //只有获取a的值，就一定会触发get方法执行
    return ++i;
  }
})
```
### 5. 内置call
```javascript
Function.prototype.changeThis = function changeThis(context,...args){
  if(context == null){
    //==：null和undefined在两个等号的时候相等，因为if后面，只要传递进来是undefined或者null都可以
    context = window;
  }
  if(typeof context !== 'object' && typeof context !== 'function'){
    context = new context.constructor(context);
  }
  let uniqueKey = `$$${new Date().getTime()}`;
  context[uniqueKey] = this;
  let result = context[uniqueKey](...args);
  delete context[uniqueKey];
  return result;
}
```
### 6. 内置apply
```javascript
Function.prototype.myApply = function (context) {
    // 如果没有传或传的值为空对象 context指向window
    if (context == null) {
        context = window
    }
    let fn = mySymbol(context)
    context[fn] = this //给context添加一个方法 指向this
    // 处理参数 去除第一个参数this 其它传入fn函数
    let arg = [...arguments].slice(1) //[...xxx]把类数组变成数组，arguments为啥不是数组自行搜索 slice返回一个新数组
    context[fn](arg) //执行fn
    delete context[fn] //删除方法
}

```
### 7. 内置bind
```javascript
Function.prototype.bind = function (context) {
    //返回一个绑定this的函数，我们需要在此保存this
    let self = this
    // 可以支持柯里化传参，保存参数
    let arg = [...arguments].slice(1)
    // 返回一个函数
    return function () {
        //同样因为支持柯里化形式传参我们需要再次获取存储参数
        let newArg = [...arguments]
        console.log(newArg)
        // 返回函数绑定this，传入两次保存的参数
        //考虑返回函数有返回值做了return
        return self.apply(context, arg.concat(newArg))
    }
}

```
### 8. 防抖
> 函数的防抖debounce：不是某个事件触发就去执行函数，而是在指定的时间间隔内，执行一次，减少函数执行的次数。（在一定时间只能只能执行一次）
```javascript
/*
 * @Parma
 *   func要执行的函数 
 *   wait间隔等待的时间 
 *   immediate在开始边界还是结束边界触发执行（true在开始边界）
 * @return
 *   可被调用的函数
 * by 不要情绪 on 2020/05/24
*/
function debounce(func, wait, immediate) {
     let result = null,
         timeout = null;
    return function (...args) {
        let context = this,
            now = immediate && !timeout;
       clearTimeout(timeout); //重点：在设置新的定时器之前，要把之前设置的定时器都清除，因为防抖的目的是等待时间内，只执行一次
       timeout = setTimeout(() => {
            if (!immediate) result = func.call(context, ...args);
            clearTimeout(timeout);
            timeout = null;
      }, wait);
      if (now) result = func.call(context, ...args);
   }
}
```
### 9. 节流
> 函数的节流（throttle）：为了缩减执行的频率，但不像防抖一样，一定时间内只能执行一次，而是一定时间内能执行多次
```javascript
/*
 * throttle：函数节流是为了缩减执行频率，当达到了一定的时间间隔就会执行一次
 *   @params
 *      func:需要执行的函数
 *      wait:设置的间隔时间
 *   @return
 *      返回可被调用的函数
 * by 不要情绪 on 2019/08/20
 */
let throttle = function (func, wait) {
   let timeout = null,
       result = null,
       previous = 0; //=>上次执行时间点
   return function (...args) {
       let now = new Date,
           context = this;
      //=>remaining小于等于0，表示上次执行至此所间隔时间已经超过一个时间间隔
      let remaining = wait - (now - previous);
      if (remaining <= 0) {
           clearTimeout(timeout);
           previous = now;
           timeout = null;
           result = func.apply(context, args);
     } else if (!timeout) {
           timeout = setTimeout(() => {
                previous = new Date;
                timeout = null;
                result = func.apply(context, args);
           }, remaining);
     }
 return result;
  };
};

```
### 10. 深克隆
> 不仅把第一级克隆一份给新的数组，如果原始数组中存在多级，那么是把每一级都克隆一份赋值给新数组的每一个级别
```javascript
function cloneDeep(obj) {
  // 传递进来的如果不是对象，则无需处理，直接返回原始的值即可（一般Symbol和Function也不会进行处理的）
  if (obj === null) return null;
  if (typeof obj !== "object") return obj;

  // 过滤掉特殊的对象（正则对象或者日期对象）：直接使用原始值创建当前类的一个新的实例即可，这样克隆后的是新的实例，但是值和之前一样
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Date) return new Date(obj);

  // 如果传递的是数组或者对象，我们需要创建一个新的数组或者对象，用来存储原始的数据
  // obj.constructor 获取当前值的构造器（Array/Object）
  let cloneObj = new obj.constructor;
  for (let key in obj) {
    // 循环原始数据中的每一项，把每一项赋值给新的对象
    if (!obj.hasOwnProperty(key)) break;
    cloneObj[key] = cloneDeep(obj[key]);
  }
  return cloneObj;
}

```

![image](https://imgkr.cn-bj.ufileos.com/52c32ce6-0d07-426f-84a7-9b0bfcb85972.png)

### 11. 深比较
```javascript
function _assignDeep(obj1, obj2) {
  // 先把OBJ1中的每一项深度克隆一份赋值给新的对象
  let obj = _cloneDeep(obj1);

  // 再拿OBJ2替换OBJ中的每一项
  for (let key in obj2) {
    if (!obj2.hasOwnProperty(key)) break;
    let v2 = obj2[key],
      v1 = obj[key];
    // 如果OBJ2遍历的当前项是个对象，并且对应的OBJ这项也是一个对象，此时不能直接替换，需要把两个对象重新合并一下，合并后的最新结果赋值给新对象中的这一项
    if (typeof v1 === "object" && typeof v2 === "object") {
      obj[key] = _assignDeep(v1, v2);
      continue;
    }
    obj[key] = v2;
  }

  return obj;
}
```
### 12. 验证手机号
> 规则：  
11 位  
第一位是数字 1  
第二位是数字 3-9 中的任意一位
```javascript
let reg = /^1[3-9]\d{9}$/;
console.log(reg.test('13245678945')); //true
console.log(reg.test('1324567895'));  //false
console.log(reg.test('12245678945'));  //false
```
### 13. 验证是否是有效数字
> 规则：  
开头可以有+ -  
整数位：  
如果是一位数可以是 0-9 任意数；  
如果是多位数，首位不可以是 0；  
小数位：如果有小数位，那么小数位后面至少有一位数字，也可以没有小数位  
```javascript
let reg = /^[+-]?(\d|[1-9]\d+)(\.\d+)?$/;
console.log(reg.test('0.2')); //true
console.log(reg.test('02.1'));  //false
console.log(reg.test('20.'));  //false
```
### 14. 验证密码
> 规则：   
6-16 位组成   
必须由数字字母组成
```javascript
let reg = /^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[\d(a-z)(A-Z)]{6,16}$/;
```
### 15. 验证真实姓名
> 规则：  
必须是汉字  
名字长度 2-10 位  
可能有译名：·汉字  
```javascript
let reg = /^[\u4E00-\u9FA5]{2,10}(·[\u4E00-\u9FA5]{2,10})?$/;
```
### 16. 验证邮箱
> 规则：
邮箱的名字以‘数字字母下划线-.’几部分组成，但是-/.不能连续出现也不能作为开头 \w+((-\w+)|(.\w+))*;  
 @ 后面可以加数字字母，可以出现多位 @[A-Za-z0-9]+ ;   
 对@后面名字的补充：多域名 .com.cn ;企业域名 (.|-)[A-Za-z0-9]+)*  
 .com/.cn 等域名 .[A-Za-z0-9]+
```javascript
let reg = /^\w+((-\w+)|(\.\w+))*@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/

```
### 17. 验证身份证号
```javascript
let reg =/^([1-9]\d{5})((19|20)\d{2})(0[1-9]|10|11|12)(0[1-9]|[1-2]\d|30|31)\d{3}(\d|x)$/i;
```
### 18. 时间格式字符串
> 月日不足十位补零  
换成年月日的格式
```javascript
let time = '2020-5-27';
String.prototype.formatTime = function formatTime(template) {
    let arr = this.match(/\d+/g).map(item => {
        return item.length < 2 ? '0' + item : item;
    });
    template = template || '{0}年{1}月{2}日 {3}时{4}分{5}秒';
    return template.replace(/\{(\d+)\}/g, (_, group) => {
        return arr[group] || "00";
    });
};
console.log(time.formatTime()); //=> 2020年05月27日 00时00分00秒
```
### 19. 字符串中出现次数最多的字符以及次数
```javascript
let str = "hello";
let ary = [...new Set(str.split(''))];
let max = 0;
let code = ''; for (let i = 0; i < ary.length; i++) {
    //创建正则匹配字符 
    let reg = new RegExp(ary[i], 'g');
    //利用match找出对应字符在中字符串中出现的地方，取匹配的返回数组的长度，即是对应字符串出现的次数 
    let val = str.match(reg).length; //更新出现次数最高的字符与次数 
    if (val > max) {
        max = val;
        code = ary[i];
    } else if (val === max) {
        //处理不同字符出现次数相同的情况  
        code = `${code}、${ary[i]}`;
    }
}
console.log(`出现次数最多的字符是：${code},次数为：${max}`);  //=> 出现次数最多的字符是：l,次数为：2

```
### 20. 千分符
```javascript
String.prototype.millimeter = function millimeter() {
    return this.replace(/\d{1,3}(?=(\d{3})+$)/g, value => {
        return value + ',';
    });
};
let str = "2312345638";
str = str.millimeter();
console.log(str); //=>"2,312,345,638"
```
### 21. 继承（一）原型继承
```javascript
function Parent() {
    this.x = 100;
}
Parent.prototype.getX = function getX() {
    return this.x;
};

function Child() {
    this.y = 200;
}
Child.prototype = new Parent; //=>原型继承
Child.prototype.getY = function getY() {
    return this.y;
};
let c1 = new Child;
console.log(c1);
```
### 22. 继承（二）call继承
```javascript
function Parent() {
    this.x = 100;
}
Parent.prototype.getX = function getX() {
    return this.x;
};
function Child() {
    // 在子类构造函数中，把父类当做普通方法执行（没有父类实例，父类原型上的那些东西也就和它没关系了）
    // this -> Child的实例c1
    Parent.call(this); // this.x=100 相当于强制给c1这个实例设置一个私有的属性x，属性值100，相当于让子类的实例继承了父类的私有的属性，并且也变为了子类私有的属性 “拷贝式”
    this.y = 200;
}
Child.prototype.getY = function getY() {
    return this.y;
};
let c1 = new Child;
console.log(c1);

```
### 23. 继承（三）组合式继承
```javascript
function Parent() {
    this.x = 100;
}
Parent.prototype.getX = function getX() {
    return this.x;
};
function Child() {
    Parent.call(this);//call
    this.y = 200;
}
Child.prototype = Object.create(Parent.prototype);//原型继承
Child.prototype.constructor = Child;
Child.prototype.getY = function getY() {
    return this.y;
};

let c1 = new Child;
console.log(c1);

```
### 24. 数组扁平化
> [1, 2, [3], [4, 5, [6, [7, 8, 9]]]] => [1,2,3,4,5,6,7,8,9,]
```javascript
var arr = [1, 2, [3], [4, 5, [6, [7, 8, 9]]]];
function flatten(arr) {
    return arr.reduce((res, next) => {
        return res.concat(Array.isArray(next) ? flatten(next) : next);
    }, []);
}
console.log(flatten(arr));
```
### 25. 改造代码，输出0-9
> 闭包的解决方案
```javascript
for (var i = 0; i< 10; i++){
	setTimeout(() => {
		console.log(i);
    }, 1000)
}
```
```javascript
for (let i = 0; i< 10; i++){
	setTimeout(() => {
		console.log(i);
    }, 1000)
}
```
### 26. 冒泡排序
```javascript
var ary = [3, 1, 5, 2];
function bubble(ary) {
    // 比的轮数,最后一轮不用，前面几轮比完之后，最后那个肯定就是最小的
    for (var i = 0; i < ary.length - 1; i++) {
        //两两进行比较
        for (var j = 0; j < ary.length - 1 - i; j++) {
            if (ary[j] > ary[j + 1]) {
                //解构赋值
                [ary[j], ary[j + 1]] = [ary[j + 1], ary[j]]
            }
        }
    }
    return ary;
}
var res = bubble(ary);
console.log(res);
```
### 27. 快速排序
```javascript
function quickSort(ary){
    if(ary.length<1){
        return ary;
    }
   var centerIndex=Math.floor(ary.length/2);
   // 拿到中间项的同时，把中间项从数组中删除掉
   var centerValue=ary.splice(centerIndex,1)[0];
   // 新建两个数组：leftAry,rightAry;把ary中剩余的项，给中间项做对比，如果大项就放到右数组，小项就放到左数组.
   var leftAry=[],rightAry=[];
   for(var i=0;i<ary.length;i++){
        if(ary[i]<centerValue){
            leftAry.push(ary[i]);
        }else{
            rightAry.push(ary[i]);
        }
    }
    return quickSort(leftAry).concat(centerValue,quickSort(rightAry));
}
var ary=[12,15,14,13,16,11];
var res=quickSort(ary);
console.log(res);
```
### 28. 插入排序
```javascript
var ary = [34, 56, 12, 66, 12];
function insertSort(ary) {
   //最终排序好的数组盒子
   var newAry = [];
   //拿出的第一项放进去，此时盒子中只有一项，不用个比较
   newAry.push(ary[0]);
   // 依次拿出原数组中的每一项进行插入
   for (var i = 1; i < ary.length; i++) {
      var getItem = ary[i];
      // 在插入的时候需要跟新数组中的每一项进行比较（从右向左）
      for (var j = newAry.length - 1; j >= 0; j--) {
         var newItemAry = newAry[j];
         if (getItem >= newItemAry) {
            // 如果拿出的项比某项大或者相等，就放到此项的后面
            newAry.splice(j + 1, 0, getItem);
            // 插入完毕，不用再继续比较停止循环；
            break;
         }
         if (j == 0) {
            //如果都已经比到第一项了，还没满足条件，说明这个就是最小项，我们之间插入到数组的最前面
            newAry.unshift(getItem);
         }
      }

   }
   return newAry;
}
var res = insertSort(ary);
console.log(res);
```
### 29. 倒计时案例
```javascript

let box = document.querySelector('#box'),
    content = document.querySelector('#content'),
    timer = null;
/*
 * 每次页面刷新，必须从服务器拿到时间，才可以进行下面的事情
 *    基于ajax异步去请求，请求成功才做
 *    所以需要放在回调函数里面
 *    为了防止回调地狱问题，可以使用promise
 */

//获取服务器的时间
function getServerTime() {
    return new Promise(resolve => {
        let xhr = new XMLHttpRequest;
        xhr.open('get', './data.json?_=' + Math.random());
        xhr.onreadystatechange = () => {
            if (xhr.readyState === 4 && /^(2|3)\d{2}$/.test(xhr.status)) {
                let time = xhr.getResponseHeader('date');
                time = new Date(time);
                resolve(time);
            }
        }
        xhr.send(null);
    });
}

//根据服务器时间计算倒计时
function computed(time) {
    let target = new Date('2020/05/13 18:59:00'),
        spanTime = target - time;
    if (spanTime <= 0) {
        //已经到底抢购的时间点了
        box.innerHTML = '开始抢购吧！'
        clearInterval(timer);
        return;
    }
    //计算出毫秒差中包含多少小时、多少分钟、多少秒
    let hour = Math.floor(spanTime / (60 * 60 * 1000));
    spanTime = spanTime - hour * 60 * 60 * 1000;
    let minutes = Math.floor(spanTime / (60 * 1000));
    spanTime = spanTime - minutes * 60 * 1000;
    let seconds = Math.floor(spanTime / 1000);
    hour < 10 ? hour = '0' + hour : null;
    minutes < 10 ? minutes = '0' + minutes : null;
    seconds < 10 ? seconds = '0' + seconds : null;
    content.innerHTML = `${hour}:${minutes}:${seconds}`;
}

getServerTime().then(time => {
    //获取到服务器的时间后
    computed(time);
    //每隔1秒，累加1
    timer = setInterval(() => {
        time = new Date(time.getTime() + 1000);
        computed(time);
    }, 1000);
});

```
### 30. 随机验证码
```javascript
//随机验证码：数字和字母组合，（四个数字和字母）
/* 
=> 1先把验证码准备好
=> 2随机获取相应的索引值
=> 3获取元素
*/
var code = document.getElementById("code");
var btn = document.getElementById("btn");
code.innerHTML = getCode();
btn.onclick = function () {
    code.innerHTML = getCode();
}
function getCode() {
    var str = "qwertyuiopasdfghjklzxcvbnm" + "QWERTYUIOPASDFGHJKLZXCVBNM" + "0123456789";
    var result = "";
    //==> 索引值的范围：0---61
    while (result.length < 4) {
        var index = Math.round(Math.random() * 61);
        var item = str[index];
        if (result.indexOf(item) == -1) {
            result += item;
        }
    }
    return result;
}
```
### 31. 选项卡实现
```html
<body>
    <div id="app">
        <ul class='list'>
            <li class='active'>css</li>
            <li>js</li>
            <li>html</li>
        </ul>
        <ul class='con'>
            <li class='active'>css样式</li>
            <li>js行为</li>
            <li>HTML结构</li>
        </ul>
    </div>
</body>
```
```css
 * {
    margin: 0;
    padding: 0;
    list-style: none;
  }

 #app {
      width: 500px;
      margin: 20px auto;
 }
 .list {
      overflow: hidden;
      position: relative;
      top: 1px;
 }
 .list li {
      width: 100px;
      float: left;
      border: 1px solid #333;
      text-align: center;
      margin-right: 10px;
      height: 50px;
      line-height: 50px;
      cursor: pointer;
 }
.list li.active {
      background-color: crimson;
      border-bottom-color: crimson;
 }
.con li {
    width: 100%;
    border: 1px solid #333;
    height: 300px;
    line-height: 300px;
    text-align: center;
    background-color: crimson;
    display: none;
 }
 .con li.active {
     display: block;
 }
```
```javascript
let list = document.querySelector('.list'),
    lists = list.querySelectorAll('li'),
    con = document.querySelector('.con'),
    cons = con.querySelectorAll('li');
for (let i = 0; i < lists.length; i++) {
    lists[i].onclick = function () {
        click(i);
    };
}
function click(index) {
    for (let i = 0; i < lists.length; i++) {
        lists[i].className = '';
        cons[i].className = '';
    }
    lists[index].className = 'active';
    cons[index].className = 'active';
}
```
![选项卡](https://imgkr.cn-bj.ufileos.com/97c2e631-f192-495d-b1e4-f0d77be6b9e2.gif)
## 三、总结
以上就是自己通过看其他大佬们的文章再加上自己的知识总结的常见代码片段。整理不易，如果对您有帮助可以给小姐姐一个小心心。   
后期也会把自己的学习总结发布出来和大家一起进步成长✌️
