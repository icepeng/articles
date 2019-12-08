# Monoid

모노이드 너무 어렵다

```ts
interface Monoid<T> {
  empty: T;
  append: (a: T, b: T) => T;
}

const string: Monoid<string> = {
  empty: '',
  append: (a, b) => a + b,
};

const sum: Monoid<number> = {
  empty: 0,
  append: (a, b) => a + b,
};

const all: Monoid<boolean> = {
  empty: true,
  append: (a, b) => a && b,
};

const any: Monoid<boolean> = {
  empty: false,
  append: (a, b) => a || b,
};

const concat = <T>(): Monoid<T[]> => ({
  empty: [],
  append: (a, b) => [...a, ...b],
});

const endo = <T>(): Monoid<(x: T) => T> => ({
  empty: (x: T) => x,
  append: (x, y) => a => x(y(a)),
});

function fold<M>(monoid: Monoid<M>) {
  return (arr: M[]) => arr.reduce(monoid.append, monoid.empty);
}

function foldMap<M>(monoid: Monoid<M>) {
  return <A>(f: (a: A) => M) => (arr: A[]) =>
    arr.map(f).reduce(monoid.append, monoid.empty);
}

fold(string)(['wow', 'very', 'doge']); // 'wowverydoge'
fold(sum)([1, 2, 3, 4]); // 10
fold(all)([true, true, false]); // false
fold(any)([true, true, false]); // true
fold(concat<number>())([
  [1, 2, 3],
  [4, 5, 6],
]); // [1, 2, 3, 4, 5, 6]

foldMap(any)((x: string) => x.length > 3)(['wow', 'very', 'doge']); // true

function foldr<A, B>(f: (a: A, b: B) => B) {
  return (b: B) => (as: A[]) => as.reduceRight((acc, a) => f(a, acc), b);
}

function foldl<A, B>(f: (b: B, a: A) => B) {
  return (b: B) => (as: A[]) => as.reduce((acc, a) => f(acc, a), b);
}

const add = (a: number, b: number) => a + b;

foldr(add)(0)([1, 2, 3, 4]); // 10

function foldrFromFoldMap<A, B>(f: (a: A, b: B) => B) {
  return (b: B) => (as: A[]) =>
    foldMap(endo<B>())((a: A) => (b: B) => f(a, b))(as)(b);
}

foldrFromFoldMap(add)(0)([1, 2, 3, 4]); // 10

function foldMapFromFoldr<M>(monoid: Monoid<M>) {
  return <A>(f: (a: A) => M) => (as: A[]) =>
    foldr(monoid.append)(monoid.empty)(as.map(f));
}

foldMapFromFoldr(sum)((x: number) => x * 2)([1, 2, 3, 4]); // 20

```