# Loops

There are many ways to loop through an array or object. Here are 3 commonly used ways

* `for` loop

```javascript
// syntax:
myArray = ['alice', 'tim', 'bob', 'june'];
for (startingValueOfI; endingCondition; incrementStep) {
    // do something with myArray[i]
}

// example:
someArray = ['alice', 'tim', 'bob', 'june'];
for (var i = 0; i < someArray.length; i++) {
    console.log('Hi ' + someArray[i] + '!');
}
```

* `while` loop

```javascript
var n = 0;
while (n < 3) {     // n < 3 returns true or false
    // do something
    console.log("i is " + i);

    // increment n
    n++;
}
```

* `for...of` statement

```javascript
for (variable of iterable) {
  statement
}
```

```javascript
var iterable = [10, 20, 30];

for (var value of iterable) {
  value += 1;
  console.log(value);
}
```

* `for...in` statement

```javascript
for (key in object) {
    // do something with key
};

// Example:
var car = {
    wheels: 4,
    doors: 2,
    seats: 5
};

for (var thing in car) {
    console.log("my car has " + car[thing] + " " + thing)
};
```

Make sure you remember the difference between 'for...of' and 'for...in'!

