# 1.8 XML을 이용한 설정

- 서론 
  - DaoFactory는 DI의 동작원리를 잘 활용한 독립적인 오브젝트 팩토리 클래스로 시작했지만, DI 컨테이너인 스프링을 도입하면서부터 애노테이션을 추가해서 DI 작업에 참고하는 일종의 참고정보로 사용되고 있다.

  - 범용 DI 컨테이너를 사용하면서 오브젝트사이의 의존정보는 일일이 자바 코드로 만드는 작업은 번거롭다.

  - DaoFactory은 대부분 틀에 박힌 구조가 반복된다. DI 구성이 바뀔 때마다 자바 코드를 수정하고 클래스를 다시 컴파일하는 것은 귀찮은 작업이다.

- XML

  - 스프링은 DaoFactory와 같은 자바 클래스를 이용하는 것 외에도 다양한 방법을 통해 DI 의존관계 설정정보를 만들 수 있다. 가장 대표적인 것이 XML이다.

  - XML은 단순한 텍스트 파일이기 때문에 다루고 이해하기 쉽다. 컴파일과 같은 별도의 작업이 요구되지 않는다.

  - 환경이 달라져서 오브젝트의 관계가 바뀌는 경우에도 빠르게 변경사항을 반영할 수 있다.
  
  - 스키마나 DTD를 이용해서 정해진 포맷을 따라 작성됐는지 손쉽게 확인할 수 있다.

<br />

## 1.8.1 XML 설정

- 스프링의 애플리케이션 컨텍스트는 XML에 담긴 DI 정보를 활용할 수 있다.

- DI 정보가 담긴 XML 파일은 < beans > 를 루트 엘리먼트로 사용한다.

- 이름에서 알 수 있듯이 beans 안에는 여러 개의 bean을 정의할 수 있다.

- XML 설정은 @Configuration과 @Bean이 붙은 자바 클래스로 만든 설정과 내용이 동일하다.

- @Configuration은 beans, @Bean은 bean에 대응해서 생각하면 이해하기 쉽다.

- 하나의 @Bean 메소드를 통해 얻을 수 있는 빈의 DI 정보
  
  - 빈의 이름
   
    @Bean 메소드 이름이 빈의 이름이다. 이 이름은 getBean()에서 사용된다.

  - 빈의 클래스

    빈 오브젝트를 어떤 클래스를 이용해서 만들지를 정의한다.

  - 빈의 의존 오브젝트

    빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어준다. 의존 오브젝트는 하나의 빈이므로 이름이 있고, 그 이름에 해당하는 메소드를 호출해서 의존 오브젝트를 가져온다. 의존 오브젝트는 하나 이상일 수 있다.

- XML에서 bean을 사용해도 이 세가지 정보를 정의할 수 있다.

- ConnectionMaker처럼 더 이상 의존하고 있는 오브젝트가 없는 경우에는 의존 오브젝트 정보는 생략할 수 있다.

- XML은 자바 코드처럼 유연하게 정의될 수 있는 것이 아니므로, 핵심 요소를 잘 짚어서 그에 해당하는 태그와 애트리뷰트가 무엇인지 알아야 한다.

### connectionMaker() 전환

- connectionMaker()로 정의되는 빈은 의존하는 다른 오브젝트는 없으므로 DI 정보 세가지 중  빈의 이름과 빈 클래스(이름) 두 가지를 bean 태그의 id와 class 애트리뷰트를 이용해 정의할 수 있다.

표 1-2 클래스 설정과 XML 설정의 대응항목

<img src="https://velog.velcdn.com/images%2Fpu1etproof%2Fpost%2F66a057ba-91c0-445c-8e2b-f78aebc60d24%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-10-28%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.19.16.png">

<br />

