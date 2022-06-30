# 1.3 DAO의 확장(p.71 ~ 83)

- 관심사에 따라 분리한 오브젝트는 각기 독특한 변화의 특징이 있다. 
- 변화의 성격이 다르다는 것은 변화의 이유와 시기, 주기 등이 다르다는 의미이다..

```
(예시) 
UseDao를 기준으로
1. JDBC API를 사용할 것인지
2. DB 전용 API를 사용할 것인지
3. 어떤 테이블 이름과 필드 이름을 사용해 어떤 SQL을 만들 것인지
4. 어떤 오브젝트를 통해 DB에 저장할 정보를 전달받고, DB에서 꺼내온 정보를 저장해서 넘겨줄 것인지
위와 같은 관심을 가진 코드를 모아둔 것이다.
=> 관심사가 바뀌면 변경이 발생한다.

if DB 연결 방법이 그대로라면,
DB 연결 확장 기능을 담은 NUserDao나 DUserDao는 변경되지 않는다.

if 사용자 정보를 저장하고 가져오는 방법에 대한 관심은 바뀌지 않지만 DB연결 방식이나 DB 커넥션을 가져오는 방법이 바뀌면, 
UserDao 코드는 NUserDao나 DUserDao의 코드만 변경된다.
```

- 추상 클래스를 만들고 이를 상속한 서브클래스에서 변화가 필요한 부분을 바꿔서 쓸수 있게 만든 이유는 바로 이렇게 변화의 성격이 다른 것을 분리해서, 서로 영향을 주지 않은 채로 각각 필요한 시점에 독립적으로 변경할 수 있게 하기 위해서이다.

<br />
<hr />
<br />

## 1.3.1 클래스 분리

> 두 개의 관심사를 독립시키면서 동시에 손쉽게 확장할 수 있는 방법을 알 수 있다.

    * 진행 상황
    1. 독립된 메소드를 생성하여 분리
    2. 상하위 클래스 분리
    3. 상속관계가 아닌 완전히 독립적인 클래스(Now)

- DB 커넥션과 관련된 부분을 서브클래스가 아니라, 아예 별도의 클래스에 담는다.

<br />

<img src="https://user-images.githubusercontent.com/40616436/75679116-5e1bf700-5cd2-11ea-8700-f176678ca17b.png">

<br />

1. SimpleConnectionMaker라는 새로운 클래스를 만들고 DB 생성 기능을 넣는다.
2. UserDao는 new 키워드를 사용해 SimpleConnectionMake 클래스의 오브젝트 생성하여 add, get 메소드에서 사용하면 된다

리스트 1-6. 독립된 SimpleConnectionMaker를 사용하게 만든 UserDao
```Java
package com.example.toby_spring.dao;

import com.example.toby_spring.domain.User;

import java.sql.*;

public class UserDao {

    private final SimpleConnectionMaker simpleConnectionMaker;
    
    public UserDao(){
        this.simpleConnectionMaker = new SimpleConnectionMaker();
    }
    
    public void add(User user) throws ClassNotFoundException, SQLException{
        Connection c = simpleConnectionMaker.makeNewConnection();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.makeNewConnection();    
    }
}


public class SimpleConnectionMaker {
    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
        return c;
    }
}
```

<br />

- N사와 D사에 UserDao 클래스만 공급하고 상속을 통해 DB 커넥션 기능을 확장해서 사용하는 것이 불가능해졌다. 이유는 UserDao의 코드가 SimpleConnectionMaker라는 특정 클래스에 종속되어 있기 때문에 상속을 사용했을 때처럼 수정 없이 DB 커넥션 생성 기능을 변경할 방법이 없다.

### 클래스 분리를 위해 해결해야할 문제점

1. SimpleConnectionMake의 메소드

    - makeNewConnection()을 사용해 DB 커넥션을 가져오게 했는데, 만약 D사에서 만든 DB 커넥션 제공 클래스는 openConnectionMake()이라는 메소드 이름을 사용했다면 UserDao내에 있는 add(), get() 메소드의 커넥션을 가져오는 코드를 일일이 변경해야 한다. 

