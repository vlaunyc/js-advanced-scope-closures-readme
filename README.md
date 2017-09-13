JavaScript Closures and Higher Order Functions
---

## Objectives

1. Explain what a closure is
2. Explain how a closure works
3. Practice using a closure
4. Describe use cases for closures in JavaScript

## Introduction

As you may have seen, JavaScript gives us mechanisms to construct objects with a specific state.  By that, we mean that we can construct objects such that some behavior is shared, and some data is different.

```js
  class Item {
    constructor(name, manufacturePrice){
      this.name = name
      this.manufacturePrice = manufacturePrice
    }
    retailPrice(marketMultiplier){
      return marketMultiplier * manufacturePrice;
    }
  }

  let tennisShoe = new Item('tennis shoe', 30)
  // {name:  'tennis shoe', manufacturePrice: 30}
  tennisShoe.retailPrice(1.5)
  // 45

  let tshirt = new Item('tshirt', 10)
  // {name:  'tshirt', manufacturePrice: 10}
  tshirt.retailPrice(1)
  // 10
```

In the code above, we use a class constructor to create different objects.  The objects share behavior and vary in their data.  Each item is a different `manufacturePrice`.  The market multiplier represents the varying difference in price for different products.  For example, we multiply our `tennisShoe` by `1.5` to accommodate for our New York market.  In a suburban market, the marketMultiplier of our `tshirt` is `1`, or not marked up.  

What may be surprising is that in JavaScript, we can also create **functions** that share specific capabilities but change others.  So just like we can create objects as little units of work, we can also create functions.

How do you create a function?  It's not too difficult, we simply return a function from another function.

```js
  function retailPriceMaker(manufacturePrice){
    return function(manufacturePrice, marketMultiplier){
      return marketMultiplier * manufacturePrice;
    }
  }

  typeof retailPriceMaker()
  // "function"
```

Let's pay careful attention to what happened in the above code.  We declared a function `retailPriceMaker` whose return value is a function itself.  The returned function takes arguments of  `manufacturePrice` and `marketMultiplier` and returns a retail price that is a function of the two.  

```javascript
  let retailPriceForTen = retailPriceMaker(10)
  // the execution of retailPriceMaker returns a function that takes one argument
  // We assign this returned function to a variable called retailPrice, and then can call the returned function by referencing retailPriceForManufactureTen
  retailPriceForTen(1.5)
  // 15
```

Ok, so why would we want to do such a thing?  Well just like we have objects that we want configured in a special way, where we know some data at one time, but want to execute a method on the object at a different time, the same thing occurs with functions.  Let's explore another example.  Imagine we believe the manufacture price of an item will be 20, and we want to experiment with it's price in different markets.  

```javascript
function retailPriceMaker(manufacturePrice){
  return function(marketMultiplier){
    return marketMultiplier * manufacturePrice;
  }
}

let retailPriceForThree = retailPriceMaker(3)

retailPriceForThree(1.1)
// 3.3

retailPriceForThree(1.5)
// 4.5
```

So now, by invoking `retailPriceMaker` we return a function that has it's own unique attribute of a `manufacturePrice`.  And we use this attribute at a later time.  So just like we earlier defined a `Item` class that has it's `manufacturePrice`, here we avoided all of that code by just directly giving this function knowledge of `manufacturePrice`.  So in JavaScript, functions can hold onto state in the same way that objects can.  This makes sense, as functions are first class objects.

Let's take a deeper look as to what is happening in the code. Look at the code again, below. As you see, `retailPriceForThree` points to our returned JavaScript function.  If you type `retailPriceForThree` into the console you will see that function.

```js

function retailPriceMaker(manufacturePrice){
  return function(marketMultiplier){
    return marketMultiplier * manufacturePrice;
  }
}

let retailPriceForNine = retailPriceMaker(9)

retailPriceForNine
// ƒ (marketType){
// return marketMultiplier * manufacturePrice;
//     }  
```

And if you type the `manufacturePrice` into the console, you will see that it is not defined in the current global scope.  Yet, somehow, when we execute this `retailPriceForNine` function it knows that the `manufacturePrice` is 9.  It knows this, even though `retailPriceForNine` points to a function that does not have either variable defined in its execution scope.  So how does the function have values for length and manufacturePrice?  Placing a debugger into our code and running it in our chrome console show us.  

<!-- Display Screen Shot Here -->

`manufacturePrice` price is defined because of a closure.  A closure is the attribute that all JavaScript functions have: *JavaScript functions hold onto the scope that they had when they were declared*.  Let's take a look at our code again to see how we made use of a closure.  

```javascript
function retailPriceMaker(manufacturePrice){
  return function(marketMultiplier){
    return marketMultiplier * manufacturePrice;
  }
}

let retailPriceForNine = retailPriceMaker(9)

retailPriceForNine(3)
// 27
```

