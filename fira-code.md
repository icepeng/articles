# Fira Code

저는 개발할 때 폰트로 [Fira Code](https://github.com/tonsky/FiraCode)를 사용합니다. 2017년쯤부터 사용해온 기억이 나니까 이제 3년이 다 되어가네요. 한번 설정해두고 잊고 살아왔는데 지난 9월에 v2가 릴리즈된걸 발견한 김에 글을 남겨봅니다.

![Fira Code v2](https://user-images.githubusercontent.com/285292/64554512-3e6bcd80-d344-11e9-83f7-265854b88646.png)

Fira Code의 README에서는 다음과 같은 주장을 합니다.

> Programmers use a lot of symbols, often encoded with several characters. For the human brain, sequences like ->, <= or := are single logical tokens, even if they take two or three characters on the screen. Your eye spends a non-zero amount of energy to scan, parse and join multiple characters into a single logical one. Ideally, all programming languages should be designed with full-fledged Unicode symbols for operators, but that’s not the case yet.

> 프로그래머는 종종 여러 개의 문자로 표현되는 기호를 사용합니다. ->, <= 또는 :=와 같은 문자열은 2~3개의 문자를 사용하지만, 사람에게는 하나의 논리적 토큰으로 인식됩니다. 여러분의 눈은 여러 문자를 하나의 논리적 문자로 스캔, 파싱 및 결합하기 위해 에너지를 소모하게 됩니다. 이상적으로, 모든 프로그래밍 언어는 각 연산자에 대응되는 유니코드 기호로 동작하게 만들어져야 하지만 아직 그렇지 않습니다.

그렇기에 Fira Code는 여러 개의 문자로 표현되는 토큰들을 마치 하나의 문자인 것처럼 합쳐서 보여주는 해결책을 제시합니다.

![Fira Code Javascript](https://github.com/tonsky/FiraCode/raw/master/showcases/javascript.png)

붉게 표시된 부분이 Fira Code의 특징을 보여줍니다. triple equal의 존재감이 강렬해서 double equal과 구분이 뚜렷하고, not equal의 표현도 더 직관적으로 느껴집니다. 16진법이 귀엽네요.

개인적으로는 JavaScript / TypeScript로 개발할 때 Arrow Function을 자주 사용하기 때문에 =>를 멋진 화살표로 만들어주는 게 마음에 들었습니다. 그리고 \n과 같이 escape를 위한 역슬래시 기호를 얇게 렌더링해서 실질적인 문자열을 읽는 데 도움을 주는 것도 편했습니다.

![Fira Code Stylistic Sets](https://github.com/tonsky/FiraCode/raw/master/showcases/stylistic_sets.png)

만약 부등호의 모양이 마음에 들지 않는다면 세팅을 변경해 특정 기능들을 켜고 끌 수 있습니다.

호불호가 매우 갈리는 게 재미있는 부분입니다. !== 같은 코드를 원본 텍스트를 유지하지 않고 바꿔버리는 점이 가장 크게 작용하는 것 같습니다. 저는 코드를 텍스트의 나열보다는 이미지로 인식하는 편이기 때문에 지금까지 애용하고 있지만, 제 에디터를 보고 기겁하는 동료 개발자의 반응을 보는 것도 재미있습니다.

이 폰트가 마음에 드셨다면, 에디터에 적용하는 것은 간단합니다. [VS Code Instructions](https://github.com/tonsky/FiraCode/wiki/VS-Code-Instructions)를 보시면 알 수 있듯이 폰트를 설치하고, 설정에서 기능 하나만 켜주면 그만입니다. Font Ligature를 지원하는 에디터는 전부 적용할 수 있다고 보면 됩니다.