2. DB 커넥션을 제공하는 클래스가 어떤 것인지를 UserDao가 구체적으로 알고 있어야 한다.

    - UserDao에 SimpleConnectionMaker라는 클래스 타입의 인스턴스 변수까지 정의해놓고 있으니, N사에서 다른 클래스를 구현하면 어쩔수 없이 UserDao를 수정해야 한다.
    
<br />

- 근본적인 원인은 UserDao가 바뀔 수 있는 정보, 즉 DB 커넥션을 가져오는 클래스에 대해 너무 많이 알고 있기 때문이다. 어떤 클래스가 쓰일지, 그 클래스에서 커넥션을 가져오는 메소드는 이름이 뭔지까지 일일이 알고 있어야한다.

> UserDao는 DB 커넥션을 가져오는 구체적인 방법에 종속되어 버린다.

> UserDao는 SimpleConnectionMaker라는 특정 클래스와 그 코드에 종속적이기 때문에 DB 커넥션을 가져오는 방법을 자유롭게 확장하기가 힘들다.

<br />
<hr />
<br />

## 1.3.2 인터페이스의 도입

> 가장 좋은 해결책은 두 개의 클래스가 서로 긴밀하게 연결되어 있지 않도록 중간에 추상적인 느슨한 연결고리를 만들어주는 것이다.

```
추상화
- 어떤 것들의 공통적인 성격을 뽑아내어 따로 분리해내는 작업

> 오브젝트를 만들려면 구체적인 클래스 하나를 선택해야겠지만 인터페이스로 추상해놓은 최소한의 통로를 통해 접근하는 쪽에서 오브젝트를 만들 때 사용할 클래스가 무엇인지 몰라도 된다. 인터페이스를 통해 접근하게 되면 실제 구현 클래스를 바꿔도 신경 쓸 일이 없다.
```

<br />

<img src="https://leejaedoo.github.io/assets/img/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4.jpeg">

<br />

 * 인터페이스

    - 인터페이스를 통해 원하는 기능을 사용하기만 하면 된다.

    - 인터페이스는 어떤 일을 하겠다는 기능만 정의해놓는 것이다. 

    - 인터페이스에는 어떻게 하겠다는 구현 방법은 나타나 있지 않다. 

    - 인터페이스를 구현한 클래스들이 알아서 결정할 일이다.

<br />


```Java

public class DConnectionMaker implements ConnectionMaker{
    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
    }
}

public class UserDao {

    private ConnectionMaker connectionMaker;        // 인터페이스를 통해 오브젝트에 접근하므로 구체적인 클래스 정보를 알 필요가 없다.

    public UserDao(){
        this.connectionMaker = new DConnectionMaker();      // 클래스 이름이 나오네?...
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();        // 인터페이스에 정의된 메소드를 사용하므로 클래스가 바뀐다고 해도 메소드 이름이 변경될 걱정이 없다.
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
    }
}
```

> UserDao의 다른 모든 곳에서 인터페이스를 이용하게 만들어서 DB 커넥션을 제공하는 클래스에 대한 구체적인 정보는 모두 제거가 가능했지만, 초기에 한번 어떤 클래스의 오브젝트를 사용할지를 결정하는 생성자의 코드는 제거되지 않고 남아 있다.

<br />
<hr />
<br />

## 1.3.3 관계설정 책임의 원리