- DaoFactory의 @Bean 메소드에 담긴 정보를 1:1로 XML의 태그와 애트리뷰트로 전환해주기만 하면 된다. 

  단, bean 태그의 class 애트리뷰트에 지정하는 것은 자바 메소드에서 오브젝트를 만들 때 사용하는 클래스 이름이라는 점을 주의해야하고 메소드의 리턴 타입을 class 애트리뷰트에 사용하지 않도록 해야한다.

  - class 애트리뷰트에 넣을 클래스 이름은 패키지까지 모두 포함해야 한다.

  - IDE의 자바 에디터에서는 import 문을 직접 작성하지 않아도 코드 내에서 클래스 이름만 가지고 손쉽게 패키지 정보를 가져올 수 있는데, XML에서는 이를 일일이 적어야 한다는 점이 처음에는 번거롭게 느껴질 것이다.

- @Bean은 bean 태그로, 메소드 이름은 id 애트리뷰트로, new에 사용하는 클래스 이름은 class 애트리뷰트로 대응해서 전환해주면 된다.

리스트 1-35. connectionMaker() 메소드의 bean 태그 전환
```Java
@Bean                               // <bean
public ConnectionMaker
connectionMaker(){                  // id="connectionMaker"
    return new DConnectionMaker();  // class="springbook...DConnectionMaker" />
}
```

- XML 설정파일의 bean 태그를 보면 @Bean 메소드에서와 같은 작업이 일어나겠구나라고 떠올릴 수 있으면 좋을 것이다.

- DI 컨테이너는 bean 태그의 정보를 읽어서 connectionMaker()와 같은 작업을 진행한다.

<br />
<hr />
<br />

### userDao() 전환

- userDao에는 DI 정보의 세 가지 요소가 모두 들어 있다. 관심을 가질 것은 수정자 메소드를 사용해 의존관계를 주입해주는 부분이다.

- 스프링 개발자가 수정자 메소드를 선호하는 이유 중에는 XML로 의존관계 만들 때 편리하다는 점이다.

- 자바빈의 관례를 따라 수정자 메소드는 프로퍼티가 되고 프로퍼티 이름은 메소드 이름에서 set을 제외한 나머지 부분을 사용한다.

  예를 들어 오브젝트에 setConnectionMaker()라는 이름의 메소드가 있다면 connectionMaker라는 프로퍼티를 갖는다고 할 수 있다.

- XML에서는 property태그를 사용해 의존 오브젝트와의 관계를 정의한다.

- property 태그는 name과 ref라는 두 개의 애트리뷰트를 갖는다. 
  
  - name : 프로퍼티의 이름이고 프로퍼티 이름을 통해 수정자 메소드를 알 수 있다.

  - ref : 수정자 메소드를 통해 주입해줄 오브젝트의 빈 이름이다. DI할 오브젝트도 역시 빈이다.

  @bean 메소드에서라면 다른 @Bean 메소드를 호출해서 주입할 오브젝트를 가져온다.
  
  ```Java
    userDao.setConnectionMaker(connectionMaker());
  ```

  - userDao.setConnectionMaker()는 userDao 빈의 connectionMaker 프로퍼티를 이용해 의존관계 정보를 주입한다는 의미이다.

  - 메소드의 파라미터로 넣는 connectionMaker()는 connectionMaker() 메소드를 호출해서 리턴하는 오브젝트를 주입하는 의미이다.

  - 두 가지 정보를 property의 name 애트리뷰트와 ref 애트리뷰트로 지정해주면 된다.

  <br />
 ```Java
    userDao.setConnectionMaker(connectionMaker());
  ```
  ```XML
  <property name="connectionMaker" ref="connectionMaker" />
  ```

  <br />

### XML의 의존관계 주입 정보

