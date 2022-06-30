# 1.6 싱글톤 레지스트리와 오브젝트 스코프

- 오브젝트의 동일성과 동등성
  
  - 완전히 같은 동일한(identical) 오브젝트는 동일성(identity) 비교로 == 연산자를 이용해 비교한다.
  
  - 동일한 정보를 담고 있는(equivalent) 오브젝트는 동등성(equality) equals() 메소드를 이용해 비교한다.

  - 두 개의 오브젝트가 동일하다면 하나의 오브젝트만 존재하는 것이고 두 개의 오브젝트 레퍼런스 변수를 갖고 있을 뿐이다.

  - 두 개의 오브젝트가 동일하지는 않지만 동등한 경우에는 두 개의 각기 다른 오브젝트가 메모리상에 존재하는 것인데, 오브젝트의 동등성 기준에 따라 두 오브젝트의 정보가 동등하다고 판단하는 것일 뿐이다.

    
- 예시
```
 DaoFactory의 userDao()를 여러 번 호출했을 때 동일한 오브젝트인지에 대한 여부이다. 
 
 코드를 보면 매번 userDao 메소드를 호출할 때마다 new 연산자에 의해 새로운 오브젝트가 만들어지게 되어 있기 때문에 당연히 매번 다른 오브젝트가 만들어져서 돌아올 것이라고 예상할 수 있다. 매번 다른 각기 다른 주소 값을 가진 동일하지 않은 오브젝트다.

 스프링의 애플리케이션 컨텍스트에 Dao Factory를 설정정보로 등록하고 getBean() 메소드를 이용해 userDao라는 이름으로 등록된 오브젝트를 가져오면 애플리케이션 컨텍스트가 DaoFactory의 userDao() 메소드를 호출해서 UserDao 타입 오브젝트를 만드는 것은 동일하지만 오브젝트가 동일하다는 사실을 알 수 있다. 이를 통해 스프링은 여러 번에 걸쳐 빈을 요청하더라도 매번 동일한 오브젝트를 돌려준다는 것이다.
```

리스트 1-20 직접 생성한 DaoFactory 오브젝트 출력 코드
```Java
DaoFactory factory = new DaoFactory();
UserDao dao1 = factory.UserDao();
UserDao dao2 = factory.UserDao();

System.out.println(dao1);   // @118f375
System.out.println(dao2);   // @117a8bd
```

리스트 1-21 스프링 컨텍스트로부터 가져온 오브젝트 출력 코드
```Java
ApplicationContext context = ApplicationConfigApplicationContext(DaoFactory.class);
UserDao dao3 = context.getBean("userDao", Userdao.class);
UserDao dao4 = context.getBean("userDao", Userdao.class);

System.out.println(dao3);   // @ee22f7
System.out.println(dao4);   // @ee22f7
```

<br />
<hr />
<br />

## 1.6.1 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트는 우리가 만들었던 오브젝트 팩토리와 비슷한 방식으로 동작하는 IoC 컨테이너이지만, 싱글톤을 저장하고 관리하는 싱글톤 레지스트리이기도 하다.

- 스프링은 기본적으로 별다른 설정을 하지 않으면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다. 

> (유의) 디자인 패턴에서 나오는 싱글톤 패턴과 비슷한 개념이지만 구현방법은 확연히 다르다

<br />

### 서버 애플리케이션과 싱글톤
 
 - 스프링은 싱글톤으로 빈을 만드는 것일까?
   
   - 스프링이 주로 적용되는 대상이 자바 엔터프라이즈 기술을 사용하는 서버 환경이기 때문이다.

   - 스프링은 엔터프라이즈 시스템을 위해 고완된 기술이기 때문에 서버 환경에서 사용될 때 그 가치가 있다.

```
* 상황적 예시
매번 클라이언트에서 요청이 올 때마다 각 로직을 담당하는 오브젝트를 새로 만들어 사용하는 상황을 가정해보자.

요청 한 번에 5개의 오브젝트가 새로 만들어지고 초당 500개의 요청이 들어오면, 초당 2500개의 새로운 오브젝트가 생성된다. 

1분이면 15만개, 한 시간이면 9백만개가 생성된다.

자바의 오브젝트 생성과 가비지 컬렉션의 성능이 좋아졌다고 한들 이렇게 부하가 걸리면 서버가 감당하기 힘들다. 
```

- 서블릿 
  
  - 자바 엔터프라이즈 기술의 가장 기본이 되는 서비스 오브젝트라 할 수 있다. 

  - 스펙에서 강제하지 않지만, 서블릿은 대부분 멀티스레드 환경에서 싱글톤으로 동작한다. 

  - 서블릿 클래스당 하나의 오브젝트만 만들어두고, 사용자의 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용한다.

- 애플리케이션 안에 제한된 수, 대개 한 개의 오브젝트만 만들어 사용하는 것이 싱글톤 패턴의 원리다. 따라서 서버환경에서는 서비스 싱글톤의 사용이 권장된다.

> 디자인 패턴에 소개된 싱글톤 패턴은 사용하기가 까다롭고 여러 가지 문제점이 있어 안티패턴(anti pattern)이라고 부른 사람도 있다.

```
싱글톤 패턴(Singleton Pattern)

디자인 패턴 중에서 가장 자주 활용되는 패턴이기도 하지만 가장 많은 비판을 받는 패턴이기도 하다.

싱글톤 패턴은 어떤 클래스를 애플리케이션 내에서 제한된 인스턴스 개수, 이름처럼 주로 하나만 존재하도록 강제하는 패턴이다.

하나만 만들어지는 클래스의 오브젝트는 애플리케이션 내에서 전역적으로 접근이 가능하다. 

정리하면 단일 오브젝트만 존재하고, 이를 애플리케이션의 여러 곳에서 공유하는 경우에 주로 사용한다.
```
