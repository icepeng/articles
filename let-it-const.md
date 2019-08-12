# Let it Const (Draft)

Javascript / Typescript 위에서 함수형 프로그래밍을 시작하려면, `let`을 전부 `const`로 바꾸는 연습부터 할 필요가 있다.

왜? 함수형 패러다임에서는 변수를 사용하지 않는다. `const a = 3`과 같이 한번 선언되었으면, 프로그램이 실행되는 동안 `a`는 영원히 3이다.

이 제약이 불편하다고 생각할 수 있지만, 실제로는 큰 이점을 가져다준다.

* 디버깅할 때, 값의 변화를 추적할 필요가 없다. 한번 선언된 값은 변하지 않으므로, 관심있는 값을 한번 찍어보면 그만이다.
* 코드가 명료해진다. 이런 제약에 의해 변수에 들어있는 "값" 자체보다는 그것을 변환하는 "과정", 즉 논리에 100% 집중할 수 있게 된다.

## map/reduce

```
function listDouble(arr: number[]) {
    const result = [];
    for (let item of arr) {
        result.push(item * 2);
    }
    return result;
}
```

```
function listDouble(arr: number[]) {
    return arr.map(item => item * 2);
}
```

```
function sum(arr: number[]) {
    const result = 0;
    arr.forEach(item => {
        result += item;
    });
    return result;
}
```

```
function sum(arr: number[]) {
    retrun arr.reduce((acc, item) => acc + item, 0);
}
```

## 삼항 연산자

## 메소드 분리

## 고차 함수

## 기타 잡기술