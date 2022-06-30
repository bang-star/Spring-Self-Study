# 1.5 스프링의 IoC

- 스프링은 애플리케이션 개발의 다양한 영역과 기술에 관여하고, 매우 많은 기능을 제공한다.

- 스프링의 핵심은 빈 팩토리 또는 애플리케이션 컨텍스트이다.

<br />
<hr />
<br />

## 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC

### 애플리케이션 컨텍스트와 설정정보

  - Bean
    
    - 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
    
    - 자바빈 또는 앤터프라이즈 자바빈(EJB)에서 말하는 빈과 비슷한 오브젝트 단위의 애플리케이션 컴포넌트

    - 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리키는 말이다.


- 스프링에서 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 빈 팩토리라 한다.

- 스프링은 보통 빈 팩토리보다 더 확장한 애플리케이션 컨텍스트를 주로 사용

    - 애플리케이션 컨텍스트는 IoC 방식을 따라 만들어진 일종의 빈 팩토리다.

<br />

 - 빈팩토리
    
    빈을 생성하고 관계를 설정하는 IoC의 기본 기능에 초점을 맞춘 것

 - 애플리케이션 컨텍스트
    
    애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진이라는 의미가 더 부각

    별도의 정보를 참고해서 빈(오브젝트)의 생성, 관계설정 등의 제어 작업을 총괄

    ```
    예를 들어 기존 DaoFactory 코드에 설정정보는 어떤 클래스의 오브젝트를 생성하고 어디에서 사용하도록 연결해줄 것인가 등에 관한 정보가 평범한 자바코드로 만들어져 있다. 
    
    애플리케이션 컨텍스트는 직접 이런 정보를 담고 있진 않다. 대신 별도로 설정정보를 담고 있는 무엇인가를 가져와 이를 활용하는 범용적인 IoC 엔진 같은 것이라고 볼수 있다.
    ```

- 빈 팩토리 또는 애플리케이션 컨텍스트가 사용하는 설정정보를 만드는 방법은 여러가지가 있는데, 앞에서 만든 오브젝트 팩토리인 DaoFactory도 조금만 손을 보면 설정정보로 사용할 수 있다.

- DaoFactory 자체가 설정정보까지 담고 있는 IoC 엔진이었는데 자바 코드로 만든 애플리케이션 컨텍스트의 설정정보로 활용될 것이다.

- 건물이 설계도면을 따라서 만들어지듯이, 애플리케이션도 애플리케이션 컨텍스트와 그 설정정보를 따라서 만들어지고 구성된다고 생각할 수 있다.

<br />
<hr />
<br />

### DaoFactory를 사용하는 애플리케이션 컨텍스트

- DaoFactory를 스프링의 빈 팩토리가 사용할 수 있는 본격적인 설정정보로 만들어볼 예정이다.

 1. 스프링이 빈 팩토리를 위한 설정을 담당하는 클래스라고 인식할 수 있도록 @Configuration 추가

 2. 오브젝트를 만들어주는 메소드에 @Bean 추가

 ```
 * 참고

 1. @Configuration
 - 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시

 2. @Bean
 - 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
 
 Ex) userDao메소드는 UserDao타입 오브젝트를 생성하고 초기화해서 돌려주는 메소드이므로 @Bean을 붙여준다. 

 Ex) ConnectionMaker 타입의 오브젝트를 생성해주는 connectionMaker() 메소드이므로 @Bean을 붙여준다. 
 ```

 - 두 애노테이션만으로 스프링 프레임워크의 빈 팩토리 또는 애플리케이션 컨텍스트가 IoC 방식의 기능을 제공할 때 사용할 완벽한 설정정보가 된 것이다.
 

 리스트 1-18. 스프링 빈 팩토리가 사용할 설정정보를 담은 DaoFactory 클래스
 ```Java
 @Configuration                    // 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory {

    @Bean                         // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
    public UserDao userDao(){    // 팩토리의 메소드는 UserDao 타입의 오브젝트를 어떻게 만들고, 어떻게 준비시킬지를 결정한다.
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker(){
        return new DConnectionMaker();
    }
}
 ```

<br />

 > DaoFacotry를 설정정보로 사용하는 애플리케이션 컨텍스를 만들어보자. 
 
 - 애플리케이션 컨텍스트는 ApplicationContext 타입의 오브젝트다.

 - ApplicationContext를 구현한 클래스는 여러 가지가 있는데 DaoFactory처럼 @Configuration이 붙은 자바 코드를 설정정보로 사용하려면 AnnotataionConfigApplicationContext를 이용하면 된다.

 - 애플리케이션 컨텍스트를 만들 때 생성자 파라미터로 DaoFactory 클래스를 넣어준다. 
 
 - 준비된 ApplicationContext의 getBean()이라는 메소드를 이용해 UserDao의 오브젝트를 가져올 수 있다.

<br />

리스트 1-19. 애플리케이션 컨텍스트를 적용한 UserDaoTest
```Java
public class UserDaoTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException{
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);
    }
}
```

- getBean() 메소드
    
    - ApplicationContext가 관리하는 오브젝트를 요청하는 메소드이다.
    
    - 파리미터 "userDao"는 ApplicationContext에 등록된 빈의 이름이다.

    - DaoFactory에서 @Bean이라는 애토테이션을 userDao라는 이름의 메소드에 붙였는데, 이 메소드 이름이 바로 빈의 이름이 된다.

    - userDao라는 이름의 빈을 가져온다는 것은 DaoFactory의 userDao()를 호출해서 그 결과를 가져온다고 생각하면 된다.

    - 기본적으로 Object 타입으로 리턴하게 되어 있어서 매번 리턴되는 오브젝트에 다시 캐스팅을 해줘야 하는 부담이 있다.
    
    - 자바 5이상의 제네릭 메소드 방식을 사용해 getBean()의 두 번째 파라미터에 리턴 타입을 주면, 지저분한 캐스팅 코드를 사용하지 않아도 된다.

```
* 추가할 라이브러리

com.springsource.net.sf.cglib-2.2.0.jar
com.springsource.org.apache.commons.logging-1.1.1.jar
org.springframework.asm-3.0.7.RELEASE.jar
org.springframework.beans-3.0.7.RELEASE.jar
org.springframework.context-3.0.7.RELEASE.jar
org.springframework.core-3.0.7.RELEASE.jar
org.springframework.expression-3.0.7.RELEASE.jar
```

- 스프링은 지금 우리가 구현했던 DaoFactory를 통해서 얻을 수 없는 방대한 기능과 활용 방법을 제공해준다.

