# Let it Const

Javascript / Typescript 위에서 함수형 프로그래밍을 시작하려면, `let`을 전부 `const`로 바꾸는 연습부터 할 필요가 있다.

왜? 순수 함수형 패러다임에서는 변수를 사용하지 않는다. `const a = 3`과 같이 한번 선언되었으면, 프로그램이 실행되는 동안 `a`는 영원히 3이다.

이 제약이 불편하다고 생각할 수 있지만, 실제로는 큰 이점을 가져다준다.

- 디버깅할 때, 값의 변화를 추적할 필요가 없다. 한번 선언된 값은 변하지 않으므로, 관심있는 값을 한번 찍어보면 그만이다.
- 코드가 명료해진다. 제약에 의해 변수에 들어있는 "값" 자체보다는 그것을 변환하는 "과정", 즉 논리에 100% 집중할 수 있게 된다.
- Node 환경에서는 의미없지만, 모든 값이 readonly, 읽기 전용이라는 것은 멀티스레드 환경에서 큰 이점을 가져다준다. 공유 변수로 인한 문제가 없기 때문에.

## 진정한 Const를 위해

배열 / 오브젝트의 경우에는 제약 조건이 더 추가된다.

```
const arr = [1, 2, 3];
arr.push(4);
arr[0] = 10;

const obj = {
    low: 10,
    mid: 20,
    high: 30,
}

obj.low = 0;
```

위의 모든 코드는 에러가 발생하지 않는다. `const`로 선언했음에도, 내부의 값은 여전히 변경 가능하다.

왜냐하면, JavaScript는 `Objects` 유형은 해당하는 메모리에 **값**이 아닌 **주소**가 할당되기 때문이다. (Objects에는 객체, 배열, 함수, Map, Set, 등등...이 포함된다.) `obj.low = 0;` 과 같이 하더라도 `obj`의 주소 자체는 변하지 않았기 때문에, 여전히 `const`의 정의에 위배되지 않는 것이다.

따라서, 우리는 겉보기에만 `const`가 아닌 진정한 `const`를 추구할 것이다. 이를 달성하기 위한 첫번째 규칙.

**`=` 연산자를 사용할 때, 좌변에는 무조건 `const`가 있어야 한다!**

`=` 연산자를 더이상 대입문으로 생각하지 말자. 선언문으로 생각하면 된다. 선언을 한다는 것은, 새로운 값을 만든다는 것이므로, 항상 `const a = 10;`과 같이 좌변에 `const`가 따라오게 될 것이다. 이것으로 많은 실수를 피할 수 있다.

두번째 규칙.

**내부 값을 변경하는 메소드를 사용하지 말 것.**

JavaScript의 내장 함수들 중에 주소를 그대로 두고, 내부 값을 조작하는 것들이 존재한다. 예를 들어, `Array.push()`. 이것들은 개발자가 의식하지 못한 사이 우리들의 Const 세계를 부수려고 시도하기 때문에 더 위험하다. 이들을 피하기 위해서는 위험한 메소드들을 외우는 수 밖에 없다.

- Array.push
- Array.splice
- Array.sort
- ...여기저기 더 많다.

확실하지 않을 때는, 구글에 "is \_\_\_ immutable" 이라고 검색해보는 것이 좋다.

이제 정말로 모든 준비를 마쳤다. 출발하자.

## map/filter/reduce

`Array.map()`, `Array.filter()`, `Array.reduce()`는 ES6부터 추가된 메소드로, 배열을 다루는 데 있어 유용한 방법을 제시해준다.

```ts
function listDouble(arr: number[]) {
  const result = [];
  for (let item of arr) {
    result.push(item * 2);
  }
  return result;
}
```

`Array.push()`를 사용하기 때문에, 규칙을 어겼다. 다르게 해보자.

```ts
function listDouble(arr: number[]) {
  return arr.map(item => item * 2);
}
```

단 한줄로 목표를 달성했다.

```ts
function getPositives(arr: number[]) {
  const result = [];
  for (let item of arr) {
    if (item > 0) {
      result.push(item);
    }
  }
  return result;
}
```

`Array.push()`를 없애려면 조건에 따라 배열의 일부만 남겨야 한다.

```ts
function getPositives(arr: number[]) {
  return arr.filter(item => item > 0);
}
```

```ts
function sum(arr: number[]) {
  let result = 0;
  arr.forEach(item => {
    result += item;
  });
  return result;
}
```

`let`이 등장했다. 고쳐보자.

여기서 짚고 넘어갈 점. `for-of`문이나 `Array.forEach()`를 사용하는 순간, 이미 뭔가 잘못되었을 확률이 높다. 다른 방법이 없나 생각해봐야 한다.

