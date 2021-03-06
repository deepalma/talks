# Algebraic Data Types

---

# Self Introduction

- Giulio Canti (@GiulioCanti)
- Degree in mathematics
- Writing and teaching TypeScript and functional programming for the past 2 years

---

# Agenda

1. **Introduction**
2. Product types
3. Sum types
4. Functional error handling
5. Slaying a UI Antipattern

---

# Introduction

A good first step while building a new application is to define its **domain model**.

TypeScript offers many tools to help you in this task.

Types are a **design tool**.

**Algebraic Data Types** are one of these tools.

---

# What is an ADT?

> In computer programming, especially functional programming and type theory, an algebraic data type is a kind of composite type, i.e., **a type formed by combining other types**.

Two common classes of algebraic types are

- **product types** (tuples and records)
- **sum types** (tagged unions).

---

# Agenda

1. Introduction
2. **Product types**
3. Sum types
4. Functional error handling
5. Slaying a UI Antipattern

---

# Product types

> A product type is a collection of types T<sub>i</sub> indexed by a set I

T<sub>x</sub>, T<sub>y</sub>, T<sub>z</sub>, ...

x, y, z, ... ∈ I

---

Two common members of this family are `n`-tuples, where `I` is a non empty interval of natural numbers

```ts
￼type Tuple1 = [string] // I = [0]
type Tuple2 = [string, number] // I = [0, 1]
type Tuple3 = [string, number, boolean] // I = [0, 1, 2]
```

Accessing by index

```ts
type Fst = Tuple2[0] // string
type Snd = Tuple2[1] // number
```

---

and records, where `I` is a set of labels.

```ts
// I = {"name", "age"}
interface Person {
  name: string
  age: number
}
```

Accessing by label

```ts
type Name = Person['name'] // string
type Age = Person['age'] // number
```

---

`Tuple2` and `Person` are _isomorphic_.

An **isomorphism** between two types `S` and `A` is a pair of functions `f: S -> A` and `g: A -> S` such that

```
g . f = f . g = identity
```

---

# Proof

```ts
// f: Tuple2 -> Person
const f = ([name, age]: Tuple2): Person => ({ name, age })
```

```ts
// g: Person -> Tuple2
const g = ({ name, age }: Person): Tuple2 => [name, age]
```

---

Such a isomorphism is obvious if `Person` is implemented as a class

```ts
class Person {
  constructor(public name: string, public age: number) {}
}
```

`constructor` implements the `f` function.

---

# Why "product types"?

If we write `C(A)` for the number of inhabitants of the type `A` (aka **cardinality**) then the following equality holds

```ts
C([A, B]) = C(A) * C(B)
```

> the cardinality of the product is the product of cardinalities

---

# Example

```ts
￼type Hour = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
type Period = 'AM' | 'PM'
type Clock = [Hour, Period]
```

The type `Clock` has `12 * 2 = 24` inhabitants.

---

# When you can use a product type?

Whenever its components are **independent**.

```ts
type Tuple2 = [string, number]
```

`string` and `number` are independent.

---

# Two notable product types

- `NonEmptyArray`
- `Zipper`

---

# `NonEmptyArray`

Data structure which represents non-empty arrays

```ts
//   type parameter ↓
class NonEmptyArray<A> {
  constructor(readonly head: A, readonly tail: Array<A>) {}
}
```

`NonEmptyArray` is a **polymorphic** product type.

---

# `Zipper`

Let's say we want to model the following data structure

> a non-empty list of items, one of which is the current selection

---

# A naive model

```ts
interface Selection<A> {
  items: Array<A>
  current: number
}
```

Defects

- the list can be empty
- the current index can be out of range

---

# A naive model

```ts
interface Selection<A> {
  items: NonEmptyArray<A>
  current: number
}
```

Defects

- the current index can be out of range

---

# A correct-by-construction model

```ts
interface Zipper<A> {
  prev: Array<A>
  current: A
  next: Array<A>
}
```

"Make illegal states unrepresentable"

---

# Agenda

1. Introduction
2. Product types
3. **Sum types**
4. Functional error handling
5. Slaying a UI Antipattern

---

# Sum types

Product types correspond to **cartesian products** of sets.

Sum types correspond to **disjoint unions** of sets.

---

# Example: redux actions

```ts
type Action =
  | {
      type: 'ADD_TODO' // string literal type
      text: string
    }
  | {
      type: 'UPDATE_TODO' // string literal type
      id: number
      text: string
      completed: boolean
    }
  | {
      type: 'DELETE_TODO' // string literal type
      id: number
    }
```

