# 모나드

이 글은 Typescript를 이용해 모나드를 구현 및 설명하기 위해 작성되었습니다.

> 당신이 만약 모나드를 이해하고 나면, 그걸 다른 사람에게 설명할 수 있는 능력을 잃어버리게 된다. 이것이 모나드의 저주다. - Douglas Crokford

Javascript를 얼마나 사용해 보았는가? `Promise.then()`을 사용해 보았는가?

그렇다면 이미 모나드를 체험해본 것이다. 바로 출발하자.

## 함수형 프로그래밍

함수형 프로그래밍에 대한 정확한 정의는 따로 있지만, 여기서는 몇 가지 규칙을 통해 함수형 코드를 짤 수 밖에 없도록 유도할 것이다.

* `let`이 없는 코드, `const`만 사용하는 코드
* `this`가 없는 코드
* `throw`가 없는 코드

이 규칙을 따르면 처음에는 불편할 것이다. 하지만 더 연습해보면 평소에 쓰던 코드는 거의 전부 위 3가지 규칙을 따르면서 똑같이 표현 가능하다는 것을 알게 될 것이다. 그런 새로운 표현법을 습득하는 순간, 요즘 세상이 "함수형"이라고 부르는 코드를 쓸 수 있게 된다.

모나드는 이 규칙들을 따르는 코드를 작성하기 더 쉽게 만들어주는 유용한 도구다.

## pipe()

`pipe()`는 함수 합성을 도와주는 헬퍼 함수다.

```ts
const multi2 = (x: number) => x * 2;

const add2 = (x: number) => x + 2;
```

이렇게 두 함수가 정의되어 있다고 하자. 이 때, "2를 곱한 다음, 2를 더한다" 라는 로직이 코드 내에서 여러번 사용되고 있다면, 한번에 두 동작을 모두 수행하는 새로운 함수를 정의하고자 하는 욕구가 생길 것이다.

```ts
const multi2AndAdd2 = (x: number) => add2(multi2(x));
```

`add2(multi2(x))`가 읽기 쉬운가? 함수명은 `multi2AndAdd2`인데, 구현은 `add2(multi2(x))`인 상황이다. 직관성이 떨어지는 코드다. 누군가는 이게 별 문제가 아니라고 할 수 있겠지만, 모두가 고개를 끄덕일만한 모습은 아니라고 생각한다. 만약 5개의 함수를 합성한다고 하면 더 끔찍해질 것이다.

그렇다면 상상해보자. `const multi2AndAdd2 = multi2 |> add2`와 같이 간편하게 쓸 방법은 없을까?

