
# Commonjs

提到 CommonJs，第一时间其实想到的就是 Node.js 的模块，这种模块化的写法浏览器是不支持的，其实这句话只说对了一半，CommonJs 是一种模块化规范，Node.js 只是实现了这种规范，就比如 Promise A+ 这是一个实现异步的一种规范，JavaScript只是根据这个规范实现了这个功能，它浏览器端和 Node.js 中的实现也是不完全一致，其他语言中也有根据这个规范实现的库。

   总的来说，CommonJs 是一种规范，今天来讲讲 Node.js 中的模块化语法，它是一个轻微修改版本的CommonJs（这个描述来自于红宝书第四版），就叫它 Node.js 版本的 CommonJs 吧，那其中 module.exports 和 exports 有哪些关联，一起来看看。

话不多说了，直接上例子

```js
// AModule.js
const foo = () => {
 return Math.random();
};

// case 1
module.exports = {
 foo
};
// case 2
module.exports.foo = foo;
// case 3 
exports.foo = foo;
```

 现在我们简单实现一个 foo 函数，然后通过 Node.js 的模块化语法的这三种语法导出。

```js
// index.js

const AModule = require('./AModule');
console.log(AModule);

// case 1 
// { foo: [Function: foo] }

// case 2
// { foo: [Function: foo] }

// case 3
// { foo: [Function: foo] }
```

以上都是比较正确的写法，在 index.js 中都成功获取到 AModule，但是貌似写法不太一样，module.exports 和 exports 傻傻分不清楚，

```js
// AModule.js

const a = '1';
const foo = () => {
 return Math.random();
};

// case 4
exports = {
 foo,
 a
};
// {} 
// 看着就不正常，结果也确实没有获取到

// case 5

exports.foo = foo;
module.exports.a = a;

// { foo: [Function: foo], a: '1' }
// 看似不太正常，但是 foo 和 a 都成功获取到了

// case 6
exports.foo = foo;
module.exports = {}

```

看完前面 5 个 case，这个 case 6，相信聪明的你，已经知道打印的是什么了，已经明白了这俩玩意有什么区别了。

这个东西看完，我个人觉得这个是和对象的赋值是类似，继续往下瞧。

```js
let a = 1;
let b = a;
a = 2;
console.log(b); // 1

let c = { name: "lzb" };
let d = c;
c.name = 'lll';
console.log(d.name); // 'lll'
```

 看完这个，可能就又明白了

 再看下边的例子

```js
// AModule.js

// case 1
module.exports = {
 foo
}
// case 2
exports.foo = foo;

// 这两种是肯定生效的
// 从语法的角度上讲，exports. 和 module. 这种语法证明，
// 1. 在这个当前的作用域下，是可以获取到的有 exports 和 module 这个两个变量
// 2. 这两个变量应该都是对象

// so ?
// 那么我们直接 console.log() 一下，让他们亮个相
```

现在就根据上边的两种 case，分别看一下什么情况；

```js
// AModule.js case 1
console.log(exports);
// {}
console.log(module);
// {
//   id: '...',
//   path: '...',
//   exports: {},
//   filename: '...',
//   loaded: false,
//   children: [],
//   paths: []
// }

module.exports = {
 foo
}

console.log(exports);
// {}
console.log(module);
// 对于当前影响不重要的使用 ... 省略
// {
//   id: '...',
//   path: '...',
//   exports: { foo: [Function: foo] },
//   filename: '...',
//   loaded: false,
//   children: [],
//   paths: []
// }

```

   这里 module.exports 这个的语法，显然是给 module 的 exports 属性赋值，从这个例子中不难看出，最开始 exports 和 module.exports 都是 {}。 而后来 exports 还是 {}, 但是 module.exports 已经被赋值了，此时 exports 和 module.exports，看似有关联，实际上又发现不出来有什么关联，没有证据啊，你找的是鲁迅，跟我周树人有什关系啊，module.exports 这种语法又是大家最常用的，咱们接着往下看

```js
// AModule.js case 2
console.log(exports);
// {}
console.log(module);
// {
//   id: '...',
//   path: '...',
//   exports: {},
//   filename: '...',
//   loaded: false,
//   children: [],
//   paths: []
// }

exports.foo = foo；

console.log(exports);
// { foo: [Function: foo] },
console.log(module);
// 对于当前影响不重要的使用 ... 省略
// {
//   id: '...',
//   path: '...',
//   exports: { foo: [Function: foo] },
//   filename: '...',
//   loaded: false,
//   children: [],
//   paths: []
// }

```

   咱们再看一下这种情况，这种写法应该是第二常用的写法，module.exports.foo = foo; 虽然也好使，但是应该很少有人这个么写，开始两者都是 {}, 导出之后，两者都是 { foo: [Function: foo] }，现在感觉已经初露端倪了。
 大胆想象一下，exports === module.exports 应该是 true，准没错了

```js
// AModule.js case 1
console.log(exports === module.exports);
// true

module.exports = {
 foo
}

console.log(exports === module.exports);
// false

// AModule.js case 2
console.log(exports === module.exports);
// true

exports.foo = foo;

console.log(exports === module.exports);
// true

```

  这个情况跟我们最开始打印 exports 和 module 的情况如出一辙，由此可以断定，exports 和 module.exports 其实就是 c 和 d 一样， 最开始 exports 的值是指向 module.exports 的引用的。

   或者说它俩就是一个引用，exports.foo = foo, 只会改变这个引用的中的值， 而 module.exports = {}, 这种，直接开辟了一个新的引用， 这个 exports 还是指向 module.exports 之前的引用，这个样两者就没有干系了，

   所以也就是说，在 AModule.js 的初始化的时候相当于有这样一段代码

```js
let module = {
 exports: {},
 // ...其他属性
}
let exports = module.exports;

```

也就是说 exports.a = a, 会更新 module.exports, 最终模块的消费方或者使用方，拿的值是 module.exports 的值

总结：不总结了
评论区帮忙总结一下，我看谁说的对
