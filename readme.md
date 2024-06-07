<img alt="TypeScript Result logo" width="84px" src="./assets/typescript-result-logo.svg" />

# TypeScript Result

[![NPM](https://img.shields.io/npm/v/typescript-result.svg)](https://www.npmjs.com/package/typescript-result)
[![TYPESCRIPT](https://img.shields.io/badge/%3C%2F%3E-typescript-blue)](http://www.typescriptlang.org/)
[![BUNDLEPHOBIA](https://badgen.net/bundlephobia/minzip/typescript-result)](https://bundlephobia.com/result?p=typescript-result)
[![Weekly downloads](https://badgen.net/npm/dw/typescript-result)](https://badgen.net/npm/dw/typescript-result)

A Result type inspired by Rust and Kotlin that leverages TypeScript's powerful type system to simplify error handling and make your code more readable and maintainable with full type safety.

## Getting started

### Installation

Install using your favorite package manager:

```bash
npm install typescript-result
```

### Requirements

#### Typescript

Technically Typescript with version `4.8.0` or higher should work, but we recommend using version >= `5` when possible.

Also it is important that you have `strict` or `strictNullChecks` enabled in your `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

#### Node

Tested with Node.js version `16` and higher.

### Example

```typescript
import { Result } from "typescript-result";

// Define errors like you would normally do
class DivisionByZeroError extends Error {
  readonly type = "division-by-zero";
}

// Define a function that can either return a value or an error
function divide(a: number, b: number) {
  if (b === 0) {
    return Result.error(new DivisionByZeroError(`Cannot divide ${a} by zero`));
  }

  return Result.ok(a / b);
}

const result = divide(10, 2);
if (result.isOk()) {
  // TS infers that result.value is a number
  console.log(result.value); // 5
} else if (result.isError()) {
  // TS infers that result.error is a `DivisionByZeroError`
  console.error(result.error);
}
```

## Why should you use a result type?

### Errors as values

The Result type is a product of the ‘error-as-value’ movement which in turn has its roots in function programming. When throwing exceptions, all errors are treated equally, and behave different compared to the normal flow of the program. Instead, we like to make a distinction between expected errors and unexpected errors, and make the expected errors part of the normal flow of the program. By explicitely defining that a piece of code can either fail or succeed using the Result type, we can leverage TypeScript's powerful type system to keep track for us everything that can go wrong in our code, and let it correct us when we overlooked certain scenarios by performing exhaustive checks. This makes our code more type-safe, easier to maintain, and more transparent.

### Ergonomic error handling

The goal is to keep the effort in using this library as light as possible, with a relatively small API surface. We don't want to introduce a whole new programming model where you would have to learn a ton of new concepts. Instead, we want to build on top of the existing features and best practices of the language, and provide a simple and intuitive API that is easy to understand and use. It also should be easy to incrementally adopt with existing codebases.

## Why should you use this library?

There are already a few quality libraries out there that provide a Result type or similar for TypeScript. We believe that there are two reasons why you should consider using this library.

### Async support

Result instances that are wrapped in a Promise can be painful to work with, because you would have to `await` every async operation before you can _chain_ next operations (like 'map', 'fold', etc.). To solve this and to make your code more ergonomic we provide an `AsyncResult` that is essentially a regular Promise that contains a `Result` type, along with a couple of methods, to make it easier to chain operations without having to assign the intermediate results to a variable or having to use `await` for each async operation.

So instead of writing:

```typescript
const firstAsyncResult = await someAsyncFunction1();
if (firstAsyncResult.isOk()) {
  const secondAsyncResult = await someAsyncFunction2(firstAsyncResult.value);
  if (secondAsyncResult.isOk()) {
    const thirdAsyncResult = await someAsyncFunction3(secondAsyncResult.value);
    if (thirdAsyncResult.isOk()) {
      // do something
    } else {
      // handle error
    }
  } else {
    // handle error
  }
} else {
  // handle error
}
```

You can write:

```typescript
const result = await Result.fromAsync(someAsyncFunction1())
  .map((value) => someAsyncFunction2(value))
  .map((value) => someAsyncFunction3(value))
  .fold(
    (value) => {
      // do something on success
    },
    (error) => {
      // handle error
    }
  );
```

### _Full_ type safety without a lot of boilerplate

This library is able to track all possible outcomes simply by using type inference. Of course, there are edge cases, but most of the time all you have to do is to simply return `Result.ok()` or `Result.error()`, and the library will do the rest for you.
In the example below, Typescript will complain that not all code paths return a value. Rightfully so, because we forgot to implement the case where there is not enough stock:

```typescript
class NotEnoughStockError extends Error {
  readonly type = "not-enough-stock";
}

class InsufficientBalanceError extends Error {
  readonly type = "insufficient-balance";
}

function order(basket: Basket, stock: Stock, account: Account) {
  if (basket.getTotalPrice() > account.balance) {
    return Result.error(new InsufficientBalanceError());
  }

  if (!stock.hasEnoughStock(basket.getProducts())) {
    return Result.error(new NotEnoughStockError());
  }

  const order: Order = { /skipped for brevity */ }

  return Result.ok(order);
}

function handleOrder(products: Product[], userId: number) {
  /skipped for brevity  */

  return order(basket, stock, account).fold(
    () => ({
      status: 200,
      body: "Order placed successfully",
    }),
    (error) => { // TS-Error: Not all code paths return a value
      switch(error.type) {
        case "insufficient-balance":
          return {
            status: 400,
            body: "Insufficient balance",
          }
      }
    }
  );
}
```

# API Reference

## Table of contents

- [Result](#result)
  - Properties
    - [isResult](#isresult)
    - [value](#value)
    - [error](#error)
  - Instance methods
    - [isOk()](#isok)
    - [isError()](#iserror)
    - [errorOrNull()](#errorornull)
    - [getOrNull()](#getornull)
    - [getOrDefault(defaultValue)](#getordefaultdefaultvalue)
    - [getOrElse(onFailure)](#getorelseonfailure)
    - [getOrThrow()](#getorthrow)
    - [fold(onSuccess, onFailure)](#foldonsuccess-onfailure)
    - [onFailure(action)](#onfailureaction)
    - [onSuccess(action)](#onsuccessaction)
    - [map(transform)](#maptransform)
    - [mapCatching(transform)](#mapcatchingtransform)
    - [recover(onFailure)](#recoveronfailure)
    - [recoverCatching(onFailure)](#recovercatchingonfailure)
  - Static methods
    - [Result.ok(value)](#resultokvalue)
    - [Result.error(error)](#resulterrorerror)
    - [Result.isResult(possibleResult)](#resultisresultpossibleresult)
    - [Result.isAsyncResult(possibleAsyncResult)](#resultisasyncresultpossibleasyncresult)
    - [Result.all(items)](#resultallitems)
    - [Result.allCatching(items)](#resultallcatchingitems)
    - [Result.wrap(fn)](#resultwrapfn)
    - [Result.try(fn, [transform])](#resulttryfn-transform)
    - [Result.fromAsync(promise)](#resultfromasyncpromise)
    - [Result.fromAsyncCatching(promise)](#resultfromasynccatchingpromise)
    - [Result.assertOk(result)](#resultassertokresult)
    - [Result.assertError(result)](#resultasserterrorresult)
- [AsyncResult](#asyncresult)
  - Properties
    - [isAsyncResult](#isasyncresult)
  - Instance methods
    - [errorOrNull()](#errorornull-1)
    - [getOrNull()](#getornull-1)
    - [getOrDefault(defaultValue)](#getordefaultdefaultvalue-1)
    - [getOrElse(onFailure)](#getorelseonfailure-1)
    - [getOrThrow()](#getorthrow-1)
    - [fold(onSuccess, onFailure)](#foldonsuccess-onfailure-1)
    - [onFailure(action)](#onfailureaction-1)
    - [onSuccess(action)](#onsuccessaction-1)
    - [map(transformFn)](#maptransformfn-1)
    - [mapCatching(transformFn)](#mapcatchingtransformfn-1)
    - [recover(onFailure)](#recoveronfailure-1)
    - [recoverCatching(onFailure)](#recovercatchingonfailure-1)

## Result

Represents the outcome of an operation that can either succeed or fail.

```ts
class Result<Value, Error> {}
```

### isResult

Utility getter that checks if the current instance is a `Result`.

### value

Retrieves the encapsulated value of the result when the result is successful.

> [!NOTE]
> You can use [`Result.isOk()`](#isok) to narrow down the type to a successful result.

#### Example
obtaining the value of a result, without checking if it's successful
```ts
declare const result: Result<number, Error>;

result.value; // number | undefined
```

#### Example
obtaining the value of a result, after checking for success
```ts
declare const result: Result<number, Error>;

if (result.isOk()) {
  result.value; // number
}
```

### error

Retrieves the encapsulated error of the result when the result represents a failure.

> [!NOTE]
> You can use [`Result.isError()`](#iserror) to narrow down the type to a failed result.

#### Example
obtaining the value of a result, without checking if it's a failure

```ts
declare const result: Result<number, Error>;

result.error; // Error | undefined
```

#### Example
obtaining the error of a result, after checking for failure
```ts
declare const result: Result<number, Error>;

if (result.isError()) {
  result.error; // Error
}
```

### isOk()

Type guard that checks whether the result is successful.

**returns** `true` if the result is successful, otherwise `false`.

#### Example
checking if a result is successful
```ts
declare const result: Result<number, Error>;

if (result.isOk()) {
  result.value; // number
}
```

### isError()

Type guard that checks whether the result is successful.

**returns** `true` if the result represents a failure, otherwise `false`.

#### Example
checking if a result represents a failure
```ts
declare const result: Result<number, Error>;

if (result.isError()) {
  result.error; // Error
}
```

### errorOrNull()

**returns** the encapsulated error if the result is a failure, otherwise `null`.

### getOrNull()

**returns** the encapsulated value if the result is successful, otherwise `null`.

### getOrDefault(defaultValue)

Retrieves the value of the result, or a default value if the result is a failure.

#### Parameters

- `defaultValue` The value to return if the result is a failure.

**returns** The encapsulated value if the result is successful, otherwise the default value.

#### Example
obtaining the value of a result, or a default value
```ts
declare const result: Result<number, Error>;

const value = result.getOrDefault(0); // number
```

#### Example
using a different type for the default value
```ts
declare const result: Result<number, Error>;

const value = result.getOrDefault("default"); // number | string
```

### getOrElse(onFailure)

Retrieves the value of the result, or transforms the error using the `onFailure` callback into a value.

#### Parameters

- `onFailure` callback function which allows you to transform the error into a value. The callback can be async as well.

**returns** either the value if the result is successful, or the transformed error.

#### Example
transforming the error into a value
```ts
declare const result: Result<number, Error>;

const value = result.getOrElse((error) => 0); // number
```

#### Example
using an async callback
```ts
const value = await result.getOrElse(async (error) => 0); // Promise<number>
```

### getOrThrow()

Retrieves the value of the result, or throws an error if the result is a failure.

**returns** The value if the result is successful.

**throws** an error if the result is a failure.

> [!IMPORTANT]
> The error thrown will have the original error set as the `cause` property.

#### Example
obtaining the value of a result, or throwing an error
```ts
declare const result: Result<number, Error>;

const value = result.getOrThrow(); // number
```

### fold(onSuccess, onFailure)

Returns the result of the `onSuccess` callback when the result represents success or
the result of the `onFailure` callback when the result represents a failure.

> [!NOTE]
> Any exceptions that might be thrown inside the callbacks are not caught, so it is your responsibility
> to handle these exceptions

#### Parameters

- `onSuccess` callback function to run when the result is successful. The callback can be async as well.
- `onFailure` callback function to run when the result is a failure. The callback can be async as well.

**returns** * the result of the callback that was executed.

#### Example
folding a result to a response-like object

```ts
declare const result: Result<User, NotFoundError | UserDeactivatedError>;

const response = result.fold(
  (user) => ({ status: 200, body: user }),
  (error) => {
    switch (error.type) {
      case "not-found":
        return { status: 404, body: "User not found" };
      case "user-deactivated":
        return { status: 403, body: "User is deactivated" };
    }
  }
);
```

### onFailure(action)

Calls the `action` callback when the result represents a failure. It is meant to be used for
side-effects and the operation does not modify the result itself.

#### Parameters

- `action` callback function to run when the result is a failure. The callback can be async as well.

**returns** the original instance of the result.

> [!NOTE]
> Any exceptions that might be thrown inside the `action` callback are not caught, so it is your responsibility
> to handle these exceptions

#### Example
adding logging between operations
```ts
declare const result: Result<number, Error>;

result
  .onFailure((error) => console.error("I'm failing!", error))
  .map((value) => value 2); // proceed with other operations
```

### onSuccess(action)

Calls the `action` callback when the result represents a success. It is meant to be used for
side-effects and the operation does not modify the result itself.

#### Parameters

- `action` callback function to run when the result is successful. The callback can be async as well.

**returns** * the original instance of the result. If the callback is async, it returns a new [`AsyncResult`](#asyncresult) instance.

> [!NOTE]
> Any exceptions that might be thrown inside the `action` callback are not caught, so it is your responsibility
> to handle these exceptions

#### Example
adding logging between operations
```ts
declare const result: Result<number, Error>;

result
  .onSuccess((value) => console.log("I'm a success!", value))
  .map((value) => value 2); // proceed with other operations
```

#### Example
using an async callback
```ts
declare const result: Result<number, Error>;

const asyncResult = await result.onSuccess(async (value) => someAsyncOperation(value));
```

### map(transformFn)

Transforms the value of a successful result using the `transform` callback.
The `transform` callback can also return other `Result` or [`AsyncResult`](#asyncresult) instances,
which will be returned as-is (the `Error` types will be merged).
The operation will be ignored if the result represents a failure.

#### Parameters

- `transformFn` callback function to transform the value of the result. The callback can be async as well.

**returns** * a new [`Result`](#result) instance with the transformed value, or a new [`AsyncResult`](#asyncresult) instance
if the `transformFn` function is async.

> [!NOTE]
> Any exceptions that might be thrown inside the `transformFn` callback are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`Result.mapCatching()`](#mapcatchingtransformfn) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
transforming the value of a result
```ts
declare const result: Result<number, Error>;

const transformed = result.map((value) => value 2); // Result<number, Error>
```

#### Example
returning a result instance
```ts
declare const result: Result<number, Error>;
declare function multiplyByTwo(value: number): Result<number, Error>;

const transformed = result.map((value) => multiplyByTwo(value)); // Result<number, Error>
```

#### Example
doing an async transformation
```ts
declare const result: Result<number, Error>;

const transformed = result.map(async (value) => value 2); // AsyncResult<number, Error>
```

#### Example
returning an async result instance

```ts
declare const result: Result<number, Error>;
declare function storeValue(value: number): AsyncResult<boolean, Error>;

const transformed = result.map((value) => storeValue(value)); // AsyncResult<boolean, Error>
```

### mapCatching(transformFn)

Like [`Result.map`](#maptransformfn) it transforms the value of a successful result using the `transform` callback.
In addition, it catches any exceptions that might be thrown inside the `transform` callback and encapsulates them
in a failed result.

#### Parameters

- `transformFn` callback function to transform the value of the result. The callback can be async as well.

**returns** * a new [`Result`](#result) instance with the transformed value, or a new [`AsyncResult`](#asyncresult) instance if the transform function is async.

### recover(onFailure)

Transforms a failed result using the `onFailure` callback into a successful result. Useful for falling back to
other scenarios when a previous operation fails.
The `onFailure` callback can also return other `Result` or [`AsyncResult`](#asyncresult) instances,
which will be returned as-is.
After a recovery, logically, the result can only be a success. Therefore, the error type is set to `never`, unless
the `onFailure` callback returns a result-instance with another error type.

#### Parameters

- `onFailure` callback function to transform the error of the result. The callback can be async as well.

**returns** a new successful [`Result`](#result) instance or a new successful [`AsyncResult`](#asyncresult) instance
when the result represents a failure, or the original instance if it represents a success.

> [!NOTE]
> Any exceptions that might be thrown inside the `onFailure` callback are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`Result.recoverCatching`](#recovercatchingonfailure) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
transforming the error into a value
Note: Since we recover after trying to persist in the database, we can assume that the `DbError` has been taken care
of and therefore it has been removed from the final result.
```ts
declare function persistInDB(item: Item): Result<Item, DbError>;
declare function persistLocally(item: Item): Result<Item, IOError>;

persistInDB(item).recover(() => persistLocally(item)); // Result<Item, IOError>
```

### recoverCatching(onFailure)

Like [`Result.recover`](#recoveronfailure) it transforms a failed result using the `onFailure` callback into a successful result.
In addition, it catches any exceptions that might be thrown inside the `onFailure` callback and encapsulates them
in a failed result.

#### Parameters

- `onFailure` callback function to transform the error of the result. The callback can be async as well.

**returns** a new successful [`Result`](#result) instance or a new successful [`AsyncResult`](#asyncresult) instance when the result represents a failure, or the original instance if it represents a success.

### Result.ok(value)

Creates a new result instance that represents a successful outcome.

#### Parameters

- `value` The value to encapsulate in the result.

**returns** a new [`Result`](#result) instance.

#### Example
```ts
const result = Result.ok(42); // Result<number, never>
```

### Result.error(error)

Creates a new result instance that represents a failed outcome.

#### Parameters

- `error` The error to encapsulate in the result.

**returns** a new [`Result`](#result) instance.

#### Example
```ts
const result = Result.error(new NotFoundError()); // Result<never, NotFoundError>
```

### Result.isResult(possibleResult)

Type guard that checks whether the provided value is a [`Result`](#result) instance.

#### Parameters

- `possibleResult` any value that might be a [`Result`](#result) instance.

**returns* `true` if the provided value is a [`Result`](#result) instance, otherwise `false`.

### Result.isAsyncResult(possibleAsyncResult)

Type guard that checks whether the provided value is a [`AsyncResult`](#asyncresult) instance.

#### Parameters

- `possibleAsyncResult` any value that might be a [`AsyncResult`](#asyncresult) instance.

**returns** `true` if the provided value is a [`AsyncResult`](#asyncresult) instance, otherwise `false`.

### Result.all(items)

Similar to `Promise.all`, but for results.
Useful when you want to run multiple independent operations and bundle the outcome into a single result.
All possible values of the individual operations are collected into an array. `Result.all` will fail eagerly,
meaning that as soon as any of the operations fail, the entire result will be a failure.
Each argument can be a mixture of literal values, functions, [`Result`](#result) or [`AsyncResult`](#asyncresult) instances, or `Promise`.

#### Parameters

- `items` one or multiple literal value, function, [`Result`](#result) or [`AsyncResult`](#asyncresult) instance, or `Promise`.

**returns** combined result of all the operations.

> [!NOTE]
> Any exceptions that might be thrown are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`Result.allCatching`](#resultallcatchingitems) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
basic usage
```ts
declare function createTask(name: string): Result<Task, IOError>;

const tasks = ["task-a", "task-b", "task-c"];
const result = Result.all(...tasks.map(createTask)); // Result<Task[], IOError>
```

#### Example
running multiple operations and combining the results
```ts
const result = Result.all(
  "a",
  Promise.resolve("b"),
  Result.ok("c"),
  Result.try(async () => "d"),
  () => "e",
  () => Result.try(async () => "f"),
  () => Result.ok("g"),
  async () => "h",
); // AsyncResult<[string, string, string, string, string, string, string, string], Error>
```

### Result.allCatching(items)

Similar to [`Result.all`](#resultallitems), but catches any exceptions that might be thrown during the operations.

#### Parameters

- `items` one or multiple literal value, function, [`Result`](#result) or [`AsyncResult`](#asyncresult) instance, or `Promise`.

**returns** combined result of all the operations.

### Result.wrap(fn)

Wraps a function and returns a new function that returns a result. Especially useful when you want to work with
external functions that might throw exceptions.
The returned function will catch any exceptions that might be thrown and encapsulate them in a failed result.

#### Parameters

- `fn` function to wrap. Can be synchronous or asynchronous.

**returns** a new function that returns a result.

#### Example
basic usage
```ts
declare function divide(a: number, b: number): number;

const safeDivide = Result.wrap(divide);
const result = safeDivide(10, 0); // Result<number, Error>
```

### Result.try(fn, [transform])

Executes the given `fn` function and encapsulates the returned value as a successful result, or the
thrown exception as a failed result. In a way, you can view this method as a try-catch block that returns a result.

#### Parameters

- `fn` function with code to execute. Can be synchronous or asynchronous.
- `transform` optional callback to transform the caught error into a more meaningful error.

**returns** a new [`Result`](#result) instance.

#### Example
basic usage
```ts
declare function saveFileToDisk(filename: string): void; // might throw an error

const result = Result.try(() => saveFileToDisk("file.txt")); // Result<void, Error>
```

#### Example
basic usage with error transformation
```ts
declare function saveFileToDisk(filename: string): void; // might throw an error

const result = Result.try(
  () => saveFileToDisk("file.txt"),
  (error) => new IOError("Failed to save file", { cause: error })
); // Result<void, IOError>
```

### Result.fromAsync(promise)

Utility method to transform a Promise, that holds a literal value or
a [`Result`](#result) or [`AsyncResult`](#asyncresult) instance, into an [`AsyncResult`](#asyncresult) instance. Useful when you want to immediately chain operations
after calling an async function.

#### Parameters

- `promise` a Promise that holds a literal value or a [`Result`](#result) or [`AsyncResult`](#asyncresult) instance.

**returns** a new [`AsyncResult`](#asyncresult) instance.

> [!NOTE]
> Any exceptions that might be thrown are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`Result.fromAsyncCatching`](#resultfromasynccatchingpromise) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
basic usage

```ts
declare function someAsyncOperation(): Promise<Result<number, Error>>;

// without 'Result.fromAsync'
const result = (await someAsyncOperation()).map((value) => value 2); // Result<number, Error>

// with 'Result.fromAsync'
const asyncResult = Result.fromAsync(someAsyncOperation()).map((value) => value 2); // AsyncResult<number, Error>
```

### Result.fromAsyncCatching(promise)

Similar to [`Result.fromAsync`](#resultfromasyncpromise) this method transforms a Promise into an [`AsyncResult`](#asyncresult) instance.
In addition, it catches any exceptions that might be thrown during the operation and encapsulates them in a failed result.

### Result.assertOk(result)

Asserts that the provided result is successful. If the result is a failure, an error is thrown.
Useful in unit tests.

#### Parameters

- `result` the result instance to assert against.

### Result.assertError(result)

Asserts that the provided result is a failure. If the result is successful, an error is thrown.
Useful in unit tests.

#### Parameters

- `result` the result instance to assert against.

## AsyncResult

Represents the asynchronous outcome of an operation that can either succeed or fail.

```ts
class AsyncResult<Value, Error> {}
```

### isAsyncResult

Utility getter that checks if the current instance is an `AsyncResult`.

### errorOrNull()

**returns** the encapsulated error if the result is a failure, otherwise `null`.

### getOrNull()

**returns** the encapsulated value if the result is successful, otherwise `null`.

### getOrDefault(defaultValue)

Retrieves the encapsulated value of the result, or a default value if the result is a failure.

#### Parameters

- `defaultValue` The value to return if the result is a failure.

**returns** The encapsulated value if the result is successful, otherwise the default value.

#### Example
obtaining the value of a result, or a default value
```ts
declare const result: AsyncResult<number, Error>;

const value = await result.getOrDefault(0); // number
```

#### Example
using a different type for the default value
```ts
declare const result: AsyncResult<number, Error>;

const value = await result.getOrDefault("default"); // number | string
```

### getOrElse(onFailure)

Retrieves the value of the result, or transforms the error using the `onFailure` callback into a value.

#### Parameters

- `onFailure` callback function which allows you to transform the error into a value. The callback can be async as well.

**returns** either the value if the result is successful, or the transformed error.

#### Example
transforming the error into a value
```ts
declare const result: AsyncResult<number, Error>;

const value = await result.getOrElse((error) => 0); // number
```

#### Example
using an async callback
```ts
const value = await result.getOrElse(async (error) => 0); // number
```

### getOrThrow()

Retrieves the encapsulated value of the result, or throws an error if the result is a failure.

**returns** The encapsulated value if the result is successful.

**throws** an error if the result is a failure.

> [!IMPORTANT]
> The error thrown will have the original error set as the `cause` property.

#### Example
obtaining the value of a result, or throwing an error
```ts
declare const result: AsyncResult<number, Error>;

const value = await result.getOrThrow(); // number
```

### fold(onSuccess, onFailure)

Returns the result of the `onSuccess` callback when the result represents success or
the result of the `onFailure` callback when the result represents a failure.

> [!NOTE]
> Any exceptions that might be thrown inside the callbacks are not caught, so it is your responsibility
> to handle these exceptions

#### Parameters

- `onSuccess` callback function to run when the result is successful. The callback can be async as well.

- `onFailure` callback function to run when the result is a failure. The callback can be async as well.

**returns** the result of the callback that was executed.

#### Example
folding a result to a response-like object

```ts
declare const result: AsyncResult<User, NotFoundError | UserDeactivatedError>;

const response = await result.fold(
  (user) => ({ status: 200, body: user }),
  (error) => {
    switch (error.type) {
      case "not-found":
        return { status: 404, body: "User not found" };
      case "user-deactivated":
        return { status: 403, body: "User is deactivated" };
    }
  }
);
```

### onFailure(action)

Calls the `action` callback when the result represents a failure. It is meant to be used for
side-effects and the operation does not modify the result itself.

#### Parameters

- `action` callback function to run when the result is a failure. The callback can be async as well.

**returns** the original instance of the result.

> [!NOTE]
> Any exceptions that might be thrown inside the `action` callback are not caught, so it is your responsibility
> to handle these exceptions

#### Example
adding logging between operations
```ts
declare const result: AsyncResult<number, Error>;

result
  .onFailure((error) => console.error("I'm failing!", error))
  .map((value) => value 2); // proceed with other operations
```

### onSuccess(action)

Calls the `action` callback when the result represents a success. It is meant to be used for
side-effects and the operation does not modify the result itself.

#### Parameters

- `action` callback function to run when the result is successful. The callback can be async as well.

**returns** the original instance of the result.

> [!NOTE]
> Any exceptions that might be thrown inside the `action` callback are not caught, so it is your responsibility
> to handle these exceptions

#### Example
adding logging between operations
```ts
declare const result: AsyncResult<number, Error>;

result
  .onSuccess((value) => console.log("I'm a success!", value))
  .map((value) => value 2); // proceed with other operations
```

#### Example
using an async callback
```ts
declare const result: AsyncResultResult<number, Error>;

const asyncResult = await result.onSuccess(async (value) => someAsyncOperation(value));
```

### map(transformFn)

Transforms the value of a successful result using the `transform` callback.
The `transform` callback can also return other `Result` or [`AsyncResult`](#asyncresult) instances,
which will be returned as-is (the `Error` types will be merged).
The operation will be ignored if the result represents a failure.

#### Parameters

- `transformFn` callback function to transform the value of the result. The callback can be async as well.

**returns** a new [`AsyncResult`](#asyncresult) instance with the transformed value

> [!NOTE]
> Any exceptions that might be thrown inside the `transform` callback are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`AsyncResult.mapCatching`](#mapcatchingtransformfn-1) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
transforming the value of a result
```ts
declare const result: AsyncResult<number, Error>;

const transformed = result.map((value) => value 2); // AsyncResult<number, Error>
```

#### Example
returning a result instance
```ts
declare const result: AsyncResult<number, Error>;
declare function multiplyByTwo(value: number): Result<number, Error>;

const transformed = result.map((value) => multiplyByTwo(value)); // AsyncResult<number, Error>
```

#### Example
doing an async transformation
```ts
declare const result: AsyncResult<number, Error>;

const transformed = result.map(async (value) => value 2); // AsyncResult<number, Error>
```

#### Example
returning an async result instance

```ts
declare const result: AsyncResult<number, Error>;
declare function storeValue(value: number): AsyncResult<boolean, Error>;

const transformed = result.map((value) => storeValue(value)); // AsyncResult<boolean, Error>
```

### mapCatching(transformFn)

Like [`AsyncResult.map`](#maptransformfn-1) it transforms the value of a successful result using the `transformFn` callback.
In addition, it catches any exceptions that might be thrown inside the `transformFn` callback and encapsulates them
in a failed result.

#### Parameters

- `transformFn` callback function to transform the value of the result. The callback can be async as well.

**returns** a new [`AsyncResult`](#asyncresult) instance with the transformed value

### recover(onFailure)

Transforms a failed result using the `onFailure` callback into a successful result. Useful for falling back to
other scenarios when a previous operation fails.
The `onFailure` callback can also return other `Result` or [`AsyncResult`](#asyncresult) instances,
which will be returned as-is.
After a recovery, logically, the result can only be a success. Therefore, the error type is set to `never`, unless
the `onFailure` callback returns a result-instance with another error type.

#### Parameters

- `onFailure` callback function to transform the error of the result. The callback can be async as well.

**returns** a new successful [`AsyncResult`](#asyncresult) instance when the result represents a failure, or the original instance
if it represents a success.

> [!NOTE]
> Any exceptions that might be thrown inside the `onFailure` callback are not caught, so it is your responsibility
> to handle these exceptions. Please refer to [`AsyncResult.recoverCatching`](#recovercatchingonfailure-1) for a version that catches exceptions
> and encapsulates them in a failed result.

#### Example
transforming the error into a value
Note: Since we recover after trying to persist in the database, we can assume that the `DbError` has been taken care
of and therefore it has been removed from the final result.
```ts
declare function persistInDB(item: Item): AsyncResult<Item, DbError>;
declare function persistLocally(item: Item): AsyncResult<Item, IOError>;

persistInDB(item).recover(() => persistLocally(item)); // AsyncResult<Item, IOError>
```

### recoverCatching(onFailure)

Like [`AsyncResult.recover`](#recoveronfailure-1) it transforms a failed result using the `onFailure` callback into a successful result.
In addition, it catches any exceptions that might be thrown inside the `onFailure` callback and encapsulates them
in a failed result.

#### Parameters

- `onFailure` callback function to transform the error of the result. The callback can be async as well.

**returns** a new successful [`AsyncResult`](#asyncresult) instance when the result represents a failure, or the original instance
if it represents a success.