리스트 1-37. 완성된 XML 설정정보
```XML
<beans>
    <bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
    <bean id="userDao" class="springbook.user.dao.UserDao">
      <property name="connectionMaker" ref="connectionMaker">
    </bean>
</beans>
```
  
 - property 태그의 name과 ref는 그 의미가 다르므로 이름이 같더라도 어떤 차이가 있는지 구별할 수 있어야 한다.

   - name 애트리뷰트
   
       DI에 사용할 수정자 메소드의 프로퍼티 이름
    
    - ref 애트리뷰트
      
       주입할 오브젝트를 정의한 빈의 ID

 - 보통 프로퍼티 이름과 DI 되는 빈의 이름이 같은 경우가 많다.

 - 프로퍼티 이름은 주입할 빈 오브젝트의 인터페이스를 따르는 경우가 많고, 빈 이름도 인터페이스 이름을 사용하는 경우가 많기 때문이다.

 - 바뀔 수 있는 클래스 이름보다는 대표적인 인터페이스 이름을 따르는 편이 자연스럽다.

 - 프로퍼티 이름이나 빈의 이름은 인터페이스 이름과 다르게 정해도 상관없다. 

 - 빈의 이름을 바꾸는 경우 그 이름을 참조하는 다른 빈의 property 태그에 ref 애트리뷰트의 값도 함께 변경해주어야 한다.

 - 만약 connectionMaker 빈을 myConnectionMaker 라는 이름으로 변경했다면 userDao 빈의 connectionMaker 프로퍼티 ref 값도 따라서 변경해줘야 한다.

리스트 1-38. 빈의 이름과 참조 ref의 변경
```XML
<beans>
  <bean id="myConnectionMaker" class="springbook.user.dao.DConnectionMaker" />

  <bean id="userDao" class="springbook.user.dao.UserDao">
    <property name="connectionMaker" ref="myConnectionMaker" />
  </bean>
</beans>
```

<br />

- 같은 인터페이스를 구현한 의존 오브젝트를 여러 개 정의해두고 그중에서 원하는 걸 골라서 DI 하는 경우도 있다. 이때는 각 빈의 이름을 독립적으로 만들어두고 ref 애트리뷰트를 이용해 DI 받은 빈을 지정해주면 된다.

- 리스트 1-39는 LocalDB, TestDB와 ProductionDB를 사용하는 ConnectionMaker 인터페이스 구현 클래스를 각각 정의해두고 DAO에서 하나를 선택해서 사용할 수 있도록 구성한 XML 설정이다. 이때는 같은 인터페이스를 구현한 빈이 여러 개이므로 빈의 이름을 의미 있게 구분해서 정해줄 필요가 있다.

리스트 1-39. 같은 인터페이스 타입의 빈을 여러 개 정의한 경우
```XML
<beans>
  <bean id="localDBConnectionMaker" class="...LocalDBConnectionMaker" />
  <bean id="testDBConnectionMaker" class="...TestDBConnectionMaker" />
  <bean id="productionDBConnectionMaker" class="...ProductionDBConnectionMaker" />

  <bean id="userDao" class="springbook.user.dao.UserDao">
    <property name="connectionMaker" ref="localDBConnectionMaker">
  </bean>
</beans>
```

- 사용환경에 따라 userDao 빈의 ref 값을 다르게 지정한 XML 파일을 두고 사용하면 된다.

```
* DTD와 스키마

XML 문서는 미리 정해진 구조를 따라서 작성됐는지 검사할 수 있다. XML 문서의 구조를 정의하는 방법에는 DTD와 스키마(schema)가 있다. 스프링의 XML 설정파일은 이 두가지 방식을 모두 지원한다.

DTD를 사용할 경우에는 beans 엘리먼트 앞에 다음과 같은 DTD 선언을 넣어준다.

<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">

스프링은 DI를 위한 기본 태그인 <beans>, <bean> 외에도 특별한 목적을 위해 별도의 태그를 사용할 수 있는 방법을 제공한다. 이 태그들은 각각 별개의 스키마 파일에 정의되어 있고 독립적인 네임스페이스를 사용해야만 한다. 따라서 이런 태그를 사용하려면 DTD 대신 네임스페이스가 지원되는 스키마를 사용해야 한다. <beans> 태그를 기본 네임스페이스로 하는 스키마 선언은 다음과 같다.

<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi::schemaLocation="http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

특별한 이유가 없다면 DTD보다는 스키마를 사용하는 편이 바람직하다.
```