```
 문제점 
 
 UserDao와 ConnectionMaker는 두 개의 관심을 인터페이스를 사용하여 거의 완벽하게 분리했지만 UserDao가 인터페이스뿐 아니라 구체적인 클래스까지 알아야 한다.

 원인 
 UserDao에 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하는 코드가 남아있기 때문이다.
 
 -> 인터페이스를 이용하여 분리했음에도 불구하고 여전히 UserDao 변경 없이는 DB 커넥션 기능의 확장이 자유롭지 못한다.

 해결책
 UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계를 설정해주는 것에 관한 관심이다. 

 * 오브젝트 관계 *  
 두 개의 오브젝트가 있고 한 오브젝트가 다른 오브젝트의 기능을 사용한다면, 사용되는 쪽이 사용하는 쪽에게 서비스를 제공하는 셈이다. 사용되는 오브젝트를 서비스, 사용하는 오브젝트를 클라이언트라 부를 수 있다.

 오브젝트 사이의 관계는 런타임 시에 한쪽이 다른 오브젝트의 레퍼런스를 갖고 있는 방식으로 만들어진다.

 예를 들면, DConnectionMaker의 오브젝트의 레퍼런스를 UserDao의 connectionMaker 변수에 넣어서 사용하게 함으로써, 이 두개의 오브젝트가 '사용'이라는 관계를 맺게 해준다.

 * 오브젝트 사이의 관계 형성 방법
    1. 직접 생성자를 호출하는 방법
    2. 외부에서 만들어 준 것을 가져오는 방법(메소드 파라미터, 생성자 파라미터 등)

UserDao 오브젝트가 동작하려면 특정 클래스의 오브젝트와 관게를 맺어야 한다. 
-> DB 커넥션을 제공하는 기능을 가진 오브젝트를 사용해야하기 때문이다. 

클래스 사이의 관계는 코드에 다른 클래스 이름이 나타나기 때문에 만들어지는 것이지만, 오브젝트 사이의 관계는 해당 클래스가 구현한 인터페이스를 사용했다면 그 클래스의 오브젝트를 인터페이스 타입으로 받아서 사용할 수 있다. (객체지향 프로그래밍의 다형성)
``` 

<br />

<img src="https://leejaedoo.github.io/assets/img/%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C%EC%9D%98%EC%A1%B4%EA%B4%80%EA%B3%84.jpeg">

<br />

- UserDao 오브젝트가 DConnectionMaker 오브젝트를 사용하게 하려면 두 클래스의 오브젝트 사이에 런타임 사용관계 또는 링크, 또는 의존관계라고 불리는 관계를 맺어주면 된다.

<br />

<img src="https://user-images.githubusercontent.com/40616436/75683933-5280fe00-5cdb-11ea-9c4c-a5a8ad5f3885.png">

<br />

- UserDao도 아니고 DConnectionMaker와 같은 ConnectionMaker 구현 클래스도 아닌 제3의 오브젝트라 했던 UserDao의 클라이언트는 런타임 오브젝트 관계를 갖는 구조로 만들어주는게 클라이언트의 책임이다.

- 현재 UserDao 클래스의 main메소드가 UserDao 클라이언트이다.

```Java
public UserDao(ConnectionMaker connectionMaker){
    this.connectionMaker = connectionMaker;   // << 클래스 이름이 나오는 문제점
}
```

- DConnectionMaker은 왜 사라졌을까? 
> DConnectionMaker를 생성하는 코드는 UserDao와 특정 ConnectionMaker 구현 클래스의 오브젝트 간 관계를 맺는 책임을 담당하는 코드이기 때문에 그것을 UserDao의 클라이언트에게 넘겨버렸다.

<br />

리스트 1-12 관계설정 책임이 추가된 UserDao 클라이언트인 main 메소드

<br />

```Java
public class UserDaoTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        // UserDao가 사용할 ConnectionMaker 구현 클래스를 결정하고 오브젝트를 만든다.
        ConnectionMaker connectionMaker = new DConnectionMaker();

        // 1. UserDao 생성
        // 2. 사용할 ConnectionMaker 타입의 오브젝트 제공
        // 결국 두 오브젝트 사이의 의존관계 설정 효과
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

- UserDaoTest는 UserDao와 ConnectionMaker 구현 클래스와의 런타임 오브젝트 의존 관계를 설정하는 책임을 담당해야 하므로 특정 ConnectionMaker 구현 클래스의 오브젝트를 만들고, UserDao 생성자 
파라미터에 넣어 두 개의 오브젝트를 연결해준다.

- UserDao는 자신의 관심사이자 책임인 사용자 데이터 액세스 작업을 위해 SQL을 생성하고, 이를 실행하는 데만 집중할 수 있게 됐다. 더 이상 DB 생성 방법이나 전략에 대해서는 조금도 고민할 필요가 없다. 

- DB 커넥션을 가져오는 방법을 어떻게 변경하든 UserDao 코드는 아무런 영향을 받지 않는다.

그림 1-7 관계설정 책임을 담당한 클라이언트 UserDaoTest가 추가된 구조
<img src="https://leejaedoo.github.io/assets/img/%EA%B4%80%EA%B3%84%EC%84%A4%EC%A0%95%EC%B1%85%EC%9E%84.jpeg">

