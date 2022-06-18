# 1.3.4 원칙과 패턴

## 개방 폐쇄 원칙(OCP)
- 개방 폐쇄 원칙(Open-Closed Principle)을 이용하면 지금까지 해온 리팩토링 작업의 특징과 최종적으로 개선된 설계와 코드의 장점이 무엇인지 효과적으로 설명할 수 있다.

> 개방 폐쇄 원칙은 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다. (예를 들면) UserDao는 DB 연결 방법이라는 기능을 확장하는 데 열려 있고, UserDao에 전혀 영향을 주지 않고도 얼마든지 확장할 수 있게 되어 있다. 동시에 UserDao 자신의 핵심 기능을 구현한 코드는 그런 변화에 영향을 받지 않고 유지할 수 있으므로 변경에는 닫혀 있다고 말할 수 있다.

- 인터페이스를 통해 제공되는 확장 포인트는 확장을 위해 개방되어 있지만, 인터페이스를 이용하는 클래스는 자신의 변화가 불필요하게 일어나지 않도록 굳게 폐쇄되어 있다.

```
객체지향 설계 원칙(SOLID)

- 객체지향 프로그래밍 언어의 종류도 다양하고 객체지향 기술을 받아들이고 적용하는 관점과 기법도 조금씩 차이가 있다.
- 객체지향 설계원칙은 객체지향의 특징을 잘 살릴 수 있는 설계의 특징을 말한다.
- 원칙이라는 것은 어떤 상황에서든 100% 지켜져야 하는 절대적인 기준이라기보다는, 예외는 있겠지만 대부분의 상황에 잘 들어맞는 가이드라인과 같은 것이다.

- 디자인 패턴은 특별한 상황에서 발생하는 문제에 대한 구체적인 솔루션이라고 한다면, 객체지향 설계 원칙은 좀더 일반적인 상황에서 적용 가능한 설계 기준이라고 볼 수 있다.

- SOLID는 아래 5가지 원칙의 첫 글자를 따서 만든 단어다.

  1. SRP(The Single Responsibility Principal)
  : 단일 책임 원칙

  2. OCP(The Open Closed Principal) 
  : 개방 폐쇄 원칙

  3. LSP(The Liskov Substitution Principal)
  : 리스코프 치환 원칙

  4. ISP(The Interface Segretaion Principal)
  : 인터페이스 분리 원칙

  5. DIP(The Dependency Inversion Principal)
  : 의존 관계 역전 원칙
```
* 추천 자료
- [로버트 마틴이 정리한 객체지향 설계 원칙인 SOLID 소개](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
- [UML for Java Programming](https://www.csd.uoc.gr/~hy252/references/UML_for_Java_Programmers-Book.pdf)
- [밥 마틴 - 소프트웨어 개발의 지혜:원칙, 디자인 패턴, 실천 방법](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=471997)

