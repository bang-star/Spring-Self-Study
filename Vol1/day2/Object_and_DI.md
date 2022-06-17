# 1장. Object_and_DI
- page : p.53 ~ 

```
- 스프링이 자바에서 가장 중요하게 가치를 두는 것은 객체지향 프로그래밍이 가능한 언어라는 점

- 스프링의 핵심 철학은 자바 엔터프라이즈 기술의 혼란 속에서 잃어버렸던 객체지향 기술의 진정한 가치를 회복시키고, 객체지향 프로그래밍이 제공하는 폭넓은 헤택을 누릴 수 있도록 기본으로 돌아가자는 것이다.

=> 스프링이 가장 관심을 많이 두는 것은 Object
=> 스프링을 이해라면 Object에 깊은 관심을 가져야 함
=> 애플리키에시녕에서 Object 생성, 관계, 사용, 소멸의 전 과정을 진지하게 생각해볼 필요가 있다.
=> Object는 어떻게 설계돼야 하는지, 어떤 단위로 만들어지며 어떤 과정을 통해 자신의 존재를 드러내고 등장해야 하는지에 대해서도 살펴봐야한다.
```

> 정리하면 객체지향 설계의 기초와 원칙을 비롯해서, 다양한 목적을 위해 재활용 가능한 설계 방법인 디자인 패턴, 좀 더 깔끔한 구조가 되도록 지속적으로 개선해나가는 작업인 리팩토링, 오브젝트가 기대한 대로 동작하고 있는지를 효과적으로 검증하는 데 쓰이는 단위 테스트와 같은 오브젝트 설계와 구현에 관한 여러 가지 응용 기술과 지식이 요구된다.

<br />

### 학습 목표
  - 1장에서는 스프링이 어떤 것이고, 무엇을 제공하는지보다는 스프링이 관심을 갖는 대상인 Object의 설계와 구현, 동작원리에 더 집중해야한다.

<br />

## 1.1 초난감 DAO

> 문제 : 사용자 정보를 JDBC API를 통해 DB에 저장하고 조회할 수 있는 간단한 DAO를 만들어라.

* DAO : DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 Object

<br />

### 1.1.1 User
- 사용자 정보를 저장할 때는 자바빈 규약을 따르는 오브젝트를 이용하면 편리한다.

- User 클래스
```Java
package com.example.week1.domain;

public class User {
    String id;
    String name;
    String password;
    
    public String getId(){
        return id;
    }
    
    public void setId(String id){
        this.id = id;
    }
    
    public String getName(){
        return name;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public String getPassword(){
        return password;
    }
    
    public void setPassword(String password){
        this.password = password;
    }
}

```

- User 테이블
```SQL
create table users(
  id varchar(10) primary key,
  name varchar(20) not null,
  password varchar(10) not null
)
```

```
* 자바빈
- 자바빈(JavaBean)은 원래 비주얼 툴에서 조작 가능한 컴포넌트를 말한다. 자바의 주력 개발 플랫폼이 웹 기반의 엔터프라이즈 방식으로 바뀌면서 비주얼 컴포넌트로서 자바빈은 인기를 잃어갔지만, 자바빈의 몇 가지 코딩 관례는 JSP 빈, EJB와 같은 표준 기술과 자바빈 스타일의 오브젝트를 사용하는 오픈소스 기술을 계속 이어져 왔다. 자바빈이라고 말하면 비주얼 컴포넌트라기보다는 다음 두 가지 관례를 따라 만들어진 오브젝트를 가리킨다. 간단히 빈이라고 부르기도 한다.

1. 디폴트 생성자 : 자바빈은 파리멑가 없는 디폴트 생성자를 갖고 있어야 한다. 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하다.

2. 프로퍼티 : 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 한다. 프로퍼티는 set으로 시작하는 수정자 메소드와 get으로 시작하는 접근자 메소드를 이용해 수정 또는 조회할 수 있다.
```

<br />
<hr />
<br />

### 1.1.2 UserDao

- 사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스
- 사용자 정보를 관리하는 DAO 이므로 UserDao라는 이름으로 클래스를 생성한다.

- JDBC를 이용하는 작업의 일반적인 순서

  1. DB 연결을 위한 Connection을 가져온다.
  2. SQL을 담은 Statement(또는 PreparedStatement)를 만든다.
  3. 만들어진 Statement를 실행한다.
  4. 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트(User)에 옮겨준다.
  5. 작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
  6. JDBC API가 만들어내는 예외(Exception)을 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생하면 메소드 밖으로 던지게 한다.

<br />

- 아래 클래스가 제대로 동작하는지 확인하는 방법은 DAO의 기능을 사용하는 웹 애플리케이션을 만들어 서버에 배치하고, 웹 브라우저를 통해 DAO 기능을 사용해보는 것이다.

```Java
package com.example.week1.dao;

import com.example.week1.domain.Users;

import java.sql.*;

public class UserDao {

    public void add(Users user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?, ?, ?)");

        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

    ps.executeUpdate();                                 // Query 실행

        ps.close();                                     // preparedStatement 닫기
        c.close();                                      // Connection 종료
    }

    public Users get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");

        PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        Users user = new Users();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```

<br />
<hr />
<br />

## 1.1.3 main()을 이용한 DAO 테스트 코드

- 코드의 기능을 검증하고자 할 때 사용할 수 있는 가장 간단한 방법은 오브젝트 스스로 자신을 검증하도록 만들어주는 것

- UserDao 클래스 코드의 문제점은 무엇일까?
  
  - 왜 이 코드에 문제가 많다고 하는 것일까?
  - 잘 동작하는 코드를 굳이 수정하고 개선해야하는 이유는 무엇일까?
  - DAO 코드를 개선했을 때의 장점은 무엇일까?
  - 장점들이 당장에, 또는 미래에 주는 유익은 무엇인가?
  - 객체지향 설계의 원칙과는 무슨 상관이 있을까?
  - DAO를 개선하는 경우와 그대로 사용하는 경우, 스프링을 사용하는 개발에서 무슨 차이가 있을까?

```
1. 스프링을 공부한다는 것은 바로 이런 문제 제기와 의문에 대한 답을 찾아나가는 과정이다.
2. 스프링을 이용해 개발자 스스로 만들어내는 것이지, 스프링이 덥석 해답을 줄 수 있는 게 아니다.
3. 스프링은 단지 그 과정에서 고민을 제대로 하고 있는지 끊임없이 확인해주고, 좋은 결론을 내릴 수 있도록 객체지향 기술과 자바 개발의 선구자들이 먼저 고민하고 제안한 방법에 대한 힌트를 제공해줄 뿐이다.
```

