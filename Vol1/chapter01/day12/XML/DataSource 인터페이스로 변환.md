# 1.8.3 DataSource 인터페이스로 변환(p.136 ~ 139)

## DataSource 인터페이스 적용

- DataSource는 getConnection()이라는 DB 커넥션을 가져오는 기능 외에도 여러 개의 메소드를 갖고 있어서 인터페이스를 직접 구현하기는 부담스럽다.

- DataSource를 구현해서 DB 커넥션을 제공하는 클래스를 만들 일은 거의 없다. 이미 다양한 방법으로 DB 연결과 풀링 기능을 갖춘 많은 DataSource 구현 클래스가 존재하고, 이를 가져다 사용하면 충분하기 때문이다.

- DataSource 인터페이스에서 실제로 관심을 가질 것은 getConnection() 메소드 하나뿐이다. 이름만 다르지 ConnectionMaker 인터페이스의 makeConnection()과 목적이 동일한 메소드다.

<br />

리스트 1-41. DataSource 인터페이스
```Java
public interface DataSource extends CommonDataSource, Wrapper {
    Connection getConnection() throws SQLException;
    ...
}
```

리스트 1-42. DataSource를 사용하는 UserDao
 ```Java
 public class UserDao {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource){
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException {
        Connection c = dataSource.getConnection();
        ...
    }

    ...
 }
 ```

- 스프링이 제공해주는 DataSource 구현 클래스 중에 테스트환경에서 간단히 사용할 수 있는 SimpleDriverDataSource 클래스를 사용하도록 DI를 재구성할 것이다.

- SimpleDriverDataSource는 DB 연결에 필요한 필수 정보를 제공받을 수 있도록 여러 개의 수정자 메소드를 갖고 있다. 예를 들어 JDBC 드라이버 클래스, JDBC URL, 아이디, 비밀번호 등이다.

<br />

## 자바 코드 설정 방식

- connectionMaker() 메소드를 dataSource()로 변경하고 SimpleDriverDataSource 오브젝트를 리턴

- DB 연결과 관련된 정보를 수정자 메소드를 이용해 지정

    리스트 1-43. DataSource 타입의 dataSource 빈 정의 메소드
    ```Java
    @Bean
    public DataSource dataSource(){
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        
        dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
        dataSource.setUrl("jdbc:mysql://localhost/springbook");
        dataSource.setUsername("spring");
        dataSource.setPassword("book");
        
        return dataSource;
    }
    ```

    리스트 1-44. DataSource 타입의 빈을 DI 받는 userDao() 빈 정의 메소드
    ```Java
    @Bean     // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
    public UserDao userDao(){
        UserDao userDao = new UserDao();
        userDao.setDataSource(dataSource());
        return userDao;
    }
    ```

- UserDao에 DataSource 인터페이스를 적용하고 SimpleDriverDataSource의 오브젝트를 DI로 주입해서 사용할 수 있다.

<br />

## XML 설정 방식

- id가 connectionMaker인 bean을 없애고 dataSource라는 이름의 bean을 등록한다. 클래스를 SimpleDriverDataSource로 변경해주면 리스트 1-45와 같이 bean 설정이 만들어진다.

리스트 1-45.dataSource 빈
```XML
<bean id="dataSource" class="org.springframework.jdbc.SimpleDriverDataSource" />
```

- 문제는 bean 설정으로 SimpleDriverDataSource의 오브젝트를 만드는 것까지 가능하지만, dataSource() 메소드에서 SimpleDriverDataSource 오브젝트의 수정자로 넣어준 DB 접속정보는 나타나 있지 않다는 점이다.

- UserDao처럼 다른 빈에 의존하는 경우에는 property 태그와 ref 애트리뷰트로 의존할 빈 이름을 넣어주면 된다.

- SimpleDriverDataSource 오브젝트의 경우는 단순 Class 타입의 오브젝트나 텍스트 값이다.
