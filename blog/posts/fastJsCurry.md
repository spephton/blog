# A fast and efficient JavaScript curry implementation

This was a [Thirty Days of JavaScript](https://leetcode.com/discuss/study-guide/3458761/Open-to-Registration!-30-Days-of-LC-JavaScript-Challenge/?utm_campaign=Banner10&utm_medium=Banner&utm_source=Banner&gio_link_id=noqbNLA9) challenge over on LeetCode. It's a nice sequence, building up to harder problems and focussing on fundametals, I do recommend it!

I was proud that I managed to work this out sans google/the editorial. It's also faster *and* simpler than the provided solutions and 90%+ of submissions on speed *and* memory efficiency. This is efficiency comes from avoiding object creation and recursion, and we don't add complexity to avoid those.

## What the heck is currying?

If you have a function of several arguments, e.g. `add` below,it might be convenient to set the value of the first argument in a new function like `add2`.

```javascript
function add(a, b) {
    return a + b;
}

add2 = add(2); // returning add, but with a set to 2

// which we would call like
console.log(add2(4)); // 6
```

This is a nice interface when you want a more specific function derived from a more general one. The process of taking a function and making it callable in this manner is called 'currying' after Haskell Curry, who was a freak for it. Currying enables what we call partial application (like `add2`, which is a partially applied version of `add`).

You can achieve partial application with tools like JS's `Function.prototype.bind()` and Python's `functools.partial()`, but a function `curry()` that takes a normal function like `add()` and makes it partially applyable in the manner depicted is handy -- and that's what is implemented below.

## Code
```javascript
const curry = function(fn) {
    let argsArr = new Array();
    return function curried(...args) {
        argsArr.push(...args);
        if (argsArr.length >== fn.length) {
            return fn(...argsArr);
        }
        else {
            return curried;
        }
    }
}
```
We define a [closure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#), `curried()`, over an array where we can store arguments. Whenever the closure is called, we add the args to our array. If we have enough args in the array to call the function, we do so and return the result. If we don't, we return our closure, so that next time it's called we can collect more arguments. That's it!

You can then use it like this:


```javascript
const curry = function(fn) {
 * function sum(a, b) { return a + b; }
 * const csum = curry(sum);
 * csum(1)(2) // 3
 */
```

Other implementations suggested over there involved a bunch of recursion and spread syntax that some people get off on, but this is easier to understand and faster.

Bye!
