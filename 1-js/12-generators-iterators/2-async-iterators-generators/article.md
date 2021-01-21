
# অ্যাসিঙ্ক ইটারেশন এবং জেনারেটর

অ্যাসিঙ্ক্রোনাসলি আসা ডাটাগুলোকে অ্যাসিঙ্ক্রোনাস ইটারেশনের সাহায্যে আমরা আমাদের চাহিদা মত ইটারেট করতে পারি। উদাহরণস্বরূপ, যখন আমরা নেটওয়ার্কের মাধ্যমে ছোট ছোট অংশ ডাউনলোড করি। এটি অ্যাসিঙ্ক্রোনাস জেনারেটরের সাহায্যে আরো সহজে করা যায়।

চলুন প্রথমে একটি উদাহরণ দেখি, সিনট্যাক্সগুলো বুঝার চেষ্টা করি এবং পরে একটি বাস্তবিক ব্যবহার দেখব।

## পুনরায় ইটারেবল

আসুন পুনরায় ইটারেবলের টপিক্সটি দেখি।

আমাদের একটি অবজেক্ট আছে, যেমন `range`:
```js
let range = {
  from: 1,
  to: 5
};
```

...এবং আমরা `for..of` লুপ চালাব, যেমন `for(value of range)`, `1` হতে `5` পর্যন্ত মানগুলো পেতে।

অন্য কথায় বলা যায়, আমরা আমাদের অবজেক্টটিতে *iteration ability* সাপোর্ট করাব।

এটি আমরা করতে পারি একটি বিশেষ মেথডের সাহায্যে `Symbol.iterator`:

- This method is called in by the `for..of` construct when the loop is started, and it should return an object with the `next` method.
- For each iteration, the `next()` method is invoked for the next value.
- The `next()` should return a value in the form `{done: true/false, value:<loop value>}`, where `done:true` means the end of the loop.

Here's an implementation for the iterable `range`:

```js run
let range = {
  from: 1,
  to: 5,

*!*
  [Symbol.iterator]() { // called once, in the beginning of for..of
*/!*
    return {
      current: this.from,
      last: this.to,

*!*
      next() { // called every iteration, to get the next value
*/!*
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for(let value of range) {
  alert(value); // 1 তারপর 2, তারপর 3, তারপর 4, তারপর 5
}
```
যদি আপনি সাধারণ ইটারেটর নিয়ে আরো বিস্তারিত জানতে চান দয়া করে এটি দেখুন [ইটারেবল চ্যাপ্টার](info:iterable)।

If anything is unclear, please visit the chapter [](info:iterable), it gives all the details about regular iterables.

## অ্যাসিঙ্ক ইটারেটর

অ্যাসিঙ্ক্রোনাস ইটারেটর অনেকটা সাধারণ ইটারেটরের মত, শুধু কিছু সিনট্যাক্সের পার্থক্য আছে।

একটি "সাধারণ" ইটারেবল অবজেক্ট, যা আমরা এই অধ্যায়ে জানতে পারি <info:iterable>, দেখতে অনেকটা এমনঃ

```js run
let range = {
  from: 1,
  to: 5,

  // for..of range কে কল করা হলে সবার শুরুতে এই মেথডটি এক্সিকিউট হবে
*!*
  [Symbol.iterator]() { // called once, in the beginning of for..of
*/!*
    // ...এটি রিটার্ন করে ইটারেটর অবজেক্ট:
    // পরবর্তীতে, for..of শুধু অবজেক্টির সাথে কাজ করে এবং এর মাধ্যমে আমরা মান গুলো জানতে পারি
    // পরবর্তী মানটি next() এর মাধ্যমে জানা যায়
    return {
      current: this.from,
      last: this.to,

      // for..of লুপের প্রতিবার ইটারেশনে next() কল হয়
*!*
      next() { // (2)
        // এটি রিটার্ন করবে একটি অবজেক্ট {done:.., value :...}
*/!*
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for(let value of range) {
  alert(value); // 1 তারপর 2, তারপর 3, তারপর 4, তারপর 5
}
```
যদি আপনি সাধারণ ইটারেটর নিয়ে আরো বিস্তারিত জানতে চান দয়া করে এটি দেখুন [ইটারেবল চ্যাপ্টার](info:iterable)।

