# Let it Const

Javascript / Typescript 위에서 함수형 프로그래밍을 시작하려면, `let`을 전부 `const`로 바꾸는 연습부터 할 필요가 있다.

왜? 순수 함수형 패러다임에서는 변수를 사용하지 않는다. `const a = 3`과 같이 한번 선언되었으면, 프로그램이 실행되는 동안 `a`는 영원히 3이다.

이 제약이 불편하다고 생각할 수 있지만, 실제로는 큰 이점을 가져다준다.

- 디버깅할 때, 값의 변화를 추적할 필요가 없다. 한번 선언된 값은 변하지 않으므로, 관심 있는 값을 한번 찍어보면 그만이다.
- 코드가 명료해진다. 제약 때문에 변수에 들어 있는 "값" 자체보다는 그것을 변환하는 "과정", 즉 논리에 100% 집중할 수 있게 된다.
- Node 환경에서는 의미 없지만, 모든 값이 readonly, 읽기 전용이라는 것은 멀티스레드 환경에서 큰 이점을 가져다준다. 공유 변수로 인한 문제가 없기 때문이다.

## 진정한 Const를 위해

배열 / 오브젝트의 경우에는 제약 조건이 더 추가된다.

```js
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

위의 모든 코드는 에러가 발생하지 않는다. `const`로 선언했음에도, 내부의 값은 여전히 변경할 수 있다.

왜냐하면, JavaScript는 `Objects` 유형은 해당하는 메모리에 **값**이 아닌 **주소**가 할당되기 때문이다. (Objects에는 객체, 배열, 함수, Map, Set, 등등...이 포함된다) `obj.low = 0;` 과같이 하더라도 `obj`의 주소 자체는 변하지 않았기 때문에, 여전히 `const`의 정의에 위배되지 않는 것이다.

따라서, 우리는 겉보기에만 `const`가 아닌 진정한 `const`를 추구할 것이다. 이를 달성하기 위한 첫 번째 규칙.

**`=` 연산자를 사용할 때, 좌변에는 무조건 `const`가 있어야 한다!**

`=` 연산자를 더는 대입문으로 생각하지 말자. 선언문으로 생각하면 된다. 선언한다는 것은, 새로운 값을 만든다는 것이므로, 항상 `const a = 10;`과 같이 좌변에 `const`가 따라오게 될 것이다. 이것으로 많은 실수를 피할 수 있다.

두 번째 규칙.

**내부 값을 변경하는 메소드를 사용하지 말 것.**

JavaScript의 내장 함수 중에 주소를 그대로 두고, 내부 값을 조작하는 것들이 존재한다. 예를 들어, `Array.push()`. 이것들은 개발자가 의식하지 못한 사이 우리들의 Const 세계를 부수려고 시도하기 때문에 더 위험하다. 이들을 피하기 위해서는 위험한 메소드들을 외우는 수밖에 없다.

- Array.push
- Array.splice
- Array.sort
- ...여기저기 더 많다.

확실하지 않을 때는, 구글에 "is \_\_\_ immutable" 이라고 검색해보는 것이 좋다.

이제 정말로 모든 준비를 마쳤다. 출발하자.

## map/filter/reduce

`Array.map()`, `Array.filter()`, `Array.reduce()`는 ES6부터 추가된 메소드로, 배열을 다루는 데 있어 유용한 방법을 제시해준다.

### Array.map

**Let-code**
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

**Const-code**
```ts
function listDouble(arr: number[]) {
  return arr.map(item => item * 2);
}
```

`Array.map()`을 활용해 단 한 줄로 목표를 달성했다.

### Array.filter

**Let-code**
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

`Array.push()`를 없애려면 조건에 따라 배열의 일부만 남겨야 한다. `Array.filter()`가 원하는 기능을 제공해준다.

**Const-code**
```ts
function getPositives(arr: number[]) {
  return arr.filter(item => item > 0);
}
```

### Array.reduce

**Let-code**
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

여기서 짚고 넘어갈 점. `for-of`문이나 `Array.forEach()`를 사용하는 순간, 이미 뭔가 잘못되었을 확률이 높다. 이들은 그 자체로 let의 존재를 암시하고 있기 때문이다. 다른 방법이 없나 생각해보자.

**Const-code**
```ts
function sum(arr: number[]) {
  return arr.reduce((acc, item) => acc + item, 0);
}
```

reduce는 배열 관련 메소드 중 가장 강력하다. 위의 map, filter 모두 reduce로 동일한 기능을 구현할 수 있다. 하지만 그렇기 때문에 reduce를 사용할 때는 정말 다른 방법으로 해결할 수 없는지 고민해봐야 한다. reduce에 중독되어 map, filter로 해결할 수 있음에도 괜히 코드가 장황해지는 경우가 종종 발견된다.

위 3가지 메소드 모두 모두 인자로 함수를 전달했다. 즉, "함수를 인자로 받아서 적절히 배열에 적용해주는 함수"인 것이다. 고차 함수의 활용 중 가장 자주 접하게 되는 예시이다.

## 삼항 연산자

let - if에서 단순한 형태라면 `const a = condition ? x : y`와 같이 엮어버리면 된다. 삼항 연산자가 가장 유용해지는 순간이라고 생각한다.

**Let-code**
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

**Const-code**
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

하지만 이 방식에 중독되면 안 된다. 단 1개의 조건문, 2개의 값만 존재할 때 이 방법을 활용하는 것이 좋다. 조건이 늘어나면 다른 방법을 시도해보자.

## 메소드 분리

let - if로 된 로직을 if - return 하는 별도 함수로 뜯어낼 수 있다.

**Let-code**
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

**Const-code**
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

이런 리팩토링은 간단하게 수행할 수 있다. 그리고 이 정도만 하더라도 충분히 유용하다. 하지만 요구되는 조건이 10가지를 넘어가기 시작한다면? 그리고 동일한 목적에 Action의 종류만 달라진 `applyAction`의 변종이 마구 생겨난다면? 더 좋은 방법은 없을까?

## 고차 함수

Array.map과 비슷하게, 본인이 필요한 대로 "함수를 받는 함수"를 구성해서 쓰면 훨씬 더 범용성이 높은 함수를 작성할 수 있다.

**Normal function**
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

**Higher order function**
```ts
interface User {
  username: string;

