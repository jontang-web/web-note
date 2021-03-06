### 内存分配
- 32位操作系统给V8执行引擎分配的内存空间是700Mb；
- 64位操作系统给V8执行引擎分配的内存空间是1.4Gb
- JS把内存空间分为栈，堆和池（也叫栈中）

##### 栈（stack）
> 主要用于存储JS中的基本数据类型，包括Boolean、Number、String、undefined、null，以及引用类型的指针。栈是一个线性排列的内存空间，存储的内容都是空间大小已知或者有有范围上限的，如基本类型的值、堆内存空间块的起始地址等，算作一种简单存储。

特点：
1. 内存空间由系统自动分配和自动释放；
2. 线性存储，先进后出，后进先出；

##### 堆（heap）
> 主要用于存储JS中的引用数据类型，包括Object，Array，Function和new函数产生的对象实例（本质是Object）等。引用类型的指针（堆内存的起始地址）存储在内存栈中，访问引用类型数据时，先从内存栈中获取引用类型的指针，然后根据堆内存地址读取引用类型数据。

特点：
1. 按内存块非线性存储，动态分配，手动释放，可能出现内存泄露；
2. 内存块的起始地址作为一个固定的值存储在内存栈中；

##### 常见问题
1. let和const都不能重复定义变量。在let和const定义变量和常量之前，都会检查栈内存中是否存在同名变量，如果存在就会抛出异常；
2. const定义的基本数据类型不能修改，引用数据类型可以修改内容，且在定义的时候必须初始化。const定义的常量实际上是栈内存中的一个内存空间地址，对于基本数据类型，其值就是内存地址，所以不能修改；对于引用类型，常量的值是堆内存的地址，其值也不能修改，但是堆内存中的内容可以修改，也就是const定义的引用类型内容是可以修改的；
```
const PI = 3.1415;
PI = 3.14;   //报错：TypeError: Assignment to constant variable

const person = {
    name: jontang,
    age: 27
};
person.age = 28;  //正确
```


### 内存回收
JS的内存回收机制：标记清除 和 引用计数
> 引用计数：跟踪记录每个值被引用的次数。增加一个对象引用时，引用次数+1，释放一个引用时，引用次数-1。最终垃圾回收机制会回收那些引用次数为0的对象。但是引用计数算法有一个致命的问题：循环引用。如果两个对象出现相互引用，引用次数永远不会变成0，也就是内存永远不会被回收。为了解决循环引用造成的问题，现代浏览器都通过使用标记清除算法来实现垃圾回收。

算法流程：
1. JS全局保存一个引用计数对象，当一个对象（变量，函数等）被创建时，会在引用计数对象里添加一个该引用次数的标记；
2. 对象被引用增加一次，相应的标记+1；
3. 对象被引用减少一次，相应的标记-1，直至变成0；
4. 最后垃圾回收机制遍历引用计数对象，回收那些引用标记次数为0的对象内存空间

> 标记清除：从算法根部（root，在JS中就是全局对象）出发，定时扫描内存中的对象。凡是能从根部到达的对象，都是还需要使用的。那些无法由根部出发触及到的对象被标记为不再使用，垃圾回收机制进行内存收回

算法流程：
1. 垃圾收集器会在运行的时候会给存储在内存中的所有变量都加上标记；
2. 从根部出发将能触及到的对象的标记清除；
3. 那些还存在标记的变量被视为准备删除的变量；
4. 最后垃圾收集器会执行最后一步内存清除的工作，销毁那些带标记的值并回收它们所占用的内存空间

### 内存泄漏
##### 引起浏览器内存泄漏的情况：
1. 滥用全局变量，因为全局变量在整个程序执行期间都不会回收。解决办法：使用严格模式避免 或者 在全局变量使用完成后设置为null/undefined；
2. 闭包使用不当，闭包可以维持外部函数的执行环境，使其内存不被回收。解决办法：在函数退出之前，把不使用的局部函数全部删除；
3. DOM引用未删除，特别是子元素引用被全局缓存，即使元素被删除，元素引用以及元素的父元素依然会保存在内存中不会被回收； 
4. 定时器没有clear

##### 引起浏览器内存溢出的情况：
1. 全局缓存不设置size，比如用于全局缓存的数组不设置大小。解决办法：添加全局缓存数组size判断，模拟队列（先进先出，unshift）缓存数据
2. 操作大文件，比如上传超过1G的大文件时，会出现内存不够而奔溃。解决办法：应该对大文件进行切片，做断点续传；

##### WeakMap & WeakSet
WeakMap 和 WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakMap / WeakSet 对该对象的引用。也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