```ts
function sum(arr: number[]) {
  return arr.reduce((acc, item) => acc + item, 0);
}
```

## 삼항 연산자

let - if에서 단순한 형태라면 `const a = condition ? x : y`와 같이 엮어버리면 된다

```ts
function example() {
  const result = doSomething();
  let status;

  if (result.error) {
    status = 'ERROR';
  } else {
    status = 'SUCCESS';
  }

  return {
    result,
    status,
  };
}
```

```ts
function example() {
  const result = doSomething();
  const status = result.error ? 'ERROR' : 'SUCCESS';

  return {
    result,
    status,
  };
}
```

## 메소드 분리

let - if로 된 로직을 if - return 하는 별도 함수로 뜯어낼 수 있다

```ts
function example() {
  const action = getAction();
  let score = getScore();

  if (action === 'DOUBLE') {
    score = score * 2;
  }

  if (action === 'HALF') {
    score = score / 2;
  }

  if (action === 'ZERO') {
    score = 0;
  }

  updateScore(score);
}
```

```ts
function applyAction(score: number, action: 'DOUBLE' | 'HALF' | 'ZERO') {
  if (action === 'DOUBLE') {
    return score * 2;
  }

  if (action === 'HALF') {
    return score / 2;
  }

  if (action === 'ZERO') {
    return 0;
  }
}

function example() {
  const action = getAction();
  const score = getScore();
  const nextScore = applyAction(score);
  updateScore(nextScore);
}
```

## 고차 함수

Array.map과 비슷하게, 본인이 필요한대로 "함수를 받는 함수"를 구성해서 쓰면 어지간한 문제를 해결 가능

```ts
interface User {
  username: string;

  password: string;

  address: string;
}

function validateUser(user: User) {
  if (user.username.length === 0) {
    return 'Username 길이는 0보다 길어야 합니다';
  }

  if (user.username.length > 24) {
    return 'Username 길이는 24 이하입니다';
  }

  if (user.password.length === 0) {
    return 'Password 길이는 0보다 길어야 합니다';
  }

  if (user.password.length > 50) {
    return 'Password 길이는 50 이하입니다';
  }

  if (user.address.length === 0) {
    return 'Address 길이는 0보다 길어야 합니다';
  }

  if (user.address.length > 50) {
    return 'Address 길이는 50 이하입니다';
  }
}

console.log(
  validateUser({
    username: 'username',
    password: 'password',
    address: '',
  }),
);
```

```ts
interface User {
  username: string;

  password: string;

  address: string;
}

interface Validator<T> {
  constraint: (model: T) => boolean;
  message: string;
}

function validatorFactory<T>(validators: Validator<T>[]) {
  return (model: T) => {
    const matchedValidator = validators.find(validator =>
      validator.constraint(model),
    );
    if (!matchedValidator) {
      return undefined;
    }
    return matchedValidator.message;
  };
}

const userValidators: Validator<User>[] = [
  {
    constraint: user => user.username.length === 0,
    message: 'Username 길이는 0보다 길어야 합니다',
  },
  {
    constraint: user => user.username.length > 24,
    message: 'Username 길이는 24 이하입니다',
  },
  {
    constraint: user => user.password.length === 0,
    message: 'Password 길이는 0보다 길어야 합니다',
  },
  {
    constraint: user => user.password.length > 50,
    message: 'Password 길이는 50 이하입니다',
  },
  {
    constraint: user => user.address.length === 0,
    message: 'Address 길이는 0보다 길어야 합니다',
  },
  {
    constraint: user => user.address.length > 50,
    message: 'Address 길이는 50 이하입니다',
  },
];

const validateUser = validatorFactory(userValidators);

console.log(
  validateUser({
    username: 'username',
    password: 'password',
    address: '',
  }),
);
```

## 기타 잡기술

default 값 - || 사용
nullish 체크 - && 사용

```ts
let score = getScore();

if (!score) {
  score = 0;
}

const user = getUser();
let name;

if (user.profile) {
  name = user.profile.name;
}

const score = getScore() || 0;

const user = getUser();
const name = user.profile && user.profile.name;
```

destructuring, rest operator 응용한 잡기술

```ts
const product = getProduct();
const name = product.name;
const price = product.price;
//
const { name, price } = getProduct();
```

```ts
const excel = getExcel();
const header = excel[0];
const data = excel.slice(1);
//
const [header, ...data] = getExcel();
```

async/await보다 .then()이 깔끔한 경우

```ts
const name = (await getProduct()).name;
//
const name = getProduct().then(product => product.name);
```