অ্যাসিঙ্ক্রোনাসলি ইটারেটরেবল অবজেক্ট তৈরি করতে:
1. আমাদের `Symbol.iterator` এর পরিবর্তে `Symbol.asyncIterator` ব্যবহার করা লাগবে।
2. `next()` অবশ্যই একটি প্রমিস রিটার্ন করবে।
3. এইধরনের অবজেক্ট ইটারেট করতে, আমরা ব্যবহার করব `for await (let item of iterable)` লুপ।

চলুন আগেরটির মত একটি ইটারেটরেবল `range` অবজেক্ট তৈরি করি কিন্তু এটি অ্যাসিঙ্ক্রোনাসলি প্রতি সেকেন্ডে একটি মান রিটার্ন করবেঃ

```js run
let range = {
  from: 1,
  to: 5,

  // for await..of কে কল করা হলে সবার শুরুতে এই মেথডটি এক্সিকিউট হবে
*!*
  [Symbol.asyncIterator]() { // (1)
*/!*
    // ...এটি রিটার্ন করে ইটারেটর অবজেক্ট:
    // পরবর্তীতে, for await..of শুধু অবজেক্টির সাথে কাজ করে এবং এর মাধ্যমে আমরা মান গুলো জানতে পারি
    // পরবর্তী মানটি next() এর মাধ্যমে জানা যায়
    return {
      current: this.from,
      last: this.to,

      //for await..of লুপের প্রতিবার ইটারেশনে next() কল হয়
*!*
      async next() { // (2)
        // এটি রিটার্ন করবে একটি অবজেক্ট {done:.., value :...}
        // (স্বয়ংক্রিয়ভাবে এটি অ্যাসিঙ্ক এর মাধ্যমে একটি প্রমিসে র‍্যাপ হয়)
*/!*

*!*
        // অ্যাসিঙ্ক কাজগুলো আমরা এওয়েট এর ভিতর করতে পারিঃ
        await new Promise(resolve => setTimeout(resolve, 1000)); // (3)
*/!*

        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

(async () => {

*!*
  for await (let value of range) { // (4)
    alert(value); // 1,2,3,4,5
  }
*/!*

})()
```

আমরা দেখতে পাচ্ছি এটির স্ট্রাকচার অনেকটা সাধারণ ইটারেটরের মতঃ

1. একটি অ্যাসিঙ্ক্রোনাসলি ইটারেটরেবল অবজেক্ট তৈরি করতে `Symbol.asyncIterator` `(1)` মেথডটি অবশ্যই লাগবে।
2. মেথডটি `next()` মেথডে একটি প্রমিস এর মাধ্যমে অবজেক্টটি রিটার্ন করে `(2)`।
3. The `next()` method doesn't have to be `async`, it may be a regular method returning a promise, but `async` allows to use `await`, so that's convenient. Here we just delay for a second `(3)`।
4. ইটারেটের জন্য আমরা `for await(let value of range)` `(4)` ব্যবহার করি, লুপে "for" এর পর "await" ব্যবহার করুন। এটি প্রথমে কল করে `range[Symbol.asyncIterator]()` কে এবং পরে মানের জন্য `next()` কে।

ছোট একটি চিটশিট:

|       | ইটারেটর | অ্যাসিঙ্ক ইটারেটর |
|-------|-----------|-----------------|
| ইটারেটরে প্রদত্ত অবজেক্ট মেথড | `Symbol.iterator` | `Symbol.asyncIterator` |
| `next()` এর রিটার্নকৃত মান              | যেকোন মান         | `Promise`  |
| ব্যবহৃত লুপ                          | `for..of`         | `for await..of` |


````warn header="স্প্রেড অপারেটর `...` অ্যাসিঙ্ক্রোনাসলি কাজ করে না"
এটি সিঙ্ক্রোনাস ইটারেটরের একটি ফিচার, অ্যাসিঙ্ক্রোনাসের সাথে কাজ করে না।


উদাহরণস্বরূপ, এখানে স্প্রেড অপারেটর কাজ করবে নাঃ
```js
alert( [...range] ); // Error, no Symbol.iterator
```

এটিই স্বাভাবিক কেননা এটি `Symbol.iterator` কে খুঁজে `for..of` এর মত `Symbol.asyncIterator` কে না।
````

## অ্যাসিঙ্ক জেনারেটর

ইতোমধ্যে আমরা জানি জাভাস্ক্রিপ্ট জেনারেটর সাপোর্ট করে যা ইটারেবল।