  password: string;

  address: string;
}

// Validator는 조건식과 에러 메세지로 구성되어 있다
interface Validator<T> {
  constraint: (model: T) => boolean;
  message: string;
}

// validatorFactory는 Validator의 배열을 받아 (User) => Error Message 함수 1개로 합쳐준다
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

이렇게 변경된 코드의 강점은 User가 아닌 다른 객체들에 대한 Validator도 생성할 때 드러난다. 만약 Company, Product, Client, ... 20개의 객체에 대한 Validation function을 작성해야 한다고 해보자. 그리고 Normal function과 같은 형태로 20개 각각 작성했다고 생각해보자.

여기서 갑자기 "처음 발견된 에러 1개 말고, 전체를 배열로 돌려주세요."라는 요청이 들어온다면? 당신은 20개의 함수를 전부 수정해야 하고, 테스트하고, 야근하게 될 것이다. 하지만 Higher order function과 같이 작성했다면 그럴 걱정이 없다. `validatorFactory()` 하나만 적절히 수정해주면 그만이다.

```ts
// 수정 전
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

// 수정 후
function validatorFactory<T>(validators: Validator<T>[]) {
  return (model: T) => {
    const matchedValidators = validators.filter(validator =>
      validator.constraint(model),
    );
    if (matchedValidators.length === 0) {
      return undefined;
    }
    return matchedValidators.map(validator => validator.message);
  };
}
```
매우 간단한 수정으로 기능 변경이 완료되었고, 당신은 정시 퇴근을 지켜냈다.

이게 가능한 이유를 알아보자. Validator 함수는 분석해보면 다음 3가지 요소로 구성되어 있다.

* 검증할 객체의 타입
* 검증할 조건식 & 메세지 목록
* 결과를 합쳐서 전달하는 방법

여기서 앞의 2개는 검증할 객체에 따라 유동적으로 변할 수 있다. 하지만 결과를 합쳐서 전달하는 방법은 Validator라면 전부 동일하게 동작해야만 한다! 그렇기 때문에 해당 부분만 따로 분리해서 고차 함수로 작성하고, 나머지 2개의 요소는 input을 통해 전달받는 것이다.

## 기타 잡기술

아래는 let -> const에 한정하지 않고, 코드를 짧게 쓰기 위한 각종 잡기술을 정리한다.

### Default & Nullish check
* default 값 - || 혹은 ?? 사용
* nullish 체크 - && 혹은 Optional Chaining 사용

?? 연산자와 Optional Chaining은 현재 [TC39](https://github.com/tc39/proposals) Stage 3 Proposal에 올라와있다. Stage 3는 앞으로의 JavaScript에 포함되는 것이 확정적이기 때문에 미리 배워두면 좋다고 생각한다. 그 때문에 TypeScript에서는 3.7버전부터 미리 지원하기 시작한 기능이다.

??가 &&보다 더 안전하다. ??는 nullish 체크이기 때문에 `null`, `undefined`만 우측 값을 평가하는 반면, ||는 falsy 체크이기 때문에 `0`, `''` 까지도 범위에 포함하는 문제가 있다. 만약 `null`, `undefined`만을 체크하는 목적으로 사용했다면 예상치 못한 버그가 발생할 수 있다. 이는 Optional Chaining에도 동일하게 적용된다.

**Long Code**
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
```

**Short Code, not safe**
```ts
const score = getScore() || 0;

const user = getUser();
const name = user.profile && user.profile.name;
```

**Shorter Code, safe**
```ts
// TypeScript 3.7 or TC39 Stage 3 Proposal
const score = getScore() ?? 0;

const user = getUser();
const name = user.profile?.name;
```

### destructuring, rest operator 응용

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

### async/await과 .then()을 함께 사용하는게 깔끔한 경우

```ts
const name = (await getProduct()).name;
//
const name = await getProduct().then(product => product.name);
```

---

긴 글 읽어주셔서 고맙습니다. 부족한 내용, 질문 등은 Issue로 남겨주시면 최대한 피드백 드릴 수 있도록 하겠습니다.