# 고통 없이 TDD를 하는 방법

이 글은 Vladimir Khorikov의 블로그에 쓰인 [How to do painless TDD](https://enterprisecraftsmanship.com/2015/07/06/how-to-do-painless-tdd/)를 번역한 글입니다. 원작자의 허락을 받았습니다.

---

지난주에 우리는 코드를 테스트하기 위해 디자인에 가져오는 피해, 소위 테스트-유발 디자인 손상과 관련된 문제의 근본적인 원인을 찾아냈습니다. 오늘 우리는 그 피해를 완화시킬 수 있는 방법, 즉 고통 없는 TDD를 어떻게 할 것인지 살펴볼 것입니다.

-   테스트-유발 디자인 손상, TDD는 왜 고통스러울까
-   **고통 없이 TDD를 하는 방법**
-   통합 테스트 또는 밤에 편히 자는법
-   가장 중요한 TDD 규칙
-   Stubs vs Mocks
-   TDD 모범 사례

## 고통 없는 TDD: Mock을 사용 할 것인가, 말 것인가?

그렇다면 코드를 손상시키지 않고 테스트 가능하게 유지할 수 있을까요? 당연히 그렇습니다. 당신은 그저 **Mocking을 전부 제거해야 할 뿐입니다.** 과장스럽게 들릴 것을 알고 있기에, 이제부터 설명을 해보겠습니다.

소프트웨어 개발은 ​​모두 타협의 문제입니다. 그것은 CAP 정리, 속도-비용-품질 삼각형 등 어디에나 존재합니다. 단위 테스트의 상황도 마찬가지입니다. 100 % 테스트 커버리지가 필요한가요? 일반적인 엔터프라이즈 어플리케이션을 개발할 경우, 아마도, 아닙니다. 당신에게 필요한 것은 모든 중요한 비즈니스 코드를 커버하는 견고한 테스트 모음입니다. 당신은 코드베이스의 사소한 부분을 단위 테스트하는 데 시간을 낭비하고 싶지 않을 것입니다.

좋아요, 하지만 Mock을 도입하지 않고도 중요한 비즈니스 로직을 테스트 할 수 없다면 어떻게 될까요? 이 경우, 외부 의존성이 있는 메소드에서 해당 로직을 추출해내야 합니다. [이전 글](https://github.com/icepeng/articles/blob/master/test-induced-design-damage-or-why-tdd-is-so-painful.md)의 메소드를 살펴보겠습니다.

```cs
public class CustomerController : ApiController
{
  private readonly ICustomerRepository _repository;
  private readonly IEmailGateway _emailGateway;
  public CustomerController(ICustomerRepository repository,
  IEmailGateway emailGateway)
  {
  _emailGateway = emailGateway;
  _repository = repository;
  }
  [HttpPost]
  public HttpResponseMessage CreateCustomer([FromBody] string name)
  {
    Customer customer = new Customer();
    customer.Name = name;
    customer.State = CustomerState.Pending;
    _repository.Save(customer);
    _emailGateway.SendGreetings(customer);
    return Ok();
  }
}
```

여기서 테스트 할 가치가 있는 로직은 무엇입니까? 저는 CreateCustomer 메소드의 첫 세 줄만 있다고 주장하겠습니다. Repository.Save 메소드는 중요하지만, Mock이 실제로 동작하는지 확인하는 데 도움이 안되며, 단지 CreateCustomer 메소드가 이를 호출하는지 확인할 뿐입니다. SendGreetings 메소드 또한 마찬가집니다.

따라서, 생성자에서 해당 라인들을 추출해야 합니다 (혹은, 더 복잡한 로직이 있으면 팩토리 내에서).

```cs
[HttpPost]
public HttpResponseMessage CreateCustomer([FromBody] string name)
{
  Customer customer = new Customer(name);
  _repository.Save(customer);
  _emailGateway.SendGreetings(customer);
  return Ok();
}
public class Customer
{
  public Customer(string name)
  {
  Name = name;
  State = CustomerState.Pending;
  }
}
```

이러한 코드의 단위 테스트는 간단한 일이 됩니다.

```cs
[Fact]
public void New_customer_is_in_pending_state()
{
  var customer = new Customer("John Doe");
  Assert.Equal("John Doe", customer.Name);
  Assert.Equal(CustomerState.Pending, customer.State);
}
```

이 테스트에는 Arrange 섹션이 없습니다. 이는 유지보수성이 높은 테스트의 신호입니다. 단위 테스트에서 수행하는 작업이 적을수록 유지보수가 쉽습니다. 물론 완전히 제거 할 수 있는 것은 아니지만, 일반적인 규칙 - Arrange 섹션을 가능한 작게 유지해야 한다 - 는 것은 그대로 유지됩니다. 이를 위해서는 단위 테스트에서 Mock 사용을 그만둬야 합니다.

하지만 컨트롤러는 어떨까요? 단위 테스트를 그냥 그만두면 될까요? 정확합니다. 컨트롤러는 이제 필수 로직을 아예 포함하지 않고, 단지 다른 액터들의 동작을 조정 할 뿐입니다. 그 로직은 이제 사소합니다. 이러한 로직을 테스트하면 이익보다도 많은 비용이 발생하므로, 시간을 낭비하지 맙시다.

## 코드의 유형

TDD를 고통스럽지 않게 만드는 방법에 관한 몇 가지 흥미로운 결론을 이끌어 냈습니다. 우리는 다음과 같은 특정 유형의 코드만을 단위 테스트해야합니다.

-   외부 의존성이 없는 코드
-   도메인을 표현하는 코드

외부 의존성이란 외부 세계의 상태에 의존하는 객체를 의미합니다. 예를 들어, 리포지토리는 데이터베이스의 데이터에 의존하고, 파일 관리자는 파일 시스템의 파일에 의존합니다. Mark Seemann은 자신의 저서인 Dependency Injection in .NET에서 이러한 의존성을 Volatile\*이라고 묘사합니다. 다른 유형의 의존성 (문자열, DateTime 또는 심지어 도메인 클래스)은 그렇게 취급되지 않습니다.

> Volatile\*: 휘발성. C에서 프로그램 외부 요인으로 값이 변할 수 있는 변수를 volatile int i = 0 과 같이 선언하는 것을 의미하는 것으로 보임.

다음은 다양한 유형의 코드를 도식화한 것입니다.

![codetype.png](https://miro.medium.com/max/607/1*FGfOIv1SscUsj8M_7X5k9w.png)

Steve Sanderson이 이 주제에 대한 [훌륭한 글](http://blog.stevensanderson.com/2009/11/04/selective-unit-testing-costs-and-benefits/)을 작성했으므로 더 자세한 내용을 알고싶다면 확인해보세요.

일반적으로, 도메인 로직을 포함하고 외부 의존성 (도식에서 "Mess" 영역)을 갖는 코드의 유지 관리 비용은 너무 비쌉니다. 우리가 처음에 본 컨트롤러 코드가 그렇습니다. 외부 의존성에 의존하고 동시에 도메인-특정 작업을 수행했기 때문입니다. 이와 같은 코드는 분리되어야 합니다. 도메인 로직을 도메인 객체에 배치하고 컨트롤러는 그들을 조절하고 통합하는 역할만 해야 합니다.

코드베이스의 나머지 3 가지 유형(도메인 모델, 사소한 코드, 컨트롤러)의 코드에서 도메인 관련 코드만 유닛 테스트해야합니다. 이곳이야말로 투자 수익을 극대화 할 수 있습니다. 이런 트레이드-오프를 당신이 해야 한다고 저는 주장합니다.

제 포인트는 다음과 같습니다.

-   모든 코드가 똑같이 중요하지는 않습니다. 당신의 어플리케이션에는 당신의 노력을 집중해야하는 중요한 비즈니스 파트 (도메인 모델)가 포함되어 있습니다.
-   도메인 모델은 자체적으로 표현되며, 외부 세계에 의존하지 않습니다. 즉, 테스트하기 위해 Mock을 사용할 필요가 없으며, 테스트를 할 필요도 없습니다. 따라서, 도메인 모델에 대한 단위 테스트는 쉽게 구현하고 유지보수 할 수 ​​있습니다.

이러한 방식을 따르면 견고한 단위 테스트 모음을 구축하는 데 도움이 됩니다. 단위 테스트의 목표는 100 % 테스트 커버리지가 아닙니다 (가끔 합리적이긴 합니다만). 목표는 변경 사항과 새로운 코드가 기존 기능을 깨트리지 않는다는 확신을 얻는 것입니다.

저는 오래 전에 Mock 사용을 그만뒀습니다. 그 후로 제가 작성한 코드의 품질은 떨어지지 않았을뿐만 아니라, 제가 작업하는 프로젝트의 코드베이스를 발전시키는데 도움되는 유지보수 가능하고 신뢰할 수 있는 단위 테스트 모음을 가지게 되었습니다. TDD는 제게 있어서 고통스럽지 않습니다.

이 가이드라인을 지킨다고 해서 테스트의 Mock을 완전히 제거 할 수는 없다고 주장 할 수도 있습니다. 이 예시에서 어쩌면 고객의 전자 메일을 고유하게 가져야만 할 수도 있습니다. 우리는 그러한 로직을 테스트하기 위해 리포지토리를 Mock해야합니다. 걱정하지 마세요, 다음 글에서 이 케이스(및 다른것들)에 대해 다루겠습니다.
