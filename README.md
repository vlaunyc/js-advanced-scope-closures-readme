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
  class Room {
    constructor(length, width){
      this.length = length
      this.width = width
    }
    volume(height){
      return length*width*height;
    }
  }

  let bedRoom = new Room(3*3*3)
  // {length: 3, width: 3}
  bedRoom.volume(3)
  // 27

  let livingRoom = new Room(4, 4)
  // {length: 4, width: 4}
  livingRoom.volume(4)
  // 64
```

In the code above, we use a class constructor to create different objects.  The objects share behavior and vary in their data.  

What may be surprising is that in JavaScript, we can also create **functions** that share specific capabilities but change others.  So just like we can manufacture objects as little units of work, we can also manufacture functions.

How do you manufacture a function?  It's not too difficult, we simply return a function from another function.

```js
  function volumeMaker(){
    return function(length, width, height){
      return length*width*height;
    }
  }

  typeof volumeMaker()
  // "function"
```

Let's pay careful attention to what happened in the above code.  We declared a function `volumeMaker` whose return value is a function itself.  The returned function takes arguments of length, width and height and returns their product.  

```javascript
  let volume = volumeMaker()
  // the execution of volumeMaker returns a function that takes three arguments
  // We assign this returned function to a variable called volume, and then can call the returned function by referencing volume
  volume(3, 3, 3)
  // 27
```

Ok, so why would we want to do such a thing?  Well just like we have objects that we want configured in a special way, where we know data at one time, but want to execute them at a different time, the same thing occurs with functions.  Let's explore an example.  Imagine we know the area dimensions of a room, but do not know the height.  In fact we may want to explore different heights.

```javascript
function volumeMaker(length, width){
  let length;
  let width;
  return function(height){
    return length*width*height;
  }
}

let volumeForNine = volumeMaker(3, 3)

volumeForNine(3)
// 27

volumeForNine(4)
// 36
```

So now, by invoking `volumeMaker` we return a function that has it's own unique attributes of length and width.  And we use these attributes at a later time.  So just like we earlier defined a room class that has it's length and width, here we avoided all of that code by just giving this function some knowledge of length and width.  So in JavaScript, functions can hold onto state in the same that objects can.  This makes sense, as functions are first class objects.

Let's take deeper look as to what is happening in the code. Look at the code again, below. As you see, `volumeForNine` points to our returned JavaScript function.  If you type `volumeForNine` into the console you will see that function.

```js

function volumeMaker(length, width){
  let length;
  let width;
  return function(height){
    return length*width*height;
  }
}

let volumeForNine = volumeMaker(3, 3)

volumeForNine
// ƒ (height){
//       return length*width*height;
//     }  
```
And if you type the variables length or width into the console, you will see that neither are defined in our current global scope.  Yet, somehow, when we execute this `volumeForNine` function it knows that length is 3 and width is 3.  It knows this, even though `volumeForNine` points to a function that does not have either variable defined in its execution scope.  So how does the function have values for length and width?  Placing a debugger into our code and running it in our chrome console show us.  

<!-- Display Screen Shot Here -->

Length and width are defined because of a closure.  A closure is the attribute that all JavaScript functions have: *JavaScript functions hold onto the scope that they had when they were declared*.  Let's take a look at our code again to see how we made use of a closure.  

```javascript
function volumeMaker(length, width){
  return function(height){
    return length*width*height;
  }
}

let volumeForNine = volumeMaker(3, 3)

volumeForNine(3)
// 27
```

So every time we execute the `volumeMaker` function we are declaring a new function.  That's what our `volumeMaker` function does: declare a function that it then returns.  And when that function is declared, both length and width are in scope.  So it doesn't matter that length and width are not in scope when we later execute a function.  There is a closure such that the function that `volumeMaker` returns holds onto the scope it was declared with.  This becomes a very powerful feature in JavaScript.  Closures allow us to  build functions that have their own capabilities.  

So just like we can return a function called `volumeForNine` and then see the volume returned with various heights passed through, we can construct another function called `volumeForSixteen` for say a different room.

```js
function volumeMaker(length, width){
  return function(height){
    return length*width*height;
  }
}

