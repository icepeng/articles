# Redux-Saga vs. Redux-Observable

프런트엔드 상태 관리의 중요성에 대해 열심히 설파하고 다니던 어느 날, 저는 다음과 같은 질문을 받게 되었습니다.
> Redux-Saga나 Redux-Observable같은 미들웨어를 사용하는 이유와 어떤 것을 사용할지 고르는 방법을 듣고 싶습니다. 요새 Redux를 쓰면 다들 Redux-Saga를 쓰니 자연스럽게 따라가게 되는데, 각각 어떤 경우에 사용하는 게 좋은지 알고 싶습니다.

남들이 쓰니까 그냥 가져다 쓰는 게 아닌, 근본부터 정확히 파악하고자 하는 게 너무 좋은 질문이었습니다. 하지만 당시에는 제가 정확히 설명해 드리기에는 공부가 부족하다 느꼈기에 다음에 글로 정리하겠다는 약속을 했었습니다. 알아보면서 저 또한 많은 공부가 되었고, 마침내 글로 정리해볼 수 있겠다는 생각이 들어 작성하기 시작했습니다.

들어가기에 앞서, 저는 Angular의 [NgRx](ngrx.io) 라이브러리를 주로 사용했고, 여기서 부작용 처리를 위해 사용하는 Effects 모듈이 Redux-Observable과 매우 유사합니다. 때문에 어느 정도 치우친 견해가 있을 수 있다는 점을 미리 알려드립니다.

## 목적성

두 라이브러리 모두 동일한 목적을 가지고 작성되었습니다. Redux에서 **비동기** 액션을 처리하는 것입니다. 더 정확히는 **부작용**을 관리하는 게 목적입니다. 부작용을 관리하는것이 왜 중요할까요? [Redux의 3가지 원칙](https://redux.js.org/introduction/three-principles) 중 하나인 **변경은 순수 함수로만 이루어져야 한다**는 것이 그 이유입니다.

Redux의 동작 방식을 아주 간략하게 묘사하면

1. Action을 dispatch한다
2. 순수 함수인 reducer 조각들이 action.type에 따라 변경을 수행하고, Store를 업데이트 한다

만약 여기서 API를 호출한 결과를 Store에 저장하고 싶다고 가정합시다. API 호출은 당연히 비동기로 동작할것이고, 호출할 때마다 다른 값이 돌아올 수도 있고, 심지어 실패할 수도 있습니다.

(비동기가 부작용의 대표적인 예시라는 점 설명)

(액션을 트리거/성공/실패 3가지로 나눠서 동작하는 방식 설명)

## Redux-Saga

### Generator

Generator Function은 매우 흥미로운 면이 많은 기능이지만, 실제로 유용하게 활용되는 경우는 찾아보기 힘듭니다. 저는 Redux-Saga를 알아보면서 Generator Function을 문제 해결에 굉장히 잘 활용했다고 느꼈습니다.

더 깊이 들어가려면 일단 Generator Function에 대한 이해가 필요합니다. Generator Function을 아주 간략하게 설명하자면 `return`을 여러번 할 수 있는 함수입니다.

(Generator 안팎으로 값을 주고받는 과정 설명)

(아래는 async-await이 Generator 응용의 한 종류라는 것을 설명하기 위한 예시)
```js
function async(gen) {
  const it = gen();

  function rec(res) {
    if (res.done) {
      return res.value;
    }
    return res.value.then(x => rec(it.next(x)));
  }

  return rec(it.next());
}

const result = async(function*() {
  const a = yield Promise.resolve(1);
  const b = yield Promise.resolve(3);

  console.log(a); // 1
  console.log(b); // 3

  return 10;
});

result.then(x => console.log(x)); // 10
```

(Generator를 이용해 부작용을 관리하는 방법 설명)
(Worker & Watcher 구조 설명)

## Redux-Observable

### RxJS

(RxJS에 대한 간략한 소개)

(Redux-Observable에서 어떻게 활용하는가, 실제 미들웨어 구현은 100라인 가량에 불과함)

(어째서 RxJS가 필요한 기능을 이미 전부 제공해주고 있는지에 대한 설명)

## 결론

|         | Redux-Saga | Redux-Observable |
| :-----: | :--------: | :--------------: |
|  핵심 요소  | Generator  |    Observable    |
|  학습 속도  |     빠름     |        느림        |
| 지식의 범용성 |     낮음     |        높음        |

Redux-Saga는 "Redux에서 부작용을 관리한다"는 목적에 정확히 들어맞는 기능 세트를 제공합니다. 반면 Redux-Observable은 RxJS가 따로 설치되어야 하는 점을 제외하면 자체적으로 제공해주는 것은 거의 없습니다. 원래 목적이였던 부작용 관리에 필요한 요소들을 RxJS가 이미 제공해주고 있기 때문이죠.

비유하자면 다음에 볼 시험에 대비해 기출문제를 잔뜩 뽑아서 유형별로 최적화된 풀이 패턴을 미리 만들어두는게 Redux-Saga, 기본 원리를 확실하게 익혀서 어떤 문제가 주어지더라도 시간을 들이면 풀 수 있는게 Redux-Observable 이라고 생각합니다. 당연히 시험을 잘 본다는 목적만을 위해서는 전자가 훨씬 효율적입니다. 하지만 후자에도 나름의 가치가 있다는 것은 확실합니다.

결론적으로 저는 다음과 같이 제안합니다.

**Redux-Saga**
* Redux를 사용하며 어느정도 개발된 프로젝트에 비동기 처리만을 추가할 경우
* Redux에 입문한지 얼마 되지 않은 경우
  * 입문 단계에서 참고할 수 있는 한국어 자료가 많다는 점은 매우 큰 장점입니다
* Observable을 사용할 특별한 이유가 없는 경우
  * 특별한 이유가 없는 한 외부 의존성은 적을수록 좋다고 생각합니다
  * 하지만 내가 친숙하다, 잘 활용할 수 있다는 자신감은 특별한 이유가 되기에 충분합니다

**Redux-Observable**
* Reactive Programming에 대한 본인과 팀원들의 이해도가 높은 경우
* 이미 프로젝트에서 RxJS를 활용한 코드가 존재하는 경우
  * Angular로 작성된 프로젝트는 사실상 확정적으로 사용중일 것
* Reactive Programming을 공부해보고 싶은 경우
  * 회사에서 사용한다면 팀원들의 공감대를 이끌어내야 함
  * 첫 도입은 이왕이면 덜 중요한 백오피스나 개인적인 프로젝트로...