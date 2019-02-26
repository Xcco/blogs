> * 原文地址：[Avoiding those dang cannot read property of undefined errors](https://css-tricks.com/%E2%80%8B%E2%80%8Bavoiding-those-dang-cannot-read-property-of-undefined-errors/)
> * 原文作者：[Adam Giese](https://css-tricks.com/author/thirdgoose/)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/avoiding-those-dang-cannot-read-property-of-undefined-errors.md](https://github.com/xitu/gold-miner/blob/master/TODO1/avoiding-those-dang-cannot-read-property-of-undefined-errors.md)
> * 译者：
> * 校对者：

# 避免那些可恶的"cannot read property of undefined"错误

`Uncaught TypeError: Cannot read property 'foo' of undefined.` 是一个我们在 JavaScript 开发中都遇到过的可怕错误。 或许是某个 API 返回了意料外的空值，又或许是其他什么原因， 这个错误是如此的普遍而广泛以至于我们无法判断。

我最近遇到了一个问题，某一环境变量出于某种的原因没有被加载，导致各种各样的报错夹杂着这个错误摆在我面前。 不论什么原因，放着这个错误不处理都会是灾难性的。所以我们该怎么从源头阻止这个问题发生呢？

让我们一起来找出解决方案。

### 工具库

如果你已经在项目里用到一些工具库，很有可能库里已经有了阻止这个问题产生的函数。lodash 里的 `_.get`([docs](https://lodash.com/docs/4.17.11#get)) 或者 Ramda 里的 `R.path`([docs](https://ramdajs.com/docs/#path)) 都能确保你安全使用对象。  
  
如果你已经使用了工具库，那么这看起来已经是最简单的方法了。如果你没有使用工具库，继续读下去吧！

### && 短路

JavaScript 里一个关于逻辑运算符有趣的事实就是它不总是返回布尔值。[根据说明](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-binary-logical-operators)，“`&&` 或者 `||` 操作符的返回值并不需要是布尔值。返回值总是两个操作表达式的其中之一。”  
 
举个 `&&` 操作符的例子, 如果第一个表达式的值是“假”的，那么该值就会被返回。否则，第二个表达式就会被使用。这说明表达式 `0 && 1` 将会得到 `0` (一个假值), and the expression `2 && 3` will be evaluated as `3`. If multiple `&&` expressions are chained together, they will evaluate to either the first falsy value or the last value. For example, `1 && 2 && 3 && null && 4` will evaluate to `null`, and `1 && 2 && 3` will evaluate to `3`.

How is this useful for safely accessing nested object properties? Logical operators in JavaScript will "short-circuit." In this case of `&&`, this means that the expression will cease moving forward after it reaches its first falsy value.

```
const foo = false && destroyAllHumans();
console.log(foo); // false, and humanity is safe
```

In this example, `destroyAllHumans` is never called because the `&&` operand stopped all evaluation after false​.

This can be used to safely access nested properties.

```
const meals = {
  breakfast: null, // I skipped the most important meal of the day! :(
  lunch: {
    protein: 'Chicken',
    greens: 'Spinach',
  },
  dinner: {
    protein: 'Soy',
    greens: 'Kale',
  },
};

const breakfastProtein = meals.breakfast && meals.breakfast.protein; // null
const lunchProtein = meals.lunch && meals.lunch.protein; // 'Chicken'
```

Aside from its simplicity, one of the main advantages of this approach is its brevity when dealing with small chains. However, when accessing deeper objects, this can be quite verbose.

```
const favorites = {
  video: {
    movies: ['Casablanca', 'Citizen Kane', 'Gone With The Wind'],
    shows: ['The Simpsons', 'Arrested Development'],
    vlogs: null,
  },
  audio: {
    podcasts: ['Shop Talk Show', 'CodePen Radio'],
    audiobooks: null,
  },
  reading: null, // Just kidding -- I love to read
};

const favoriteMovie = favorites.video && favorites.video.movies && favorites.video.movies[0];
// Casablanca
const favoriteVlog = favorites.video && favorites.video.vlogs && favorites.video.vlogs[0];
// null
```

The more deeply nested an object is, the more unwieldy it gets.

### The “Maybe Monad”

Oliver Steele came up with this method and goes through it in much more detail in his blog post, ["Monads on the Cheap I: The Maybe Monad."](https://blog.osteele.com/2007/12/cheap-monads/) I will attempt to give a brief explanation here.

```
const favoriteBook = ((favorites.reading||{}).books||[])[0]; // undefined
const favoriteAudiobook = ((favorites.audio||{}).audiobooks||[])[0]; // undefined
const favoritePodcast = ((favorites.audio||{}).podcasts||[])[0]; // 'Shop Talk Show'
```

Similar to the short-circuit example above, this method works by checking if a value is falsy. If it is, it will attempt to access the next property on an empty object. In the example above, favorites.reading is null, so the books property is being accessed from an empty object. This will result in an undefined, so the 0 will likewise be accessed from an empty array.

The advantage of this method over the `&&` method is that it avoids repetition of property names. On deeper objects, this can be quite a significant advantage. The primary disadvantage would be readability — it is not a common pattern, and may take a reader a moment to parse out how it is working.

### try/catch

`try...catch` statements in JavaScript allow another method for safely accessing properties.

```
try {
  console.log(favorites.reading.magazines[0]);
} catch (error) {
  console.log("No magazines have been favorited.");
}
```

Unfortunately, in JavaScript, `try...catch` statements are not expressions. They do not evaluate to a value as they do in some languages. This prevents a concise try statement as a way of setting a variable.

One option is to use a let variable that is defined in the block above the `try...catch`.

```
let favoriteMagazine;
try { 
  favoriteMagazine = favorites.reading.magazines[0]; 
} catch (error) { 
  favoriteMagazine = null; /* any default can be used */
};
```

Although it’s verbose, this works for setting a single variable (that is, if the mutable variable doesn't scare you off). However, issues can arise if they’re done in bulk.

```
let favoriteMagazine, favoriteMovie, favoriteShow;
try {
  favoriteMovie = favorites.video.movies[0];
  favoriteShow = favorites.video.shows[0];
  favoriteMagazine = favorites.reading.magazines[0];
} catch (error) {
  favoriteMagazine = null;
  favoriteMovie = null;
  favoriteShow = null;
};

console.log(favoriteMovie); // null
console.log(favoriteShow); // null
console.log(favoriteMagazine); // null
```

If any of the attempts to access the property fails, this will cause all of them to fall back into their defaults.

An alternative is to wrap the `try...catch`​ in a reusable utility function.

```
const tryFn = (fn, fallback = null) => {
  try {
    return fn();
  } catch (error) {
    return fallback;
  }
} 

const favoriteBook = tryFn(() => favorites.reading.book[0]); // null
const favoriteMovie = tryFn(() => favorites.video.movies[0]); // "Casablanca"
```

By wrapping the access to the object in a function, you can delay the "unsafe" code and pass it into a `try...catch`.

A major advantage of this method is how natural it is to access the property. As long as properties are wrapped in a function, they are safely accessed. A default value can also be specified in the case of a non-existent path.

### Merge with a default object

By merging an object with a similarly shaped object of "defaults," we can ensure that the path that we are trying to access is safe.

```
const defaults = {
  position: "static",
  background: "transparent",
  border: "none",
};

const settings = {
  border: "1px solid blue",
};

const merged = { ...defaults, ...settings };

console.log(merged); 
/*
  {
    position: "static",
    background: "transparent",
    border: "1px solid blue"
  }
*/
```

Careful, though, because the entire nested object can be overwritten rather than a single property.

```
const defaults = {
  font: {
    family: "Helvetica",
    size: "12px",
    style: "normal",
  },        
  color: "black",
};

const settings = {
  font: {
    size: "16px",
  }
};

const merged = { 
  ...defaults, 
  ...settings,
};

console.log(merged.font.size); // "16px"
console.log(merged.font.style); // undefined
```

Oh no! To fix this, we'll need to similarly copy each of the nested objects.

```
const merged = { 
  ...defaults, 
  ...settings,
  font: {
    ...defaults.font,
    ...settings.font,
  },
};

console.log(merged.font.size); // "16px"
console.log(merged.font.style); // "normal"
```

Much better!

This pattern is common with plugins or components that accept a large settings object with included defaults.

A bonus about this approach is that, by writing a default object, we’re including documentation on how an object should look. Unfortunately, depending on the size and shape of the data, the "merging" can be littered with copying each nested object.

### The future: optional chaining

There is currently a TC39 proposal for a feature called "optional chaining." This new operator would look like this:

```
console.log(favorites?.video?.shows[0]); // 'The Simpsons'
console.log(favorites?.audio?.audiobooks[0]); // undefined
```

The `?.` operator works by short-circuiting: if the left-hand side of the `?.` operator evaluates to `null` or `undefined`, the entire expression will evaluate to `undefined` and the right-hand side will remain unevaluated.

To have a custom default, we can use the `||` operator in the case of an undefined.

```
console.log(favorites?.audio?.audiobooks[0] || "The Hobbit");
```

### Which method should you use?

The answer, as you might have guessed, is that age-old answer… "it depends." If the optional chaining operator has been added to the language and has the necessary browser support, it is likely the best option. If you are not from the future, however, there are more considerations to take into account. Are you using a utility library? How deeply nested is your object? Do you need to specify defaults? Different cases may warrant a different approach.

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
