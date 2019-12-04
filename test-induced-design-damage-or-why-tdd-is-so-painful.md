# 테스트-유발 디자인 손상, TDD는 왜 고통스러울까

이 글은 Vladimir Khorikov의 블로그에 쓰인 [Test-induced design damage or why TDD is so painful](https://enterprisecraftsmanship.com/2015/06/29/test-induced-design-damage-or-why-tdd-is-so-painful/) 을 번역한 글입니다. 원작자의 허락을 받았습니다.

---

저는 TDD의 주제로 한 몇 개의 글을 쓸 것입니다. 수년에 걸쳐, TDD를 적용하고 테스트를 작성하는 방법에 대해 일반적인 결론을 얻었기 때문에 여러분에게 도움이 될 것으로 기대합니다. 저의 경험에서 비롯된 몇 가지 포인트를 예시와 함께 설명할 것입니다.

-   테스트-유발 디자인 손상, TDD는 왜 고통스러울까
-   고통 없이 TDD를 하는 방법
-   통합 테스트 또는 밤에 편히 자는법
-   가장 중요한 TDD 규칙
-   Stubs vs Mocks
-   TDD 모범 사례

## 테스트-유발 디자인 손상

시작하기 전에 짚고 넘어갈 게 있습니다. TDD (Test-Driven Development)는 단위 테스트 작성과 같은 것이 아닙니다. TDD는 단위 테스트 작성을 요구하는 동시에, 테스트 될 코드에 앞서 단위 테스트를 작성하는 테스트 우선 접근법을 강조합니다. 이 글에서는 TDD와 테스트 작성에 대해 전반적으로 다룰 것입니다. 저는 편의를 위해 두 개념 모두 "TDD"라고 언급할 것이지만, 정확하지는 않습니다.

TDD에 집착하는 것처럼 느껴본 적이 있습니까? 아니면 테스트를 나중에 작성하는 것만으로도 해결할 수 있는 것보다 더 많은 문제가 생깁니까? 코드를 테스트할 수 있도록 만들기 위해서는 먼저 코드를 엉망으로 만들 필요가 있다는 것을 알아차렸습니까? 그리고 테스트 자체가 이해하기 어렵고 유지하기 어려운 진흙탕처럼 보이나요? 걱정하지 마세요, 당신은 혼자가 아닙니다. Martin Fowler, Kent Beck, David Heinemeier Hansson이 TDD에 대한 그들의 견해와 경험을 표현하고자 TDD가 죽었는지 여부([Is TDD Dead?](https://martinfowler.com/articles/is-tdd-dead/))에 대한 일련의 토론이 있었습니다.

이 토론에서 가장 흥미로운 주제는 David Hansson이 소개한 테스트-유발 디자인 손상의 개념입니다. 코드를 테스트할 때 코드가 손상되는 것을 피할 수는 없다는 맥락의 개념입니다.

예를 들어 보겠습니다.

```cs
[HttpPost]
public HttpResponseMessage CreateCustomer([FromBody] string name)
{
  Customer customer = new Customer();
  customer.Name = name;
  customer.State = CustomerState.Pending;
  var repository = new CustomerRepository();
  repository.Save(customer);
  var emailGateway = new EmailGateway();
  emailGateway.SendGreetings(customer);
  return Ok();
}
```

이 메소드는 매우 간단하고 자신을 잘 설명합니다. 동시에, 테스트할 수 없습니다. 격리되어 있지 않아 단위 테스트를 수행할 수 없습니다. 테스트를 실행할 때마다 데이터베이스를 건드리는 것은 너무 느리고, 실제로 이메일을 보내고 싶지도 않을 것입니다.

비즈니스 로직을 독립적으로 테스트하려면 클래스의 의존성을 외부로부터 주입받아야 합니다.

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

이런 접근법으로 메소드의 비즈니스 로직을 외부 의존성과 분리하고 적절하게 테스트할 수 있습니다. 다음은 메소드의 정확성을 확인하기 위한 전형적인 단위 테스트입니다.

```cs
[Fact]
public void CreateCustomer_creates_a_customer()
{
  // Arrange
  var repository = new Mock<ICustomerRepository>();
  Customer savedCustomer = null;
  repository
    .Setup(x => x.Save(It.IsAny<Customer>()))
    .Callback((Customer customer) => savedCustomer = customer);
  Customer emailedCustomer = null;
  var emailGateway = new Mock<IEmailGateway>();
  emailGateway
    .Setup(foo => foo.SendGreetings(It.IsAny<Customer>()))
    .Callback((Customer customer) => emailedCustomer = customer);
  var controller = new CustomerController(repository.Object, emailGateway.Object);
  // Act
  HttpResponseMessage message = controller.CreateCustomer("John Doe");
  // Assert
  Assert.Equal(HttpStatusCode.OK, message.StatusCode);
  Assert.Equal(savedCustomer, emailedCustomer);
  Assert.Equal("John Doe", savedCustomer.Name);
  Assert.Equal(CustomerState.Pending, savedCustomer.State);
}
```

친숙하게 보이나요? 저는 당신이 이런 것들을 충분히 만들어봤다고 확신합니다. 저도 그랬습니다.

물론, 이런 테스트는 옳다고 생각하지 않습니다. 간단한 동작을 테스트하기 위해서는 그 동작을 분리하기 위해 수많은 보일러플레이트 코드를 작성해야 합니다. Arrange 섹션의 크기가 얼마나 거대한지 보세요. Act와 Assert 섹션을 합쳐 5행인 것에 비해 11행이나 차지합니다.

## TDD는 왜 고통스러울까

이런 단위 테스트는 별다른 이유 없이 자주 깨집니다. 의존하는 인터페이스 중 하나의 시그니처를 약간 변경해야 합니다.

이런 테스트가 회귀 결함을 찾는 데 도움이 되나요? 몇몇 간단한 경우에는 그렇습니다. 하지만 종종 코드베이스를 리팩토링 할 때 충분한 확신을 주지 못하는 경우가 많습니다. 그 이유는 단위 테스트가 false-positive를 너무 많이 보고하기 때문입니다. 그들은 너무 깨지기 쉽습니다. 이내 개발자는 무시하기 시작합니다. 그것은 이상한 일이 아닙니다. 양치기 소년을 생각해보세요.
그렇다면 대체 왜 이런 일이 벌어질까요? 무엇이 테스트를 취약하게 하나요?

이유는 Mock입니다. Mocking 된 수많은 의존성이 있는 테스트에는 많은 유지보수가 필요합니다. 코드의 의존성이 커질수록, 코드베이스가 자라남에 따라 테스트를 실행하고 수정하는 데 더 큰 노력이 필요합니다.

코드에 외부 의존성이 없으면 유닛 테스트로 인해 디자인이 손상되지 않습니다. 이 점을 설명하기 위해 예제 코드를 준비했습니다.

```cs
public double Calculate(double x, double y)
{
  return x * x + y * y + Math.Sqrt(Math.Abs(x + y));
}
```

테스트하기 얼마나 쉬울까요? 이처럼 쉽습니다.

```cs
[Fact]
public void Calculate_calculates_result()
{
  // Arrange
  double x = 2;
  double y = 2;
  var calculator = new Calculator();
  // Act
  double result = calculator.Calculate(x, y);
  // Assert
  Assert.Equal(10, result);
}
```

혹은 더 간단하게

```cs
[Fact]
public void Calculate_calculates_result()
{
  double result = new Calculator().Calculate(2, 2);
  Assert.Equal(10, result);
}
```

이를 통해 다음과 같은 결론을 얻을 수 있습니다. 테스트-유발 디자인 손상의 개념은 Mock을 만들 필요성에 있습니다. 외부 의존성을 Mocking 할 때, 필연적으로 더 많은 코드를 작성하게 되며, 그 자체로 유지보수가 더 힘든 솔루션이 됩니다. 이 모든 요소가 유지보수 비용을 증가시키거나, 개발자에게 고통을 줍니다.

이 글이 유용했다면, 제 [Pragmatic Unit Testing Pluralsight](https://www.pluralsight.com/courses/pragmatic-unit-testing) 코스 또한 확인해주세요.

## 요약

자, 이제 우리는 TDD를 할 때 벌어지는 디자인 손상과 고통의 원인을 알게 되었습니다. 하지만 어떻게 그 고통을 완화할 수 있을까요? 뭔가 방법이 있을까요? 그것은 다음 포스트에서 다룰 것입니다.