Javascript / Typescript를 사용하는 이상 가장 이상적인 형태를 만들지는 못하지만 (미래엔 가능할지도 모른다 - https://github.com/tc39/proposal-pipeline-operator 하지만, 아직 Stage 1이라 먼 미래다), 비슷하게 만들 방법이 있다. 그게 `pipe()` 함수다.

```ts
const multi2AndAdd2 = pipe(
    multi2,
    add2,
);
```

차이가 느껴지는가?

* 실행 순서가 직관적이다. multi2 -> add2 순으로 실행될 것이라는게 한 눈에 보인다.
* x가 사라졌다. 그럼에도 우리는 이 함수가 `number`를 받아서 `number`를 반환할 것이라는 사실을 알고 있다. `number` 타입을 지닌 함수 인자의 이름이 `x`건, `wowVeryAmazingVariableName`이건 아무 상관이 없다. 그러면 애초에 이름을 부여할 필요가 없으니까 생략하는게 더 코드가 간결해진다.
  * `x => f(x)`를 축약해 `f`만 남기는 것을 Eta conversion이라고 부른다.
  * 이를 활용해 인자를 표기하지 않는 코딩을 Point-free style이라고 부른다.

글쓴이가 현재 사용하는 `pipe()` 함수의 정의는 아래와 같다. 앞으로 계속 사용하게 될테니 어딘가에 복붙해두자.

```ts
export function pipe<T extends unknown[], A>(
  fnA: (...x: T) => A,
): (...x: T) => A;
export function pipe<T extends unknown[], A, B>(
  fnA: (...x: T) => A,
  fnB: (x: A) => B,
): (...x: T) => B;
export function pipe<T extends unknown[], A, B, C>(
  fnA: (...x: T) => A,
  fnB: (x: A) => B,
  fnC: (x: B) => C,
): (...x: T) => C;
export function pipe<T extends unknown[], A, B, C, D>(
  fnA: (...x: T) => A,
  fnB: (x: A) => B,
  fnC: (x: B) => C,
  fnD: (x: C) => D,
): (...x: T) => D;
export function pipe<T extends unknown[], A, B, C, D, E>(
  fnA: (...x: T) => A,
  fnB: (x: A) => B,
  fnC: (x: B) => C,
  fnD: (x: C) => D,
  fnE: (x: D) => E,
): (...x: T) => E;
export function pipe<T extends unknown[], A, B, C, D, E, F>(
  fnA: (...x: T) => A,
  fnB: (x: A) => B,
  fnC: (x: B) => C,
  fnD: (x: C) => D,
  fnE: (x: D) => E,
  fnF: (x: E) => F,
): (...x: T) => F;
export function pipe(...fns: Array<(...x: unknown[]) => unknown>) {
  return (...x: unknown[]) => {
    const [first, ...others] = fns;
    return others.reduce((val, fn) => fn(val), first(...x));
  };
}
```

기괴한 것은 알고 있다. TypeScript의 기능으로는 이런 형태의 타이핑을 간략하게 할 방법이 없기 때문에, 오버로딩을 많이 걸 수 밖에 없었다. `pipe()`를 지원하는 RxJS 등의 다른 함수형 라이브러리들을 살펴봐도 전부 비슷하게 구현되어 있다. 그래도, 이런 더러운 정의를 뒤에서 해줘야 우리가 자주 보는 부분의 코드가 깨끗해지는 것이다. 이런 함수는 한번 정의해두면 거의 볼 필요가 없으니까, 이쪽이 더러운게 훨씬 낫다.

이쯤 되면 왜 모나드를 설명한다면서 파이프가 튀어나왔는지 의문이 들 것이다. 모나드는 `함수 합성`에 관한 패턴이다. 파이프는 `함수 합성`을 쉽게 해주는 도구다. 결국, Typescript 위에서 모나드를 설명하려면 `pipe()`를 사용하는 것이 가장 편하기 때문에 먼저 다루게 되었다.

## 아름다운 세상

`pipe()`라는 도구를 손에 넣었다. 이제 이걸 이용해서 뭐든지 합성하고, 어떤 코드도 아름답게 표현할 수 있을것만 같다. 정말 아름다운 세상이야. 그렇지? 그러니까 이제 간단한 Validation 메소드를 작성해보자.

```ts
interface User {
    name: string;
    email: string;
}

function nameNotEmpty(user: User) {
    if (user.name === '') {
        throw new Error('Name is empty');
    }
    return user;
}
```

잠깐. 기억을 되살려보자. 우리는 `throw`를 사용하지 않기로 했다. 대신, `success` 여부를 함께 리턴하는 방식으로 고쳐보자.

```ts
interface User {
    name: string;
    email: string;
}

function nameNotEmpty(user: User) {
    if (user.name === '') {
        return {
            success: false,
            err: 'Name is empty',
        };
    }
    return {
        success: true,
        value: user,
    }
}

function nameLengthMax50(user: User) {
    if (user.name.length > 50) {
        return {
            success: false,
            err: 'Name is longer than 50',
        };
    }
    return {
        success: true,
        value: user,
    }
}

function emailNotEmpty(user: User) {
    if (user.email === '') {
        return {
            success: false,
            err: 'Email is empty',
        };
    }
    return {
        success: true,
        value: user,
    }
}
```

작고 귀여운 3개의 Validation 로직을 작성했다. 이것들을 합성해서 `validateUser` 로직을 만들면 되겠다. 그 전에, 코드 중복을 줄이고 나서 시작해보자.

```ts
interface Success<T> {
    success: true;
    value: T;
}

interface Failure {
    success: false;
    err: string;
}

type Result<T> = Success<T> | Failure;

function success<T>(value: T): Result<T> {
    return {
        success: true,
        value,
    }
}

function failure(err: string): Result<never> {
    return {
        success: false,
        err,
    }
}

interface User {
    name: string;
    email: string;
}

function nameNotEmpty(user: User) {
    if (user.name === '') {
        return failure('Name is empty');
    }
    return success(user);
}

function nameLengthMax50(user: User) {
    if (user.name.length > 50) {
        return failure('Name is longer than 50');
    }
    return success(user);
}

function emailNotEmpty(user: User) {
    if (user.email === '') {
        return failure('Email is empty');
    }
    return success(user);
}
```

공통되는 로직을 `failure()`, `success()`로 추출해, 각 Validation 로직의 중복 코드를 줄였다. 훨씬 보기 좋아졌다. 이제 이 함수들을 `pipe()`로 합성해보자.

```ts
const validateUser = pipe(
    nameNotEmpty, // Argument of type '(user: User) => Failure<string> | Success<User>' is not assignable to parameter of type '(user: User) => User'
    nameLengthMax50,
    emailNotEmpty,
)
```

에러가 발생한다. 이유는 간단하다. `nameNotEmpty`가 리턴하는 유형은 `Result<User>`인데, `nameLengthMax50`이 요구하는 유형은 `User`라서 그렇다. 어떻게든 해결해보자. 이전에 이미 실패했으면 계속 실패한 채로 두고, 성공한 경우에만 새로 실행하면서 넘어가면 되지 않을까?

```ts
const validateUser = pipe(
    nameNotEmpty,
    (result: Result<User>) => {
        if (result.success === false) {
            return result;
        }
        return nameLengthMax50(result.value);
    }
    (result: Result<User>) => {
        if (result.success === false) {
            return result;
        }
        return emailNotEmpty(result.value);
    }
)
```

음... 동작한다. 동작은 한다. 하지만 이건 아름다운 세상의 코드가 아니다. 검증 로직이 10단계로 이루어져 있다면, `result.success === false`와 `result.value`를 10번 봐야한다고? 생각만 해도 끔찍하다. 다행히도, 정말 다행히도 이런 상황에 사용할 수 있는 필살기를 JavaScript가 가지고 있다. **고차 함수**다.

```ts
function bind<A, B>(fn: (a: A) => Result<B>) {
    return (result: Result<A>): Result<B> => {
        if (result.success === false) {
            return result;
        }
        return fn(result.value);
    }
}
```

`bind`는 `A => Result<B>` 꼴의 함수를 받아서 `Result<A> => Result<B>` 꼴의 함수로 변환해주는 **고차 함수**다. `bind(emailNotEmpty)`는 어떻게 동작할까? 위 정의에서 `fn` 자리를 `emailNotEmpty`로 대체해보자.

```ts
function bind<User, User>(emailNotEmpty: (a: User) => Result<User>) {
    return (result: Result<User>): Result<User> => {
        if (result.success === false) {
            return result;
        }
        return emailNotEmpty(result.value);
    }
}
```

`bind(emailNotEmpty)`가 리턴하는 함수의 모양이 익숙하지 않은가? 아까 타입 에러를 해결하기 위해 `validateUser`에 끼워넣은 로직과 모양이 완벽히 똑같다! 마침내 모든 재료가 준비되었다. `bind`를 이용해 최종적으로 `validateUser`을 완성해보자.

```ts
const validateUser = pipe(
    nameNotEmpty,
    bind(nameLengthMax50),
    bind(emailNotEmpty),
)

validateUser({
    name: 'name',
    email: 'my@email.com'
}); // { success: true, value: { name: 'name', email: 'my@email.com' } }

validateUser({
    name: '',
    email: 'my@email.com'
}); // { success: false, err: 'Name is empty' }
```

다시 아름다운 세상으로 돌아왔다. 그리고, 축하한다. 여기까지 따라온 당신은 모나드를 구현했다.

## 모나드의 발명

* 함수형 프로그래밍은 함수 합성을 적극적으로 활용한다
* 리턴값에 특성이 부여된 함수는 그대로 합성이 불가능하다
  * 성공/실패 라는 "특성" - Result<T>
  * `A => Result<B>` 꼴의 함수들간의 합성이 문제
* 그렇다면 합성이 가능하게 만들자.
  * bind()

자연스러운 흐름이다. 모나드는 어디서 뚝 떨어진게 아니고, 디자인 패턴들과 같이 많은 개발자들이 겪는 현상을 일반화해서 처리해주는 도구다. 사용법을 익힌 다음, 잘 사용하면 그만이다.

하지만 더 알고싶은 욕심이 드는건 어쩔 도리가 없다. 아래부터는 그래서 모나드가 무엇인가, 에 관한 구구절절한 설명이다.

## 모나드란 무엇인가

잘 설명할 자신도 없고, 한번에 이해한다는 기대는 전혀 없지만 최대한 풀어서 설명해보려고 한다.

```
nameNotEmpty: User => Result<User>
nameLengthMax50: User => Result<User>
emailNotEmpty: User => Result<User>
```

```
nameNotEmpty: User => Result<User>
bind(nameLengthMax50): Result<User> => Result<User>
bind(emailNotEmpty): Result<User> => Result<User>

pipe(
    nameNotEmpty,
    bind(nameLengthMax50),
    bind(emailNotEmpty),
): User => Result<User>
```

여태까지의 과정을 타입으로만 표현하면 이와 같다. 이걸 User와 Validation에 한정짓지 말고, 일반화해서 표현해보자.

```
fnA: A => M<B>
bind(fnB): M<B> => M<C>
bind(fnC): M<C> => M<D>

pipe(
    fnA,
    bind(fnB),
    bind(fnC),
): A => M<D>
```

우리가 다룬 예제에서는 `A`, `B`, `C`, `D`가 모두 `User`이고 `M`에 `Result`가 대응되었던 것이다.

모나드는 3가지 요소로 구성된다.

* 특성을 표현하는 타입 `M`
* `M`을 생성하는 함수 `unit`
  * `unit: A => M<A>`
  * `Result`의 경우 `success`, `failure` 두 종류 `unit`이 있는 셈이다
* `(A => M<B>) => (M<A> => M<B>)` 변환을 해주는 `bind`
  * `bind`의 구현은 `M`에 따라 각각 달라진다.

`M`에 대한 `unit`, `bind`를 정의할 수 있다면, 그건 모나드다.

모나드는, 모든 { `M`, `unit`, `bind` }에 대한 집합이다.

이게 대체 무슨 소린가 싶은게 당연하다. 집합을 정의하는 방법부터 생각해보자.

`A = { 1, 2, 3, 4, 5 }` - 유한한 경우 원소를 나열해서 표현할 수 있다.

`A = { x | x > 10 }` - 무한집합의 경우 이렇게 표현한다.

`Monad = { M | M은 타입 && M에 대한 unit을 정의 가능 && M에 대한 bind를 정의 가능 }` - 이렇게 표현하는 것 또한 가능하다.

이런 식으로 타입의 집합을 표현하는걸 함수형 동네에서는 `Typeclass`라고 부른다.

## Promise

서문에서 `Promise.then()`을 사용해 봤다면, 이미 모나드를 사용해본 것이라고 했다. 왜 그런지 알아보자.

* `M = Promise`
* `unit = Promise.resolve() / Promise.reject()`
* `bind = Promise.then()`

```ts
Promise.resolve(10)
    .then(x => Promise.resolve(x * 2))
    .then(x => Promise.resolve(x.toString()))
    .then(x => Promise.resolve(x + '!'))
    .then(x => console.log(x)) // '20!'
```

느낌이 오는가? .then() 안에 투입된 함수들은 `A => Promise<B>` 형태를 띄고 있다. 그리고 함수간에 합성이 잘 이뤄지고 있다. 우리가 위에서 구현했던 `Result` 모나드와 작동 방식이 동일하다! Promise가 콜백보다 사용하기 편리해진 결정적인 이유는, 모나드를 응용해 `A => Promise<B>` 꼴의 함수들을 쉽게 합성할 방법을 제시했기 때문이다!

> Promise는 엄밀한 의미의 모나드를 충족하지 않는다. 그래도 모나드처럼 사용하는 데 있어 별다른 문제는 없다.

## Array

Array 또한 모나드로 써먹을 수 있다.

```ts
function flatten<A>(arr2: A[][]): A[] {
  return arr2.reduce((a, b) => a.concat(b));
}

function bind(fn: (a: A) => B[]) {
    return (arr: A[]): B[] => flatten(arr.map(fn));
}
```

```ts
const getPattern = pipe(
    (x: number) => [x, x + 1, x + 2, x + 3],
    bind(x => [x, x * 10])
)

console.log(getPattern(1)) // [1, 10, 2, 20, 3, 30, 4, 40]
```

`number => Array<number>` 꼴의 함수들을 합성했다. 모나드!

> Array.map()은 함수형의 Functor라는 개념과 대응된다. 이 또한 설명할 기회가 있을 것이다.

## 기타

함수형 프로그래밍에서 같은 개념을 다른 이름으로 부르는 경우가 많아서 공부에 방해가 된다. 이 글을 읽는 여러분은 같은 삽질을 반복하지 않기를 바라며.

* bind = flatMap = chain
* unit = of = return
* fmap = map

당장 생각나는건 이정도가 있다.

함수형 프로그래밍을 공부하는 모든 사람들에게, 화이팅.