let volumeForNine = volumeMaker(3, 3)

let volumeForSixteen = volumeMaker(4, 4)

volumeForNine(3)
// 27

volumeForSixteen(5)
// 80
```

So as you see from above, our `volumeMaker` function lives up to its name: it returns a function that calculates the volume for various areas.  If we want to be able to calculate the volume for another area, we can simply invoke our `volumeMaker` again.

### Privacy

Thus far, we have seen how we can use closures to return functions which have various attributes that they permanently hold onto.  Closures are used for one other capability in JavaScript: privacy.  As you can see above, once we invoke `volumeMaker` we and pass through length and height, it is impossible for us to ever change these attributes.  These attributes are only even readable from inside the function, and we have defined our function in such a way that there is no other way to ever change them.  

So here, our returned functions provides some capability that JavaScript objects do not.  Encapsulation.  Remember that we can always change the data of an object.

```js
  class Room {
    constructor(length, width, height){
      this.length = length
      this.width = width
    }
    volume(height){
      return length*width*height;
    }
  }

  let room = new Room(3*3)
  // {length: 3, width: 3}
  room.length = 4
  room
  // {length: 4, width: 3}
```

 But we our attributes can be made truly private when using a closure.  

 ```js
 function volumeMaker(length, width){
   return function(height){
     return length*width*height;
   }
 }

 let volumeForNine = volumeMaker(3, 3)
 volumeForNine(3)
 // 27
 length
 // Uncaught ReferenceError: length is not defined
 height
 // Uncaught ReferenceError: height is not defined
```

Because JavaScript classes are just syntactic sugar for functions, we can use closures with our classes as well.  When would we want to do this?  Let's modify our Room class a little.

```js

let roomId = 0
class Room {
  constructor(length, width){
    this.length = length
    this.width = width
    this.id = ++roomId;
  }
  volume(height){
    return length*width*height;
  }
}
```

As you see in the above code, we need to declare our `roomId` variable outside of our class.  We do so because classes do not allow for private variables, only public methods.  Yet we want a variable the Room constructor can reference.  The problem is that `roomId` and everything else can reference it as well.  Let's change that.  

```js
function createRoom(){
  let roomId = 0
  // return the class
  return class {
    constructor(length, width){
      this.length = length
      this.width = width
      this.id = ++roomId;
    }
    volume(height){
      return length*width*height;
    }
  }
}

const Room = createRoom()
// Execute createRoom and assign the returned class to equal Room.
// We only need to call createRoom() one time in our codebase.

let bedroom = new Room(3, 3)
// {id: 1, length: 3, width: 3}

let diningRoom = new Room(4, 3)
// {id: 2, length: 4, width: 3}
```

The above code may look complicated, but our only change is to wrap the code in a function called `createRoom`.  The `createRoom` function encapsulates all of the code declared inside of it.  This prevents the `roomId` variable from being accessible outside of the `createRoom` function.  It also privatizes the Room class, so we make sure that we return that class from our `createRoom` function.  Now when we execute the `createRoom`, we assign the return value of the class to equal a constant called `Room`.  So `Room` now points to our class, and we can call `new Room` to construct a new instance of this class.  Our use of closures comes into play every time we call `new Room()`.  When we construct a new instance, the constructor method references and modifies the `roomId` variable.  Our constructor method can do so because when it'ss class was declared `roomId` was accessible, and the class holds onto the variables in scope when it was declared.  So using closures allows to construct a class that has access to variables that are only available to functions that referenced the variable when the functions were declared.  Thus it allows us to better create the scope that we want for roomId.

### Summary

In this lesson, we explored an interesting feature of JavaScript functions closures.  A closure is a feature in JavaScript such that a function holds onto the variables that it had access to when it was declared.  Closures can be used to declare functions that have specific variables always defined.  JavaScript developers also take advantage of closures to encapsulate data, as we can declare our functions in such a way that the data is only accessible from the returned function, with no way to overwrite the variables captured by the closure.    
