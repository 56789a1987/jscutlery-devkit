# @jscutlery/operators

This package regroups a couple of RxJS operators meant to simplify some common patterns.

- [@jscutlery/operators](#jscutleryoperators)
- [Installation](#installation)
- [`suspensify`](#suspensify)
  - [Goals](#goals)
  - [Usage](#usage)
    - [Basic](#basic)
    - [With Angular](#with-angular)
    - [With `@rx-angular/state`](#with-rx-angularstate)
  - [Alternatives](#alternatives)
  - [F.A.Q.](#faq)
    - [How does it defer from `materialize`?](#how-does-it-defer-from-materialize)
- [Roadmap](#roadmap)

# Installation

```sh
yarn add @jscutlery/operators

# or

npm install @jscutlery/operators
```

# `suspensify`

When dealing with an asynchronous source of data in a web application, it is a common requirement to display different content depending on the following states:

- **pending**: the data is being fetched
- **error**: an error occurred while fetching the data
- **success**: some data has been fetched
- **finalized**: there is no more data to fetch or an error happened

The most common ways of implementing this are error-prone as they are either based on a complex combination of native RxJS operators or side effects that break the reactivity chain.

The `suspensify` operator is meant to provide a simple and efficient way of dealing with data fetching by providing deriving a state from the source observable.

## Goals

- ⚡️ simplify the implementation of asynchronous data fetching
- 🎬 know when the data is being fetched and show some loading indicator
- 🐞 avoid common mistakes like showing data and the last error simultaneously
- 💥 simplify and encourage error-handling

## Usage

### Basic

```ts
interval(1000)
  .pipe(take(2), suspensify())
  .subscribe((data) => console.log(data));
```

```ts
{
  value: undefined,
  error: undefined,
  pending: true,
  finalized: false,
}
{
  value: 0,
  error: undefined,
  pending: false,
  finalized: false,
}
...
{
  value: 1,
  error: undefined,
  pending: false,
  finalized: true,
}
```

### With Angular

```ts
@Component({
  template: `
    <ng-container *ngIf="suspense$ | async as suspense">

      <my-spinner *ngIf="suspense.pending"></my-spinner>

      <div *ngIf="suspense.error">{{ suspense.error }}</div>

      <div *ngIf="suspense.value">{{ suspense.value }}</div>

    </ng-container>
  `,
})
export class MyComponent {
  suspense$ = this.fetchData().pipe(suspensify());
  ...
}
```

### With `@rx-angular/state`

```ts
@Component({
  template: `
    <my-spinner *ngIf="pending$ | async"></my-spinner>

    <div *ngIf="error$ | async as error">{{ error }}</div>

    <div *ngIf="value$ | async" as value>{{ value }}</div>
  `,
})
export class MyComponent {
  value$ = this.state.select('beer', 'value');
  error$ = this.state.select('beer', 'error');
  pending$ = this.state.select('beer', 'pending');

  constructor(private state: RxState<{ beer: Suspense<'🍻'> }>) {
    this.state.connect('beer', this.fetchBeer());
  }
}
```

## Alternatives

- [@rx-angular/state's rxLet](https://www.rx-angular.io/docs/template/api/let-directive)

## F.A.Q.

### How does it defer from `materialize`?

`materialize` doesn't produce a derived state. In fact, the `C` (complete) event doesn't contain the last emitted value.
Also, `materialized` doesn't trigger a `pending` event so the observable doesn't emit anything before the first value is emitted or an error occurs or the source completes.

# Roadmap

1. `mergeSuspense` function should merge multiple sources in one state that contains the global state of all sources and each one of them.
