# Silly programming language

# Values

Expressions produce values.

# Types

A type is a special value that classifies other values.

# Bindings

A value can be bound to a name.

## Compile time bindings

A binding created with `const` is evaluated at compile time:

```
const pi = 3.14;
```

## Run time bindings

A binding created with `let` is evaluated at run time. By default, bindings are
immutable:

```
let x = 10;
```

If a binding needs to be reassigned later, it must be declared as mutable:

```
let mut counter = 0;
counter = 1;
```

## Variable types

A variable can be annotated with a type explicitly:

```
let foo: U64 = 0;
```

The compiler can also infer the type of the value being assigned to a variable:

```
let foo = True; // True is Bool, therefore foo is Bool
```

# Unit

Unit is an empty product type. It has only one possible value and doesn't hold
any information:

```
const foo = ();
const foo: ();

let foo = ();
let foo: ();
```

# Product types

A product type holds multiple values. To instantiate a product of some product
type, supply the product type with an a compatible anonymous product value.

```
// anon
let foo = (F64, F64) (0.0, 0.0);
foo.0;

// const
const Point = (F64, F64);
let foo = Point (0.0, 0.0);
foo.0;

// const + named fields
const Point = (x: F64, y: F64);
let foo = Point (x: 0.0, y: 0.0);
foo.x;
```

# Sum type

A sum type holds a value that could take on different types. To instantiate a
sum of some sum type, supply the sum type with a compatible anonymous value.

```
// anon
let foo = F64 | () 0.0;
// can't create sum types with unnamed variants containing duplicate types
// let foo = F64 | F64;

// const
const Option = U64 | ();
let foo = Option 0;

// const + named variants
const Option = Some U64 | None; // named variant without a type parameter is unit
let foo = Option Some 0;
let foo = Option None;
```

# Destructuring

## Product type

We can specify the depth:

```
const Foo = (U64);

let foo = Foo (0);
let Foo bar = foo;   // foo: (U64) = (0)
let Foo (bar) = foo; // foo: U64 = 0
```

Or destructure one type lower with `_`:

```
const Foo = (U64);
const Bar = (Foo);

let bar = Bar (Foo (0));
let _ (foo) = bar; // foo: Foo = (0)
```

## Primitive type

Destructuring a value of a primitive type results in the value of the same type:

```
let U64 foo = 0;
let U64 U64 foo = 0;
let () foo = ();
```

# Pattern matching and type narrowing for sum types

For constant types, whose fields are not named:

```
const Option = U64 | ();

let foo = Option 69;

match foo {
    let ()     -> (),
    let _  foo -> ( // foo is U64),
};

const Num = F64 | i64 | U64;

match foo {
    let F64 foo -> ( // foo is F64 ),
    let _   bar -> ( // bar is i64 | U64 ),
}

// wildcard
match foo {
    let _ -> (),
};
```

For constant types, whose fields are named:

```
const Option = Some U64 | None;

let foo = Option Some 0;

match foo {
    let Some foo -> ( // foo is U64 ),
    let None     -> (),
};
```

# Function type

A function type constists of a parameter type and a result type:

```
U64 -> U64;
(U64, U64) -> (U64, U64);

const Option = Some U64 | None;
Option -> bool;

const Point = (x: U64, y: U64);
Point -> U64;
```

A function can be instantiated by supplying a parameter binding, that is used
to desctructure the parameter provided to the function, and a computation:

```
const add: (U64, U64) -> U64 = (a, b) -> {
    a + b
};
let foo = add (1, 2);

const Point = (x: F64, y: F64);
let dist_to_origin: Point -> F64 = (x, y) -> {
    sqrt (pow (x, 2.0) + pow (y, 2.0))
};
let dist = dist_to_origin (Point (0.0, 0.0))
```

A generic type can be defined as a funciton that takes a type and returns a type:

```
const Pair: (Type, Type) -> Type = (T, U) -> {
    (T, U)
};
```

A generic function is defined as a function that takes a type and returns a function:

```
const identity: T: Type -> (T -> T) = T -> {
    x -> { x }
}
```

And the final example of a mapping function for an optional type:

```
const Option: Type -> Type = T -> {
    Some T | None
}

const map: (T: Type, U: Type) -> (T -> U, Option T) -> Option U = (T, U) -> {
    (func, option) -> {
        match option {
            let Some val -> Option U Some (func val),
            let None     -> Option U None,
        }
    }
}
```
