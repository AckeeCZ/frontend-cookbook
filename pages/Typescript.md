# Typescript

## Article [Get started with typescript in 20119](https://www.robertcooper.me/get-started-with-typescript-in-2019)

### Types

#### Static & dynamic types difference

JavaScript comes with 7 dynamic types, which are called dynamic since they are used at runtime:

* Undefined
* Null
* Boolean
* Number
* String
* Symbol
* Object

TypeScript brings along static types to the JavaScript language, and those types are evaluated at compile time (without having to run the code). **Static types are what predict the value of dynamic types** and this can help warn you of possible errors without having to run the code.

#### tuple
A tuple is an array that contains a fixed number of elements with associated types.

```ts
let myFavoriteTuple: [string, number, boolean];

myFavoriteTuple = ['chair', 20, true]; // ✅
myFavoriteTuple = [5, 20, true]; // ❌
```

#### `enum`

An enum is a way to associate names to a constant value, which can be either a number or a string.

By default, enums are assigned numbers that start at 0 and increase by 1 for each member of the enum

```ts
enum Sizes {
    Small,
    Medium,
    Large,
}

Sizes.Small; // => 0
Sizes.Medium; // => 1
Sizes.Large; //
```

```ts
enum Sizes {
    Small = 1,
    Medium,
    Large,
}

Sizes.Small; // => 1
Sizes.Medium; // => 2
Sizes.Large; // => 3
```

```ts
enum ThemeColors {
    Primary = 'primary',
    Secondary = 'secondary',
    Dark = 'dark',
    DarkSecondary = 'darkSecondary',
}
```

TBD - backward translating

#### `any`

If the type of a variable is not known and we don't want the type checker to complain at compilation time, then the type of any can be used.

It will help when starting out with TypeScript. However, it's **best to try to reduce the usage of `any` since the usefulness of TypeScript decreases when the compiler isn't aware of the types** associated with variables.

#### `void`
It is most commonly used when specifying the return value of a function that doesn't return anything.

#### Strict `null`
By default the null and undefined types are subtypes of all other types, meaning that a variable of type string can be assigned a value of null or undefined. This is often undesirable behavior and thus it's usually recommended to set the strictNullChecks compiler option in a tsconfig.json file to true. **Setting the strictNullChecks option to true makes it so that null and undefined need to be explicitly set as a type for a variable**.

### Type inference

Type inference is what the TypeScript compiler uses to automatically determine types.

```ts
let x = 10; // x is given the number type
```
TypeScript associates the x variable with a type of number.

```ts
const tweetLength = (message = 'A default tweet') => {
    return message.length;
};
```
An inferred type of string is given to the message parameter

```ts
function add(a: number, b: number) {
    return a + b;
}

const result = add(2, 4);

result.toFixed(2); // ✅
result.length; // ❌
```
Since TypeScript is told that both parameters to the add function have a type of number, it can infer that the return type will also be a number

```ts
et list = [10, 22, 4, null, 5];

list.push(6); // ✅
list.push(null); // ✅
list.push('nope'); // ❌
```
In the above example, the array is composed of both number and null types, and therefore TypeScript expects only number and null values to be a part of the array.

### Type anotation

```ts
let aBoolean: boolean = true;
let aNumber: number = 10;
let aString: string = 'woohoo';
```

#### Arrays

Can be annotated one of two ways.
```ts
let messageArray: string[] = ['hello', 'my name is fred', 'bye'];

let messageArray: Array<string> = ['hello', 'my name is fred', 'bye'];
```

#### Interface
A way to put together multiple type annotations is by using an interface.

```ts
interface Animal {
    kind: string;
    weight: number;
}

// or

// type Animal = {
//     kind: string;
//     weight: number;
// };

let dog: Animal;

dog = {
    kind: 'mammal',
    weight: 10,
}; // ✅

dog = {
    kind: true,
    weight: 10,
}; // ❌
```

**You should generally just pick either interface or type and be consistent**. However, if writing a 3rd party public API that can be used by others, use an interface type.

Sometimes it might be more appropriate to annotate a type inline instead.

```ts
let dog: {
    kind: string;
    weight: number;
};

dog = {
    kind: 'mammal',
    weight: 10,
}; // ✅

dog = {
    kind: true,
    weight: 10,
}; // ❌
```

#### Generics

Used in situations where the specific type of a variable doesn't matter, but a relationship between the types of different variables should be enforced

```ts
const fillArray = <T>(len: number, elem: T) => {
    return new Array<T>(len).fill(elem);
};

const newArray = fillArray<string>(3, 'hi'); // => ['hi', 'hi', 'hi']

newArray.push('bye'); // ✅
newArray.push(true); // ❌
```

**It is by convention** that single letters are used for generic types (e.g. T or K) but **there is nothing stopping your from using more descriptive names** for your generic types

```ts
const fillArray = <ArrayElementType>(len: number, elem: ArrayElementType) => {
    return new Array<ArrayElementType>(len).fill(elem);
};

const newArray = fillArray<string>(3, 'hi'); // => ['hi', 'hi', 'hi']

newArray.push('bye'); // ✅
newArray.push(true); // ❌
```

#### Union type

When a **type can be one of multiple types**, a union type is used by separating the different type options with a `|`.

```ts
// The `name` parameter can be either a string or null
const sayHappyBirthdayOnFacebook = (name: string | null) => {
    if (name === null) {
        console.log('Happy birthday!');
    } else {
        console.log(`Happy birthday ${name}!`);
    }
};

sayHappyBirthdayOnFacebook(null); // => "Happy birthday!"
sayHappyBirthdayOnFacebook('Jeremy');
```

#### Intersection type

An intersection type uses the `&` symbol to combine multiple types together.

```ts
type Student = {
    id: string;
    age: number;
};

type Employee = {
    companyId: string;
};

let person: Student & Employee;

person.age = 21; // ✅
person.companyId = 'SP302334'; // ✅
person.id = '10033402'; // ✅
person.name = 'Henry'; // ❌
```

#### Optional types

When a function parameter or object property is optional, use `?` to denote it.

```ts
// Optional function parameter
function callMom(message?: string) {
    if (!message) {
        console.log('Hi mom. Love you. Bye.');
    } else {
        console.log(message);
    }
}

// Interface describing an object containing an optional property
interface Person {
    name: string;
    age: number;
    favoriteColor?: string; // This property is optional
}
```

### Recommended resources

* [TypeScript Handbook (Official TypeScript docs)](https://www.typescriptlang.org/docs/handbook/basic-types.html)
* [TypeScript Deep Dive (Online TypeScript Guide)](https://basarat.gitbooks.io/typescript/content/docs/getting-started.html)
* [Understanding TypeScript's Type Annotation (Great introductory TypeScript article)](http://2ality.com/2018/04/type-notation-typescript.html)


-----

## Article [TypeScript Tutorial For Beginners: The Missing Guide (2019)](https://www.valentinog.com/blog/typescript/)

* catch serious and silly mistakes in your code
* codebase will become well structured and almost self-documenting
* improved autocompletion in your editor

Generate `tsconfig.json`

```sh
npm run tsc -- --init
```

TypeScript compiles down to “vanilla” JavaScript. The key target determines the desired JavaScript version, ES5 (or a newest release).


With **strict set to true TypeScript enforces the maximum level of type checks** on your code enabling amongst the other:

* **noImplicitAny** true: TypeScript complains when variables do not have a defined type
* **alwaysStrict** true: 
    * prevents accidental global variables, default “this” binding, and more
    * when “alwaysStrict” is set true TypeScript emits “use strict” at the very top of every JavaScript file