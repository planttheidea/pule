# nage

Efficient, tiny object pool

## Table of contents

- [nage](#nage)
  - [Table of contents](#table-of-contents)
  - [Usage](#usage)
  - [Typings](#typings)
  - [Pool options](#pool-options)
      - [create](#create)
      - [initialSize](#initialsize)
      - [maxSize](#maxsize)
      - [name](#name)
      - [onReserve](#onreserve)
      - [onRelease](#onrelease)
      - [onReset](#onreset)
  - [Pool methods](#pool-methods)
      - [reserve](#reserve)
      - [release](#release)
      - [reserveN](#reserven)
      - [releaseN](#releasen)
      - [reset](#reset)
  - [Pool values](#pool-values)
      - [available](#available)
      - [reserved](#reserved)
      - [size](#size)
  - [Development](#development)

## Usage

```typescript
import nage from 'nage';

// create your pool
const pool = nage();

// reserve an object from the pool
const object = pool.reserve();

// do your logic

// release object back to the pool
pool.release(object);
```

## Typings

All typings for the internals are under the `Nage` namespace. The available types:

```typescript
type Entry = {
  [key: string]: any;
  [index: number]: any;
};

type Creator<Pooled> = () => Pooled;

type Handler<Pooled> = (entry: Pooled) => void;

type ResetHandler<Pooled> = (stack: Pooled[]) => void;

type Options<Pooled extends {} = Entry> = {
  create?: Creator<Pooled>;
  initialSize?: number;
  name?: number | string | symbol;
  onRelease?: Handler<Pooled>;
  onReserve?: Handler<Pooled>;
  onReset?: ResetHandler<Pooled>;
};
```

The pool itself is not namespaced, it should be available directly as `NagePool`:

## Pool options

#### create

_defaults to () => ({})_

The method used to create a new object in the pool. The default returns an empty object, but if you have objects with a consistent structure it is more memory efficient to return an object with that structure to reduce the number of hidden classes created under-the-hood.

```typescript
type Obj = {
  name: string | null;
  target: any;
}

const pool = nage<Obj>({
  create() {
    return {
      name: null,
      target: null,
    };
  },
});
```

**NOTE**: This function does not receive any arguments, and must return an object of some kind. This can be a standard POJO, array, Map, Set, etc., but it cannot be a primitive or `null`.

#### initialSize

_defaults to 1_

```typescript
const pool = nage<Obj>({ initialSize: 10 });
```

The number of objects to prepopulate the pool with. If you expect a number of objects to be used in parallel, it is advised to prepopulate the pool with the number appropriate for the use-case.

#### maxSize

_defaults to Infinity_

```typescript
const pool = nage<Obj>({ maxSize: 10 });
```

The maximum number of objects that will live in the pool. If you have reserved an object from the pool and try to release it back to the pool, it will allow the object to be garbage collected.

**NOTE**: If you provide an `initialSize` that is larger than the `maxSize`, the `maxSize` will be used as the `initialSize`.

#### name

```typescript
const pool = nage<Obj>({ name: 'custom-name' });
```

The name for the given pool. This doesn't impact anything at runtime, but can help with debugging as an identifier.

#### onReserve

```typescript
type Obj = { [key: string]: any };

const pool = nage<Obj>({
  onReserve(object) {
    object.reservedAt = Date.now();
  },
});
```

Handler called for a newly-reserved item from the pool, called with the object prior to being returned. This method is handy if you want to prepopulate the object with some data.

#### onRelease

```typescript
type Obj = { [key: string]: any };

const pool = nage<Obj>({
  onRelease(object) {
    for (const key in object) {
      object[key] = null;
    }
  },
});
```

Handler called for a newly-released item back into the pool, called with the object just prior to being added to the stack. This method is handy to perform cleanup of the object in preparation for future use.

#### onReset

```typescript
const pool = nage<Obj>({
  onReset(stack) {
    stack.forEach((entry) => {
      console.log(entry);
    });
  },
});
```

Handler called just prior to the stack being [`reset`](#reset). This method is handy if you need to take a snapshot of the existing pool prior to it being purged and repopulated.

## Pool methods

#### reserve

Reserves an object from the pool. If no objects remain to be reserved, it will create a new one for you based on the [`create`](#create) method from [options](#pool-options).

```typescript
const object = pool.reserve();
```

#### release

Releases an object back to the pool from where it came from.

```typescript
pool.release(object);
```

#### reserveN

Reserves multiple objects from the pool. If no objects remain to be reserved, it will create new ones for you based on the [`create`](#create) method from [options](#pool-options).

```typescript
const object = pool.reserveN(10);
```

#### releaseN

Releases multiple objects back to the pool from where it came from.

```typescript
pool.releaseN([object, anotherObject]);
```

#### reset

Resets the pool to its initial state, which is based on the [`initialSize`](#initialsize) value from [options](#options).

```typescript
pool.reset();
```

## Pool values

#### available

The number of objects in the pool available for reservation.

#### reserved

The number of objects in the pool unavailable because they are reserved.

#### size

The total number of objects in the pool, regardless of reservation status.

## Development

Standard stuff, clone the repo and `npm install` dependencies. The npm scripts available:

- `benchmark` => run the benchmark suite pitting `moize` against other libraries in common use-cases
- `build` => run rollup to build the distributed files in `dist`
- `clean` => run `rimraf` on the `dist` folder
- `dev` => run webpack dev server to run example app (playground!)
- `dist` => runs `clean` and `build`
- `lint` => runs ESLint against all files in the `src` folder
- `lint:fix` => runs `lint``, fixing any errors if possible
- `prepublish` => runs `compile-for-publish`
- `prepublish:compile` => run `lint`, `test:coverage`, and `dist`
- `test` => run test functions with `NODE_ENV=test`
- `test:coverage` => run `test` but with `nyc` for coverage checker
- `test:watch` => run `test`, but with persistent watcher
