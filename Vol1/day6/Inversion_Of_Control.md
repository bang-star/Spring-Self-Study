# 1.4 제어의 역전(IoC) (p.88 ~ 91)

## 1.4.1 오브젝트 팩토리

``` 
UserDaoTest는 UserDao의 기능이 잘 작동하는지를 테스트하려고 만든 것인데, 또 다른 책임까지 떠맡고 있다.
```

### 팩토리
> 분리시킬 기능을 담당할 클래스를 생성하여, 기능을 분리할 것이다.

- 클래스의 역할은 객체의 생성 방법을 결정하고 만들어진 오브젝트를 돌려주는 것인데, 이런 일을 하는 오브젝트를 흔히 팩토리라 한다.
- 디자인 패턴에서 특별한 문제를 해결하기 위해 사용되는 추상 팩토리 패턴이나 팩토리 메소드 패턴과는 다르므로 혼동하면 안된다.
- 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 깔끔하게 분리하려는 목적으로 사용하는 것이다.

<br />

```
 팩토리 역할을 맡을 클래스를 DaoFactory라 하고, UserDaoTest에 담겨있던 UserDao, ConnectionMaker 관련 생성 작업을 DaoFactory로 옮기고, UserDaoTest에 DaoFactory에 요청해서 미리 만들어진 UserDao 오브젝트를 가져와 사용하게 만든다.
 ```

```Java
public class DaoFactory {

    public UserDao userDao(){    // 팩토리의 메소드는 UserDao 타입의 오브젝트를 어떻게 만들고, 어떻게 준비시킬지를 결정한다.
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

- DaoFactory의 UserDao 메소드를 호출하면 DConnectionMaker를 사용해 DB 커넥션을 가져오도록 이미 설정된 UserDao 오브젝트를 돌려준다. UserDaoTest는 UserDao가 어떻게 만들어지는지 어떻게 초기화되어 있는지에 신경 쓰지 않고 팩토리로부터 UserDao 오브젝트를 받아서, 자신의 관심사인 테스틀 위해 활용하기만 하면 된다.

```Java
UserDao dao = new DaoFactory().userDao();
```

<br />
<hr />
<br />

### 설계도로서의 팩토리

- 분리된 오브젝트들의 역할과 관계 
    
    - UserDao와 ConnectionMaker는 각각 애플리케이션의 핵심적인 데이터 로직과 기술 로직을 담당
    - DaoFactory는 애플리케이션의 오브젝트를 구성하고 관계를 정의하는 책임을 맡고 있다.

> UserDao와 ConnectionMaker가 실질적인 로직을 담당하는 컴포넌트라면, DaoFactory는 애플리케이션을 구성하는 컴포넌트의 구조와 관계를 정의한 설계도와 같은 역할을 한다.

- 설계도는 간단히 어떤 오브젝트가 어떤 오브젝트를 사용하는지를 정의해놓은 코드라고 생각하면 된다.

<br />

<img src="https://user-images.githubusercontent.com/40616436/75882202-08775400-5e64-11ea-9e30-202c05c79682.png">

- N사와 D사에 UserDao를 공급할 때 UserDao, ConnectionMaker와 함께 DaoFactory도 제공한다.
- 새로운 ConnectionMaker 구현 클래스로 변경이 필요하면 DaoFactory를 수정해서 변경된 클래스를 생성해 설정해주도록 코드를 수정해주면 된다.

> UserDao는 변경이 필요 없으므로 안전하게 소스코드를 보존할 수 있고, 동시에 DB 연결 방식은 자유로운 확장이 가능하다.

<br />
<hr />
<br />

## 1.4.2 오브젝트 팩토리의 활용

- DaoFactory에 UserDao가 아닌 다른 DAO의 생성 기능을 넣으면 어떻게 될까?

    - UserDao를 생성하는 userDao() 메소드를 복사해서 accountDao, messageDao()메소드로 만든다면 새로운 문제가 발생한다.

    > ConnectionMaker 구현 클래스의 오브젝트를 생성하는 코드가 메소드마다 반복되는 것이다.

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
    2. 오브젝트를 만드는 코드를 별도의 메소드로 분리
    3. DAO를 생각하는 각 메소드에서 새로 만든 ConnectionMaker 생성용 메소드를 이용하도록 수정
    4. DAO 팩토리 메소드가 많아져도 문제가 없다.

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

- 제어의 역전은 간단히 프로그램의 제어 흐름 구조가 뒤바뀐 것을 의미한다.

    1. 일반적인 프로그램의 흐름
        
        1. main()메소드와 같이 프로그램이 시작되는 지점에서 사용할 오브젝트 결정
        2. 결정한 오브젝트를 생성하고, 만들어진 오브젝트에 있는 메소드 호출
        3. 해당 오브젝트 메소드 안에서 사용할 것을 결정하고 호출하는 식의 작업 반복

    > 일반적인 프로그램의 흐름 구조에서 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.
    