So every time we execute the `retailPriceMaker` function we are declaring a new function.  That's what our `retailPriceMaker` function does: declare a function that it then returns.  And when that function is declared, manufacturePrice is in scope.  So it doesn't matter that manufacturePrice price is not in scope when we later execute a function.  There is a closure such that the function that `retailPriceMaker` returns holds onto the scope it was declared with.  This becomes a very powerful feature in JavaScript.  Closures allow us to  build functions that have their own capabilities.  

So just like we can return a function called `retailPriceForNine` and then see the `retailPrice` returned with various marketTypes passed through, we can construct another function called `retailPriceForSixteen` for say a different Item.

```js
function retailPriceMaker(manufacturePrice){
  return function(marketType){
    return marketMultiplier * manufacturePrice;
  }
}

let retailPriceForNine = retailPriceMaker(9)

let retailPriceForSixteen = retailPriceMaker(16)

retailPriceForNine(2)
// 18

retailPriceForSixteen(1.5)
// 24
```

So as you see from above, our `retailPriceMaker` function lives up to its name: it returns a function that calculates the `retailPrice` for various markets.  If we want to be able to calculate the `retailPrice` for another `retailPrice`, we can simply invoke our `retailPriceMaker` again.

### Privacy

Thus far, we have seen how we can use closures to return functions which have various attributes that they permanently hold onto.  Closures are used for one other capability in JavaScript: privacy.  As you can see above, once we invoke `retailPriceMaker` and we pass through `manufacturePrice`, it is impossible for us to ever change this attribute.  This attribute is only even readable from inside the function, and we have defined our function in such a way that there is no other way to ever the `manufacturePrice`.  

So here, our returned functions provides some capability that JavaScript objects do not.  Encapsulation.  Remember that we can always change the data of an object.

```js

  class Item {
    constructor(manufacturePrice, marketType){
      this.name = name
      this.manufacturePrice = manufacturePrice
    }
    retailPrice(marketType){
      let manufacturePrice;
      return function(marketType){
        return marketMultiplier * manufacturePrice;
      }
    }
  }

  let tennisShoe = new Item('tennis shoe', 10)
  // {name:  'tennis shoe', manufacturePrice: 10}
  tennisShoe.manufacturePrice = 4
  // {name:  'tennis shoe', manufacturePrice: 4}
```

 But we our attributes can be made truly private when using a closure.  

 ```js
 function retailPriceMaker(manufacturePrice){
   return function(marketType){
     return marketMultiplier * manufacturePrice
   }
 }

 let retailPriceForNine = retailPriceMaker(9)
 retailPriceForNine(3)
```

Because JavaScript classes are just syntactic sugar for functions, we can use closures with our classes as well.  When would we want to do this?  Let's modify our Item class a little.

```js

let ItemId = 0
class Item {
  constructor(manufacturePrice){
    this.name = name
    this.manufacturePrice = manufacturePrice
    this.id = ++ItemId;
  }
  retailPrice(marketType){
    return marketMultiplier * manufacturePrice
  }
}
```

As you see in the above code, we need to declare our `ItemId` variable outside of our class.  We do so because classes do not allow for private variables, only public methods.  Yet we want a variable the Item constructor can reference.  The problem is that `ItemId` and everything else can reference it as well.  Let's change that.  

```js

function createItem(){
  let ItemId = 0
  // return the class
  return class {
    constructor(manufacturePrice){
      this.name = name
      this.manufacturePrice = manufacturePrice
      this.id = ++ItemId;
    }

    retailPrice(marketType){
      return marketMultiplier * manufacturePrice;
    }
  }
}

const Item = createItem()
// Execute createItem and assign the returned class to equal Item.
// We only need to call createItem() one time in our codebase.

let tennisShoe = new Item(3, 3)
// {id: 1, length: 3, manufacturePrice: 3}

let diningItem = new Item(4, 3)
// {id: 2, length: 4, manufacturePrice: 3}

```

The above code may look complicated, but our only change is to wrap the code in a function called `createItem`.  The `createItem` function encapsulates all of the code declared inside of it.  This prevents the `ItemId` variable from being accessible outside of the `createItem` function.  It also privatizes the Item class, so we make sure that we return that class from our `createItem` function.  Now when we execute the `createItem`, we assign the return value of the class to equal a constant called `Item`.  So `Item` now points to our class, and we can call `new Item` to construct a new instance of this class.  Our use of closures comes into play every time we call `new Item()`.  When we construct a new instance, the constructor method references and modifies the `ItemId` variable.  Our constructor method can do so because when it's class was declared `ItemId` was accessible, and the class holds onto the variables in scope when it was declared.  So using closures allows to construct a class that has access to variables that are only available to functions that referenced the variable when the functions were declared.  Thus it allows us to better create the scope that we want for ItemId.

### Summary

In this lesson, we explored an interesting feature of JavaScript functions closures.  A closure is a feature in JavaScript such that a function holds onto the variables that it had access to when it was declared.  Closures can be used to declare functions that have specific variables always defined.  JavaScript developers also take advantage of closures to encapsulate data, as we can declare our functions in such a way that the data is only accessible from the returned function, with no way to overwrite the variables captured by the closure.    
