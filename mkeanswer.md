### 一.简答题

#### 1、描述引用计数工作原理及优缺点。

答：引用计数核心思想是设置一个引用数，判断当前引用数是否为0。引用关系改变时修改引用数字，当引用数字为0时立即回收。

两个优点：

-  发现垃圾时，立即回收。
- 最大限度减少程序暂停，减少程序卡顿时间。

两个缺点：

- 无法回收循环引用的对象。例如，

  a = []  

  b = []

  a.append(b)  

  b.append(a)

  a与b相互引用，如果不存在其他对象对它们的引用，a与b的引用计数也仍然为1，所占用的内存永远无法被回收。

- 时间资源开销大。

#### 2、描述标记整理算法的工作流程。

答：**标记-整理（Mark-Compact）**算法不直接对可回收对象进行清理，而是让所有可用的对象都向一端移动。然后直接清理掉边界意外的内存。

![这里写图片描述](https://img-blog.csdn.net/20161101222342412)

很显然，整理这一下需要时间，所以与标记清除算法相比，这一步花费了不少时间，但从长远来看，这一步还是很有必要的。

​    标记整理算法可以看做是标记清除的增强，标记阶段的操作和标记清除一致，即遍历所有对象找到标记活动对象。跟标记清除区别之处在于，标记整理清除阶段会先执行整理，移动对象位置，最后遍历所有对象清除没有标记对象，回收相应空间。

​	标记整理算法可以减少碎片化空间，但不会立即回收垃圾对象。

#### 3、描述V8中新生代存储垃圾回收的流程

答：新生代的回收策略是通过scavenge中的Cheney算法实现的，这个算法主要是通过复制实现的。新生代内存主要分为两个区，From区（被使用的）和To区（闲置的），这个两个区主要就是进行来回复制。新生代的回收主要的实现步骤如下：

 （1）对象首先被分到From区。

 （2）检查对象是否是活跃对象，有的话直接复制到To区。

 （3）To区有两个指针：scanPtr和allocationPtr scanPtr主要是指向即将扫描的活跃对象 allocationPtr是为即将成为新对象分配内存。

新生代对象的晋升：

 （1）从From区到To区，检查对象的内存地址是否经历过新生代的清理。

 （2）对象从From区到To区，发现To区空间已经被使用超过25%，则对象将直接被分配到老生代。

#### 4、描述增量标记算法在何时使用及工作原理。

答：为了不阻塞js程序正常运行，提高垃圾回收的效率，将一整段垃圾回收分成多个小部分组合完成当前整个回收，从而替代之前的一次性完成的垃圾回收操作，可以实现垃圾回收与程序执行交替执行和完成。改变了以前程序执行时不能做垃圾回收，垃圾回收时程序不能执行的问题。从而时间消耗更合理。

程序刚开始执行时不需要垃圾回收，一旦触发垃圾回收后，针对老年区域，无论采用哪种算法，都会进行遍历和标记的操作，这里的标记因为存在直接可达和间接可达的操作，所以不必一次性完成，只要找到第一层的可达对象即可停下，然后让程序继续执行一会，然后再让GC做第二步的标记操作，如可达的子元素。标记一段时间后，再让GC停下来让程序继续执行，如此反复交替执行，直到完成垃圾回收。从而把以前长时间的停顿拆分成多个小段的停顿，给用户更好的体验。

### 代码题1

#### 练习1答案：

const fp = require('lodash/fp')

const last_car = fp.last(cars)

const  last_in_stock = fp.prop('in_stock', last_car)

const isLastInStock = fp.flowRight(last_in_stock, last_car)	

#### 练习2答案：	

const fp = require('lodash/fp')

const first_car = fp.first(cars)

const  first_name = fp.prop('name', first_car)

const fist_car_name = fp.flowRight(first_name, first_car)	

#### 练习3答案：

const fp = require('lodash/fp')

let   _average = xs => { fp.reduce(fp.add, 0, xs)/xs.length }

let dollar_values = fp.map(car =>car.dollar_value, cars)

averageDollaValue = fp.flowRight(_average, dollar_values)

#### 练习4答案：

const fp = require('lodash/fp')

let  _____underscore = fp.replace(/\W+/g, '____')

let sanitizeNames = fp.flowRight(fp.map(_underscore), fp.toLower)

### 代码题2

#### 练习1答案：

const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let maybe = Maybe.of([5, 6, 1])

// 实现
let ex1 = (maybe) => (y) => maybe.map(fp.map(x => fp.add(x, y)))
console.log(ex1(maybe)(1))

#### 练习2答案：

const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let xs = Container.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do'])

let ex2 = fp.map(fp.first)
console.log(xs.map(ex2)

#### 练习3答案：

const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let safeProp = fp.curry(function (x, o) { return Maybe.of(o[x]) })
let user = { id: 2, name: "Albert" }
let ex3 = fp.flowRight(fp.map(fp.first), safeProp('name'))
console.log(ex3(user))

#### 练习4档案：

const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let ex4 = function (n) {
  if (n) { return parseInt(n) }
};

ex4 = fp.flowRight(fp.map(parseInt), Maybe.of)

console.log(ex4('5'))