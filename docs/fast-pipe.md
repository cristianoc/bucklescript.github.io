---
title: Fast Pipe
---

BuckleScript and Reason have a special `|.` syntax for dealing with various situations. This operator has many uses.

## Pipelining

`a |. foo b` is equal to `foo a b`. The operator takes the item on the left and put it as the first argument of the item on the right. Great for building pipelines of data processing:

```ocaml
a
|. foo b
|. bar
```

```reason
a
|. foo(b)
|. bar
```

is equal to

```ocaml
bar(foo a b)
```

```reason
bar(foo(a, b))
```

## JS Method Chaining

JavaScript's APIs are often attached to objects, and often chainable, like so:

```js
const result = [1, 2, 3].map(a => a + 1).filter(a => a % 2 === 0);

asyncRequest().setWaitDuration(4000).send();
```

Assuming we don't need the chaining behavior above, we'd bind to each case this using `bs.send` from the previous section:

```ocaml
external map : 'a array -> ('a -> 'b) -> 'b array = "" [@@bs.send]
external filter : 'a array -> ('a -> 'b) -> 'b array = "" [@@bs.send]

type request
external asyncRequest: unit -> request = ""
external setWaitDuration: request -> int -> request = "" [@@bs.send]
external send: request -> unit = "" [@@bs.send]
```

```reason
[@bs.send] external map : (array('a), 'a => 'b) => array('b) = "";
[@bs.send] external filter : (array('a), 'a => 'b) => array('b) = "";

type request;
external asyncRequest: unit => request = "";
[@bs.send] external setWaitDuration: (request, int) => request = "";
[@bs.send] external send: request => unit = "";
```

You'd use them like this:

```ocaml
let result = filter (map [|1; 2; 3|] (fun a -> a + 1)) (fun a -> a mod 2 = 0)

let () = send(setWaitDuration (asyncRequest()) 4000)
```

```reason
let result = filter(map([|1, 2, 3|], a => a + 1), a => a mod 2 == 0);

send(setWaitDuration(asyncRequest(), 4000));
```

This looks much worse than the JS counterpart! Now we need to read the actual logic "inside-out". We also cannot use the `|>` operator here, since the object comes _first_ in the binding. But `|.` works!

```ocaml
let result = [|1; 2; 3|] |. map(fun a -> a + 1) |. filter(fun a -> a mod 2 == 0)

let () = asyncRequest () |. setWaitDuration 400 |. send ()
```

```reason
let result = [|1, 2, 3|] |. map(a => a + 1) |. filter(a => a mod 2 === 0);

asyncRequest() |. setWaitDuration(400) |. send();
```

## Pipe Into Variants

This works:

```ocaml
let result = name |. preprocess |. Some
```

```reason
let result = name |. preprocess |. Some
```

We turn this into:

```ocaml
let result = Some(preprocess(name))
```

```reason
let result = Some(preprocess(name))
```

## Get Multiple Results

The fast pipe operator can also be used on a tuple:

```ocaml
let (left, middle, right) = myData |. (getLeft, getMiddle, getRight)
```

```reason
let (left, middle, right) = myData |. (getLeft, getMiddle, getRight)
```

This turns into:

```ocaml
let left = getLeft myData
let middle = getMiddle myData
let right = getRight myData
```

```reason
let left = getLeft(myData);
let middle = getMiddle(myData);
let right = getRight(myData);
```

Convenient for certain kind of operations.

## Chaining Imperative APIs

**Unreleased**, as we're still waiting for feedback on this one; if you'd like the feature, please [upvote here](https://github.com/BuckleScript/bucklescript/issues/2748)!

Mutative, imperative APIs usually look like this:

```ocaml
let mySet = Set.make()
Set.add mySet 1
Set.add mySet 2
Set.remove mySet 1
```

```reason
let mySet = Set.make();
Set.add(mySet, 1);
Set.add(mySet, 2);
Set.remove(mySet, 1);
```

Unlike their immutable, functional counterpart, these operations cannot be chained with `|>` (their return type is `unit`). But fast pipe works!

```ocaml
let mySet = Set.make()
let () = mySet |. [
  Set.add 1;
  Set.add 2;
  Set.remove 1;
]
```

```reason
let mySet = Set.make();
mySet |. [
  Set.add(1),
  Set.add(2),
  Set.remove(1),
];
```
