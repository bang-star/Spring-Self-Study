# 1.8.2 XML을 이용하는 애플리케이션 컨텍스트(p.134 ~ 135)

- 애플리케이션 컨텍스트가 DaoFactory 대신 XML 설정정보를 활용하도록 만들어보자.

- XML에서 빈의 의존관계 정보를 이용하는 IoC/DI 작업에는 GeneralXmlApplicationContext를 사용한다.

- GeneralXmlApplicationContext의 생성자 파라미터로 XML 파일의 클래스패스를 지정해주면 된다.

- XML 설정파일은 클래스 패스 최상단에 두면 편한다.

- 애플리케이션 컨텍스트가 사용하는 XML 설정파일의 이름은 관례를 따라 applicationContext.xml이라고 만든다. 

리스트 1-40. XML 설정정보를 담은 applicationContext.xml
```XML
<?xml version="1.0" encoding="UTF-8?">
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 <bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
  <bean id="userDao" class="springbook.user.dao.UserDao">
    <property name="connectionMaker" ref="connectionMaker" />
  </bean>  
</beans>
```

- UserDaoTest의 애플리케이션 컨텍스트 생성 부분을 수정한다. 

  - DaoFactory를 설정정보로 사용했을 때  AnnotaionConfigApplicationContext → GenericXmlApplicationContext 를 이용해 애플리케이션 컨텍스트를 생성

    ```Java
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    ```

- GenericXmlApplicationContext 외에도 ClassPathXmlApplicationContext를 이용해 XML로부터 설정정보를 가져오는 애플리케이션 컨텍스트를 만들 수 있다. 

- GenericXmlApplicationContext는 클래스패스뿐 아니라 다양한 소스로부터 설정파일을 읽어올 수 있다.

- ClassPathXmlApplicationContext는 XML 파일을 클래스패스에서 가져올 때 사용할 수 있는 편리한 기능이 추가된 것이다.

- ClassPathXmlApplicationContext의 기능 중에는 클래스패스의 경로정보를 클래스에서 가져오게 하는 것이 있다.

  예를 들면 springbook.user.dao 패키지 안에 daoContext.xml이라는 설정파일이 있을 때 GenericXmlApplicationContext가 이 XML 설정파일을 사용하게 하려면 클래스패스 루트로부터 파일의 위치를 지정해야 하므로 다음과 같이 작성해야 한다. 패키지가 길어지면 클래스패스를 모두 적기가 귀찮을 수도 있다.
  
  ```Java
  new GenericApplicationContext("springbook/user/dao/daoContext.xml");
  ```
  
- ClassPathXmlApplicationContext는 XML파일과 같은 클래스패스에 있는 클래스 오브젝트를 넘겨서 클래스패스에 대한 힌트를 제공할 수 있다.

- UserDao는 springbook.user.dao 패키지에 있으므로 daoContext.xml과 같은 클래스패스 위에 있다.

- UserDao를 함께 넣어주면 XML 파일의 위치를 UserDao의 위치로부터 상대적으로 지정할 수 있다.

  ```Java
  new ClassPathXmlApplicationContexxt("daoContext.xml", UserDao.class);
  ```

  - 이 방법으로 클래스패스를 지정해야 할 경우가 아니라면 GenericXmlApplicationContext를 사용하는 편이 무난하다.