চলুন একটি সিক্যুয়েন্স জেনারেটরকে কল করি যা এই অধ্যায়ে [](info:generators) জেনেছিলাম। এটি একটি সিক্যুয়েন্স মান তৈরি করে `start` হতে `end` পর্যন্তঃ

```js run
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for(let value of generateSequence(1, 5)) {
  alert(value); // 1, তারপর 2, তারপর 3, তারপর 4, তারপর 5
}
```

সাধারণ জেনারেটরে আমরা `await` ব্যবহার করতে পারি না। সকল মান সিঙ্ক্রোনাসলি আসেঃ এখানে `for..of` এ কোন লাইনে এ ডিলে হয় না, এটি একটি সিঙ্ক্রোনাস কন্সট্রাক্ট।

কিন্ত যদি আমরা জেনারেটর বডিতে `await` ব্যবহার করি কি হবে? যেমন নেটওয়ার্ক রিকুয়েস্টের সাথে কাজ করার জন্য।

কোন সমস্যা নেই, আমরা এভাবে `async` কে প্রিপেন্ড করে নিতে পারিঃ

```js run
*!*async*/!* function* generateSequence(start, end) {

  for (let i = start; i <= end; i++) {

*!*
    // হুররে, আমরা এওয়েট ব্যবহার করতে পারছি!
    await new Promise(resolve => setTimeout(resolve, 1000));
*/!*

    yield i;
  }

}

(async () => {

  let generator = generateSequence(1, 5);
  for *!*await*/!* (let value of generator) {
    alert(value); // 1, তারপর 2, তারপর 3, তারপর 4, তারপর 5
  }

})();
```

এখন আমাদের অ্যাসিঙ্ক জেনারেটর আছে যা `for await...of` এর দ্বারা ইটারেটবল।

এটি খুব সহজ, আমরা একটি `async` যুক্ত করি এবং জেনারেটরের মধ্যে প্রমিস অথবা অন্যান্য অ্যাসিঙ্ক ফাংশনের এর উপর ভিত্তি করে `await` ব্যবহার করতে পারি।

টেকনিক্যালি, অ্যাসিঙ্ক জেনারেটরের আরেকটি পার্থক্য হল এখন `generator.next()` এই মেথডটিও অ্যাসিঙ্ক্রোনাস, যা একটি প্রমিস রিটার্ন করে।

সাধারণ জেনারেটরে আমরা মান এভাবে পাই `result = generator.next()`. কিন্তু অ্যাসিঙ্ক জেনারেটরে মান পেতে আমাদের `await` যুক্ত করতে হবে, এভাবেঃ

```js
result = await generator.next(); // result = {value: ..., done: true/false}
```
That's why async generators work with `for await...of`.
````

## অ্যাসিঙ্ক ইটারেটবল।

ইতোমধ্যে আমরা জেনেছি ইটারেবল অবজেক্ট তৈরি করতে আমাদের `Symbol.iterator` যুক্ত করতে হয়।

```js
let range = {
  from: 1,
  to: 5,
*!*
  [Symbol.iterator]() {
    return <next মেথডের সাহায্যে একটি ইটারেবল range তৈরি করি>
  }
*/!*
}
```

`Symbol.iterator` এর মাধ্যমে জেনারেটর রিটার্নের জন্য এর একটি কমন প্রাকটিস হল আগের উদাহরণের মত `next` এর মাধ্যমে অবজেক্ট রিটার্ন করা।

চলুন এই অধ্যায়ের [](info:generators) উদাহরনটি আবার কল করিঃ

```js run
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // [Symbol.iterator]: function*() এর সংক্ষিপ্তরূপ
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

for(let value of range) {
  alert(value); // 1, তারপর 2, তারপর 3, তারপর 4, তারপর 5
}
```

এখানে `range` কাস্টম অবজেক্টটি একটি ইটারেবল, এবং `*[Symbol.iterator]` জেনারেটরটি দ্বারা মানগুলো দেখাতে লজিক প্রয়োগ করি।

যদি আমরা জেনারেটরে অ্যাসিঙ্ক অ্যাকশন যুক্ত করতে চাই আমাদের `Symbol.iterator` এর পরিবর্তে async `Symbol.asyncIterator` ব্যবহার করতে হবেঃ

