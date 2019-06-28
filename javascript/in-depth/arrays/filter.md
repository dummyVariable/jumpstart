# filter

## `.filter()`

Like `.forEach()` and `.map()`, `.filter()` takes in a callback function that can take in 3 positional parameters: `element`, `index`, `container`.

However, unlike `.forEach()` and `.map()`, the callback function **must** return `true` or `false`. This callback is also known as **predicate**

Syntax:

```javascript
const newArray = someArray.filter(function(element[, index, array]) {

})
```

[Check out the docs.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

**Example**

```javascript
const numbers = [1,2,3,4,5,6]

function isEvenNumber( number) {
  return number % 2 == 0  // this returns true if the number has no remainder when divided by 2
}

const evenNumbers = numbers.filter(isEvenNumber);

console.log(evenNumbers)
// [2, 4, 6]

function isOddNumber( number) {
  return !isEvenNumber(number);
}

const oddNumbers = numbers.filter(isOddNumber);

console.log(oddNumbers)
// [1, 3, 5]
```

## Resources

* [More about filter](http://adripofjavascript.com/blog/drips/filtering-arrays-with-array-filter.html)

