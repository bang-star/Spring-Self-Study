# 2.2 UserDaoTest 개선(p. 154 ~ 161)

## 2.2.1 테스트 검증의 자동화

  - 첫 번째 문제점인 테스트 결과의 검증

     - 테스트를 통해 확인하고 싶은 사항은 add()에 전달한 user 오브젝트에 담긴 사용자 정보와 get()을 통해 다시 DB에서 가져온 User 오브젝트의 정보가 서로 정확히 일치하는가이다. 정확히 일치한다면 모든 정보가 빠짐없이 DB에 등록됐고, 이를 다시 DB에서 정확히 가져왔다는 사실을 알 수 있다.

    1. 테스트 중에 에러가 발생하는 것은 쉽게 확인 가능하지만, 테스트가 실패하는 것은 별도의 확인 작업과 결과가 있어야 한다.

    2. 모든 확인 작업을 통과하면 "테스트 성공"이라 하고 기존 "조회 성공"메시지는 get 메소드가 에러 없이 끝났다는 것만 알 수 있고 판별 작업과는 무관하다.

리스트 2-2. 수정 전 테스트 코드
```Java
System.out.println(user2.getName());
System.out.println(user2.getPassword());
System.out.println(user2.getId()+"조회 성공");
```

리스트 2-3. 수정 후 테스트 코드
```Java
if(!user.getName().equals(user2.getName())){
    System.out.println("테스트 실패 (name)");
}
else if(!user.getPassword().equals(user2.getPassword())){
    System.out.println("테스트 실패 (password)");
}else{
    System.out.println("조회 테스트 성공");
}
```

   - 변경 사항

       - 처음 add()에 추가한 User 오브젝트와 get()을 통해 가져오는 User 오브젝트의 값을 비교해서 일치하는지 확인

       - 비교 실패시 각 해당 값에 대한 에러 출력

   - 테스트는 기능이 정상적으로 동작하는지를 언제든지 손쉽게 확인할 수 있게 해준다.


   - 포괄적인 테스트(comprehensive test)

     만들어진 코드의 기능을 모두 점검할 수 있는 테스트

     개발한 애플리케이션은 어떤 과감한 수정을 하고 나서도 테스트를 통해 안전성을 확인할 수 있다. 또는 테스트를 통해 변경에 영향을 받는 부분이 정확이 확인된다면 빠르게 조취를 취할 수 있다.

<br />
<hr />
<br />

## 2.2.2 테스트의 효율적인 수행과 결과 관리

  - main() 메소드로 만든 테스트는 테스트로서 필요한 기능은 모두 갖추고 있지만, 편리하게 테스트를 수행하고 편리하게 결과를 확인하려면 main 메소드로는 한계가 있다.

  - 일정한 패턴을 가진 테스트를 만들 수 있고, 많은 테스트를 간단히 실행시킬 수 있으며, 테스트 결과를 종합해서 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있는 기능을 갖춘 테스트 지원 도구와 그에 맞는 테스트 작성 방법이 필요하다.

<br />

### JUnit 테스트로 전환

  - JUnit은 프레임워크다. 

     - 프레임워크는 개발자가 만든 클래스에 대한 제어 권한을 넘겨 받아서 주도적으로 애플리케이션의 흐름을 제어한다.

     - 개발자가 만든 클래스의 오브젝트를 생성하고 실행하는 일은 프레임워크에 의해 진행된다.

  - 프레임워크에서 동작하는 코드는 main 메소드도 필요 없고 오브젝트를 만들어서 실행시키는 코드를 만들 필요가 없다.

<br />

### 테스트 메소드 전환

  - main 메소드 테스트는 프레임워크에 적용하기에 적합하지 않다. 테스트가 main 메소드로 만들어졌다는 것은 제어권을 직접 갖는다는 의미이기 때문이다.

  - JUnit 프레임워크 요구 조건

    1. 메소드가 public으로 선언되어야 한다.

    2. 메소드에 @Test 어노테이션을 붙여주어야 한다.

<br />