```js run
let range = {
  from: 1,
  to: 5,

  // this line is same as [Symbol.asyncIterator]: async function*() {
*!*
  async *[Symbol.asyncIterator]() {
*/!*
    for(let value = this.from; value <= this.to; value++) {

      // দুটি মানের মধ্যেবর্তী কিছুক্ষণের জন্য অপেক্ষা করবে
      await new Promise(resolve => setTimeout(resolve, 1000));

      yield value;
    }
  }
};

(async () => {

  for *!*await*/!* (let value of range) {
    alert(value); // 1, তারপর 2, তারপর 3, তারপর 4, তারপর 5
  }

})();
```

এখন মানগুলো ১ সেকেন্ড পর পর আসবে।

## একটি বাস্তবিক উদাহরণ

এ পর্যন্ত আমরা কিছু সাধারন উদাহরণ এর সাহায্যে বেসিক ব্যাপারগুলো দেখলাম। চলুন এখন আমরা একটি বাস্তবিক ব্যবহার দেখি।

অনলাইনে অনেক সার্ভিস আছে যারা পেজিনেটেড ডাটা সরবরাহ করে। উদাহরণস্বরূপ, যখন আমাদের ইউজারদের একটি লিস্ট দরকার হয় এটি "এক পেজের" জন্য একটি রিকুয়েস্টে প্রি-ডিফাইন্ড (যেমনঃ ১০০ জন ইউজারের) ডাটা রিটার্ন করে , এবং পরবর্তী পেজে যাওয়ার জন্য একটি URL প্রদান করে।

এটি একটি কমন প্যাটার্ন। এটি শুধু ইউজারের জন্য না যেকোন ধরনের লিস্টের জন্য হতে পারে। উদাহরণস্বরূপ, গিটহাব হতে আমরা পেজিনেটেড উপায়ে কমিট নিয়ে আসতে পারিঃ

- এই জন্য আমাদের এই `https://api.github.com/repos/<repo>/commits` URL এ একটি রিকুয়েস্ট করা লাগবে।
- এটি জেসন ফরম্যাটে ৩০ টি কমিট রেসপন্ড করে, এবং পরবর্তী পেজে যাওয়ার জন্য `Link` হেডারের মাধ্যমে একটি লিঙ্ক প্রদান করে।
- তারপর আমরা পরবর্তী কমিটগুলোর জন্য লিঙ্কটির মাধ্যমে আরো রিকুয়েস্ট পাঠাতে পারি, এভাবে চলতে থাকবে।

এখন আমরা একটি সিম্পল API চাই: যেটি কমিটগুলোর একটি ইটারেবল অবজেক্ট হয়, তো আমরা এভাবে করতে পারিঃ

```js
let repo = 'javascript-tutorial/en.javascript.info'; // যে গিটহাব রিপোজেটরী থেকে কমিট নিয়ে আসব

for await (let commit of fetchCommits(repo)) {
  // কমিটগুলো প্রসেস করি
}
```

আমরা কমিট নিয়ে আসার জন্য একটি `fetchCommits(repo)` ফাংশন তৈরি করি, যা প্রয়োজনমত রিকুয়েস্ট তৈরি করে। এবং পেজিনেশনের সকল ব্যাপার এটির মাধ্যমে নিয়ন্ত্রিত হয়। এটি একটি সিম্পল `for await..of`।

অ্যাসিঙ্ক জেনারেটরের মাধ্যমে আমরা সহজেই এটি তৈরি করতে পারিঃ

