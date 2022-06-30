# 1.4 제어의 역전(IoC) (p.88 ~ 92)

## 1.4.1 오브젝트 팩토리

- UserDaoTest는 UserDao의 기능이 잘 작동하는지를 테스트하려고 만든 것인데, 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 기능을 떠맡고 있다.

- 분리될 기능은 UserDao와 ConnectionMaker 구현 클래스의 오브젝트를 만드는 것과 만들어진 두 개의 오브젝트가 연결돼서 사용될 수 있도록 관계를 맺어주는 것이다.

### 팩토리

-  객체의 생성 방법을 결정하고 만들어진 오브젝트를 돌려주는 일을 하는 오브젝트를 흔히 팩토리라 한다.
- 디자인 패턴에서 특별한 문제를 해결하기 위해 사용되는 추상 팩토리 패턴이나 팩토리 메소드 패턴과는 다르므로 혼동하면 안된다.
- 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 깔끔하게 분리하려는 목적으로 사용하는 것이다.

```
 * 상황 요약 

 1. 팩토리 역할을 맡을 클래스를 DaoFactor
 2. UserDaoTest에 담겨있던 UserDao, ConnectionMaker 관련 생성 작업을 DaoFactory로 이동
 3. UserDaoTest에 DaoFactory에 요청해서 미리 만들어진 UserDao 오브젝트 사용
 ```

리스트 1-14. UserDao의 생성 책임을 맡은 팩토리 클래스
```Java
public class DaoFactory {

    public UserDao userDao(){    // 팩토리의 메소드는 UserDao 타입의 오브젝트를 어떻게 만들고, 어떻게 준비시킬지를 결정한다.
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

리스트 1-15. 팩토리를 사용하도록 수정한 UserDaoTest
```Java
public class UserDaoTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        UserDao dao = new DaoFactory().userDao();
    }
}
```

<br />
<hr />
<br />

### 설계도로서의 팩토리

- 분리된 오브젝트들의 역할과 관계 
    
    - UserDao와 ConnectionMaker
        
        각각 애플리케이션의 핵심적인 데이터 로직과 기술 로직을 담당

    - DaoFactory
    
        애플리케이션의 오브젝트를 구성하고 관계를 정의하는 책임을 맡음.

<br />

> UserDao와 ConnectionMaker가 실질적인 로직을 담당하는 컴포넌트라면, DaoFactory는 애플리케이션을 구성하는 컴포넌트의 구조와 관계를 정의한 설계도와 같은 역할을 한다.

- 설계도는 간단히 어떤 오브젝트가 어떤 오브젝트를 사용하는지를 정의해놓은 코드라고 생각하면 된다.

<br />

<img src="https://user-images.githubusercontent.com/40616436/75882202-08775400-5e64-11ea-9e30-202c05c79682.png">

<br />

> UserDao는 변경이 필요 없으므로 안전하게 소스코드를 보존할 수 있고, 동시에 DB 연결 방식은 자유로운 확장이 가능하다.

<br />
<hr />
<br />

## 1.4.2 오브젝트 팩토리의 활용

- DaoFactory에 UserDao가 아닌 다른 DAO의 생성 기능을 넣으면 어떻게 될까?

    - UserDao를 생성하는 userDao() 메소드를 복사해서 accountDao, messageDao()메소드로 만든다면 ConnectionMaker 구현 클래스의 오브젝트를 생성하는 코드가 메소드마다 반복되는 문제가 발생한다.

    - 오브젝트 생성 코드가 중복되는 것은 DAO가 많아지면 구현 클래스를 바꿀 때마다 모든 메소드를 일일이 수정해야하기 때문에 좋지 않은 현상이다.

```Java
public class DaoFactory{
    
    // ConnectionMaker 구현 클래스를 선정하고 생성하는 코드의 중복

    public UserDao userDao(){
        return new UserDao(new DConnectionMaker());
    }

    public AccountDao accountDao(){
        return new AccountDao(new DConnectionDao());
    }

    public MessageDao messageDao(){
        return new MessageDao(new MessageDao());
    }
}
```

- 중복 문제를 해결하기 위해 분리해내야 한다.

    1. ConnectionMaker의 구현 클래스를 결정

    2. 구현 클래스의 오브젝트를 만드는 코드를 별도의 메소드로 분리
    
    3. DAO를 생각하는 각 메소드에서 새로 만든 ConnectionMaker 생성용 메소드를 이용하도록 수정

<br />

> getConnection 메소드를 따로 만들어 DB 연결 기능을 분리해낸 것과 동일한 리팩토링 방법이다.

<br />

```Java
public class DaoFactory{
    
    public UserDao userDao(){
        return new UserDao(connectionMaker());
    }

    public AccountDao accountDao(){
        return new AccountDao(connectionMaker());
    }

    public MessageDao messageDao(){
        return new MessageDao(connectionMaker());
    }

    // 분리해서 중복을 제거한 ConnectionMaker 타입 오브젝트 생성
    public ConnectionMaker connectionMaker(){
        return new DConnectionMaker();  
    }
}
```

<br />
<hr />
<br />

## 1.4.3 제어권의 이전을 통한 제어관계 역전

> 제어의 역전은 간단히 프로그램의 제어 흐름 구조가 뒤바뀐 것을 의미한다.

1. 일반적인 프로그램의 흐름
    
    1. main()메소드와 같이 프로그램이 시작되는 지점에서 사용할 오브젝트 결정

    2. 결정한 오브젝트를 생성하고, 만들어진 오브젝트에 있는 메소드 호출

    3. 해당 오브젝트 메소드 안에서 사용할 것을 결정하고 호출하는 식의 작업 반복

> 일반적인 프로그램의 흐름 구조에서 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.