리스트 2-4. JUnit 프레임워크에서 동작할 수 있는 테스트 메소드로 전환
```Java
import org.junit.Test;

public class UserDaoTest {
    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        ...
        UserDao dao = context.getBean("userDao", UserDao.class);
        ...
    }
}
```

   - JUnit은 전통적으로 public 메소드만을 테스트 메소드로 허용하고 있다.

### 검증 코드 전환

  - 테스트의 결과를 검증하는 if/else 문장을 JUnit이 제공하는 방법을 이용해 전환하는 과정
    
```Java
    if (!user.getName().equals(user2.getName())){ ... }
```

  - if 문장의 기능을 JUnit이 제공해주는 assertThat이라는 스태틱 메소드를 이용해 변경할 수 있다.

```Java
    assertThat(user2.getName(), is(user.getName()));
```

  - assertThat() 메소드는 첫 번째 파라미터의 값을 뒤에 나오는 매처(matcher)라고 불리는 조건으로 비교해서 일치하면 다음으로 넘어가고, 아니면 테스트가 실패하도록 만들어준다.

  - is()는 매처의 일종으로 equals()로 비교해주는 기능이다.

```Java
   assertThat(user2.getPassword(), is(user.getPassword()));
```

  - JUnit은 예외가 발생하거나 assertThat()에서 실패하지 않고 테스트 메소드의 실행이 완료되면 테스트가 성공했다고 인식한다.

  - JUnit이 테스트를 실행하고 나면 테스트 결과를 다양한 방법으로 알려주기 때문이다.

  리스트 2-5. JUnit을 적용한 UserDaoTest
  ```Java
  import static org.hamcrest.CoreMatchers.is;
  import static org.junit.Assert.assertThat;
  ...
  public class UserDaoTest {
    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);
        User user = new User();
        user.setId("gyumee");
        user.setName("박성철");
        user.setPassword("springno1");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }
  }
  ```

    ```
    추가할 라이브러리

    com.springsource.org.junit-4.7.0.jar
    ```

<br />

### JUnit 테스트 실행

  - 스프링 컨테이너와 마찬가지로 JUnit 프레임워크도 자바 코드로 만들어진 프로그램이므로 어디선가 한 번은 JUnit 프레임워크를 시작시켜 줘야 한다.

```Java
import org.junit.runner.JUnitCore;

public static void main(String[] args){
    JUnitCore.main("springbook.user.dao.UserDaoTest");
}
```

  - 이 클래스를 실행하면 다음과 같은 메시지가 출력된다.

    JUnit version 4.7   <br />
    Time : 0.578        <br />
    OK (1 test)         <br />

  - 테스트가 실행하는데 걸린 시간과 테스트 결과, 그리고 몇 개의 테스트 메소드가 실행됐는지를 알려준다.

  - 만약 코드에 이상이 있어서 assertThat()의 검증에서 실패하면 다음과 같은 메시지가 나올 것이다.

    Time : 1.094                <br />
    There was 1 failure :       <br />
    1 ) addAndGet(springbook.dao.UserDaoTest)                <br />
    java.lang.AssertionError:   <br />
    Excepted : is "박성철"      <br />
    got : null                  <br />
    ...
    at springbook.dao.UserDaoTest.main(UserDaoTest.java:36)
    FAILURES!!!                 <br />
    Tests run: 1, Failures: 1

  - 테스트가 성공 실패에 대한 메시지와 총 수행한 테스트 중에서 몇개의 테스트가 실패했는지 보여준다.

  - 출력된 메시지를 통해 실패한 원인이 무엇이고 테스트 코드에서 정증에 실패한 위치는 어디인지 확인할 수 있다.

  - JUnit은 assertThat()을 이용해 검증을 했을 떄 기대한 결과가 아니면 AssertionError를 던진다.

  - assertThat()의 조건을 만족하지 못하면 테스트는 더 이상 진행되지 않고 JUnit은 테스트가 실패했음을 알게 된다.

  - 테스트 수행 중에 일반 예외가 발생한 경우에도 마찬가지로 테스트 수행은 중단되고 테스트는 실패한다.