The `type` field is the _tag_.

---

# Example: redux actions

Constructors

```ts
const add = (text: string): Action => ({
  type: 'ADD_TODO',
  text
})
const update = (
  id: number,
  text: string,
  completed: boolean
): Action => ({
  type: 'UPDATE_TODO',
  id,
  text,
  completed
})
const del = (id: number): Action => ({
  type: 'DELETE_TODO',
  id
})
```

---

# Example: linked lists

```ts
//        ↓ type parameter
￼type List<A> =
  | { type: 'Nil' }
  | {
      type: 'Cons',
      head: A,
      tail: List<A>
      //    ↑ recursion
    }
```

`List<A>` is a polymorphic **recursive** sum type.

---

# Pattern matching

```ts
type State = Array<Todo>

//                  please note the return type ↓
function reducer(state: State, action: Action): State {
  //      ↓ switch on the tag field
  switch (action.type) {
    case 'ADD_TODO' :
      return ...
    case 'UPDATE_TODO' :
      return ...
    case 'DELETE_TODO' :
      // here `action` has type
      // `{ type: 'DELETE_TODO', id: number }`
      const id /*: number */ = action.id
      return ...
  }
  // here `action` has type `never`
}
```

---

# Exhaustivity checking

```ts
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_TODO' :
      return ...
    case 'UPDATE_TODO' :
      return ...
    //  missing case clause
  }
}
```

TypeScript raises the following error

```ts
[ts] Function lacks ending return statement
and return type does not include 'undefined'
```

---

# Exhaustivity checking

Be careful of adding a `default` clause, since you're effectively disabling this feature.

---

# Why "sum types"?

The following equality holds

```ts
C(A | B) = C(A) + C(B)
```

> the cardinality of the sum is the sum of cardinalities

---

# Example: the `Option` type

```ts
type Option<A> =
  | { type: 'None' }
  | {
      type: 'Some'
      value: A
    }
```

`Option<boolean>` has `1 + 2 = 3` inhabitants.

---

# When should you use a sum type?

When its components would be **dependent** if implemented as a product type.

---

# Example: component props

```ts
interface Props {
  editable: boolean
  onChange?: (text: string) => void
}

class Textbox extends React.Component<Props> {
  render() {
    if (this.editable) {
      // onChange: (text: string) => void | undefined
      const onChange = this.onChange
      ...
    }
  }
}
```

---

# Example: component props

`Props` is modelled as a product type but `onChange` **depends** on `editable`.

A sum type is a better choice

```ts
type Props =
  | {
      type: 'READONLY'
    }
  | {
      type: 'EDITABLE'
      onChange: (text: string) => void
    }
```

---

# Example: node callbacks

```ts
readFile(
  path: string,
  callback: (err?: Error, data?: string) => void
): void
```

The result is modelled as a product type

```ts
type CallbackArgs = [Error | undefined, string | undefined]
```

however, its components are **dependent**: you get either an error or a string.

---

# Example: node callbacks

| `Error?`    | `string?`   | Legal |
| ----------- | ----------- | ----- |
| `Error`     | `undefined` | ✓     |
| `undefined` | `string`    | ✓     |
| `Error`     | `string`    | ✘     |
| `undefined` | `undefined` | ✘     |

A sum type would be a better choice, but which one?

---

# Agenda

1. Introduction
2. Product types
3. Sum types
4. **Functional error handling**
5. Slaying a UI Antipattern

---

# Functional error handling

- `Option`
- `Either`

---

# `Option`

`Option<A>` represents the effect of a computation that can fail

```ts
type Option<A> =
  | { type: 'None' }
  | {
      type: 'Some'
      value: A
    }
```

- replaces `throws`
- can represent optional values

---

# `Option`

Constructors and pattern matching

```ts
const none: Option<never> = { type: 'None' }

const some = <A>(a: A): Option<A> => ({
  type: 'Some',
  value: a
})

const match = <A, O>(
    fa: Option<A>,
    whenNone: O,
    whenSome: (a: A) => O
  ): O => {
  switch (fa.type) {
    case 'None' :
      return whenNone
    case 'Some' :
      return whenSome(fa.value)
  }
}
```

---

# `Option`

From...

```ts
const head = <A>(as: Array<A>): A => {
  if (as.length === 0) {
    throw new Error()
  } else {
    return as[0]
  }
}

try {
  return String(head([]))
} catch (e) {
  return 'Empty array'
}
```

---

# `Option`

...to