```js
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`;

  while (url) {
    const response = await fetch(url, { // (1)
      headers: {'User-Agent': 'Our script'}, // গিটহাবে User-Agent হেডার দরকার
    });

    const body = await response.json(); // (2) রেস্পন্স করবে জেসন (কমিটের অ্যারেসমূহ)

    // (3) পরবর্তী পেজের URL টি হেডার থেকে এক্সট্রাক্ট করি
    let nextPage = response.headers.get('Link').match(/<(.*?)>; rel="next"/);
    nextPage = nextPage?.[1];

    url = nextPage;

    for(let commit of body) { // (4) কমিটগুলো একটির পর একটি আসবে পেজ শেষ না হওয়া পর্যন্ত
      yield commit;
    }
  }
}
```


1. আমরা রিমোট URL থেকে কিছু ডাউনলোডের জন্য ব্রাউজারের [fetch](info:fetch) মেথডটি ব্যবহার করতে পারি। এটির মাধ্যমে আমরা অথোরাইজেশন এবং অন্যান্য হেডার পাস করতে পারি -- গিটহাব API এ রিকুয়েস্টের জন্য আমাদের `User-Agent` হেডারটি পাঠানো প্রয়োজন
2. `fetch` এর মাধ্যমে আমরা একটি জেসন অবজেক্ট পাই, যা `fetch`এর একটি নির্দিষ্ট মেথড।
3. আমরা পরবর্তী পেজের URL টি রেস্পন্সের `Link` হেডার হতে পাই।  এটি একটি স্পেশাল ফরম্যাটে থাকে তাই আমরা রেগুলার এক্সপ্রেশন  ব্যবহারের এর মাধ্যমে পরবর্তী পেজের লিঙ্কটি পায়। পরবর্তী পেজের URL টি দেখতে এমন হয় `https://api.github.com/repositories/93253246/commits?page=2`। এটি গিটহাবের মাধ্যমে জেনারেট হয়।
4. তারপর আমরা `yield` এর মাধ্যমে কমিট গুলো রিসিভ করি এবং যখন এটি শেষ হয়, `while(url)` লুপ আবার রান হয় এবং এর ফলে আরো রিকুয়েস্ট জেনারেট হয়।

উদাহরণস্বরূপ (কনসোলে কমিটের অথরের নাম দেখায়):

```js run
(async () => {

  let count = 0;

  for await (const commit of fetchCommits('javascript-tutorial/en.javascript.info')) {

    console.log(commit.author.login);

    if (++count == 100) { // ১০০ কমিট আসার পর লুপ থেমে যায়
      break;
    }
  }

})();
```

এটি আমাদের চাহিদামত কাজ করে। এখানে পেজিনেশেনের পুরো ব্যাপারটা ইন্টারনালি সংগঠিত হয় এবং অ্যাসিঙ্ক জেনারেটর আমাদের কমিটগুলো রিটার্ন করে।

## সারাংশ

সাধারণ ইটারেটর এবং জেনারেটর গুলো সিঙ্ক্রোনাস ডাটার সাথে ভালোভাবেই কাজ করে।

যখন আমাদের ডাটা গুলো অ্যাসিঙ্ক্রোনাসলি আসে তখন আমাদের `async` ব্যবহার করা উচিত এবং `for..of` এর বদলে `for await..of` ব্যবহার করতে হবে।

অ্যাসিঙ্ক এবং সাধারণ ইটারেটরের মধ্যে সিনট্যাক্সের পার্থক্য:

|       | ইটারেবল | অ্যাসিঙ্ক ইটারেবল |
|-------|-----------|-----------------|
| ইটারেটরে ব্যবহৃত মেথড | `Symbol.iterator` | `Symbol.asyncIterator` |
| `next()` এর রিটার্ন ভ্যালু          | `{value:…, done: true/false}`         | `Promise` এর `resolve` রিটার্ন করে `{value:…, done: true/false}`  |

অ্যাসিঙ্ক এবং সাধারণ জেনারেটরের মধ্যে সিনট্যাক্সের পার্থক্যঃ

|       | জেনারেটর | অ্যাসিঙ্ক জেনারেটর |
|-------|-----------|-----------------|
| ডিক্লেয়ার করার নিয়ম | `function*` | `async function*` |
| `next()` এর রিটার্ন ভ্যালু          | `{value:…, done: true/false}`         | `Promise` এর `resolve` রিটার্ন করে `{value:…, done: true/false}`  |

ওয়েব ডেভলাপমেন্টে আমাদের প্রায়শই ছোট ছোট করে ডাটা স্ট্রীমের দরকার হয়। উদাহরনস্বরূন, একটি বড় ফাইল ডাউনলোড বা আপলোডের জন্য।


আমরা এই ধরনের ডাটার জন্য অ্যাসিঙ্ক জেনারেটর ব্যবহার করতে পারি। বিঃদ্রঃ কিছু এনভাইরনমেন্টে যেমন ব্রাউজারে একটি Streams API আছে, এটি ডাটাকে এক স্ট্রীম থেকে অন্য স্ট্রীমে পাঠাতে একটি বিশেষ ইন্টারফেস দেয় (যেমনঃ এক জায়গা হতে ডাউনলোড করে সাথে সাথে অন্য আরেক জায়গায় পাঠানো)।