```ts
const head = <A>(as: Array<A>): Option<A> => {
  if (as.length === 0) {
    return none
  } else {
    return some(as[0])
  }
}

match(head([]), 'Empty array', a => String(a))
```

---

# Either

A common use of `Either` is as an alternative to `Option` for dealing with possibly failing computations.

In this usage, `None` is replaced with a `Left` which can contain useful information. `Right` takes the place of `Some`.

---

# Either

```ts
type Either<L, R> =
  | {
      type: 'Left'
      left: L
    }
  | {
      type: 'Right'
      right: R
    }
```

---

# Either

Constructors and pattern matching

```ts
const left = <L, R>(left: L): Either<L, R> =>
  ({ type: 'Left', left })

const right = <L, R>(right: R): Either<L, R> =>
  ({ type: 'Right', right })

const match = <L, R, O>(
    fa: Either<L, R>,
    whenLeft: (left: L) => O,
    whenRight: (right: R) => O
  ): O => {
  switch (fa.type) {
    case 'Left' :
      return whenLeft(fa.left)
    case 'Right' :
      return whenRight(fa.right)
  }
}
```

---

# Example: node callbacks

```ts
readFile(
  path: string,
  callback: (err?: Error, data?: string) => void
): void
```

versus

```ts
readFile(
  path: string,
  callback: (result: Either<Error, string>) => void
): void
```

---

# Agenda

1. Introduction
2. Product types
3. Sum types
4. Functional error handling
5. **Slaying a UI Antipattern**

---

# The problem

The problem is a very common one. You are loading a list of things but instead of showing a loading indicator you just see zero items.

In JavaScript your data model may look like this

```ts
{ loading: true, items: [] }
```

But of course it's easy to forget to check the loading flag.

---

# The problem

What about using `null` in order to represent the "not loaded" case?

```ts
interface Model {
  items: Array<Item> | null
}
```

Long experience will have taught you that setting a property to `null` may be correct, but it's just asking for runtime exceptions

Fortunately, this concern evaporates when using TypeScript.

---

# The problem

But we now know a "standard" way to represent optional values: `Option`

```ts
interface Model {
  items: Option<Array<Item>>
}
```

However we can go even further and be way more sophisticated.

---

# The solution

HTTP requests have one of four states

- we haven't asked yet
- we've asked, but we haven't got a response yet
- we got a response, but it was an error
- we got a response, and it was the data we wanted

---

# The solution

With TypeScript we can easily define a type that represents these four states

```ts
type RemoteData<E, D> =
  | { type: 'NotAsked' }
  | { type: 'Loading' }
  | { type: 'Failure'; error: E }
  | { type: 'Success'; data: D }

interface Model {
  items: RemoteData<HttpError, Array<Item>>
}
```

---

# The solution

> The nice thing about this data model is, the type checker will now force you to write the correct UI code. It will keep track of the possibility of "things not loaded" and errors, and force you to handle them all in the UI. - _Kris Jenkins_

---

# The solution

TypeScript will force you to handle all cases

```tsx
const SomeView = ({ items }: Model): ReactElement<any> => {
  switch (items.type) {
    case 'NotAsked' :
      return <div>Please press the button to load the items</div>
    case 'Loading' :
      return <div>Loading items...</div>
    case 'Failure' :
      return <div>An error has occurred { items.error }</div>
    case 'Success' :
      return <div>{ items.data.map(item => { ... }) }</div>
  }
}
```

---

# The solution

Or, using a more functional style, let's define a `match` function that can be re-utilised in more use cases

```ts
const match = <E, D, O>(
  rd: RemoteData<E, D>,
  notAsked: () => O,
  loading: () => O,
  failure: (error: E) => O,
  success: (data: D) => O
): O {
  switch (rd.type) {
    case 'NotAsked' :
      return notAsked()
    case 'Loading' :
      return loading()
    case 'Failure' :
      return failure(rd.error)
    case 'Success' :
      return success(rd.data)
  }
}
```

---

# The solution

Now `SomeView` can be defined as

```tsx
const SomeView = ({ items }: Model): ReactElement<any> => {
  return match(
    items,
    () => <div>Please press the button to load the items</div>,
    () => <div>Loading items...</div>,
    error => <div>An error has occurred {error}</div>,
    data => <div>{data.map(item => { ... })}</div>
  )
}
```

---

# Thanks

- Slides https://github.com/gcanti/talks/tree/master/adt

### Functional programming

- Free book (italian) https://github.com/gcanti/functional-programming
- TypeScript library https://github.com/gcanti/fp-ts
