# 2.3 개발자를 위한 테스팅 프레임워크 JUnit(p. 161 ~ 182)

## JUnit 
  
  - JUnit(제이유닛)은 자바 프로그래밍 언어용 유닛 테스트 프레임워크이다. 

  - JUnit은 테스트 주도 개발 면에서 중요하며 SUnit과 함께 시작된 XUnit이라는 이름의 유닛 테스트 프레임워크 계열의 하나이다.
 
  - JUnit 테스트는 main() 메소드와 System.out.println()으로 만든 테스트만큼 단순하기 때문에 빠르게 작성할 수 있다.

  - 테스트 작성 시 자주 편리하고 필요한 여러 가지 부가기능도 제공한다.

  - 대부분의 자바 IDE는 JUnit 테스트를 손쉽게 실행할 수 있는 JUnit 테스트 지원 기능을 내장하고 있어서 더욱 편리하게 JUnit 테스트를 만들고 활용할 수 있게 해준다.

<br />

## 2.3.1 JUnit 테스트 실행 방법

  - JUnitCore를 이용해 테스트를 실행하고 콘솔에 출력된 메시지를 보고 결과를 확인하는 방법은 가장 간단하지만 테스트의 수가 많아지면 관리하기가 힘들어진다는 단점이 있다.

  - 가장 좋은 JUnit 테스트 실행 방법은 자바 IDE에 내장된 JUnit에 내장된 JUnit 테스트 지원 도구를 사용하는 것이다.

<br />

### IDE

  - IDE 테스트를 진행하면 뷰에서 테스트의 총 수행시간, 실행한 테스트의 수, 테스트 에러의 수, 테스트 실패의 수를 확인할 수 있다. 또한 어떤 테스트 클래스를 실행했는지도 알 수 있다.
  
  - 테스트 클래스 내에 있는 @Test가 붙은 테스트 메소드의 이름도 보여준다.

  - 각 테스트 메소드와 클래스의 테스트 수행에 걸린 시간을 보여준다.

  - 테스트가 실패할 경우 뷰를 통해 자세히 알 수 있다.

  - 테스트 코드에서 검증에 실패한 assertThat()의 위치를 파악할 수 있다.

  - JUnit은 한 번에 여러 테스트 클래스를 동시에 실행할 수 있다.

  - IDE에 소스 트리에서 특정 패키지를 선택하여 테스트를 실행하면 해당 패키지 아래에 있는 모든 JUnit 테스트를 한 번에 실행해준다.

<br />

### 빌드 툴

  - 프로젝트의 빌드를 위해 ANT나 Maven 같은 빌드 툴과 스크립트를 사용하고 있다면, 빌드 툴에서 제공하는 JUnit 플러그인이나 Task를 이용해 JUnit 테스트를 실행할 수 있다.

  - 테스트 실행 결과는 옵션에 따라서 HTML이나 텍스트 파일의 형태로 보기 좋게 만들어진다.

  - 여러 개발자가 만든 코드를 모두 통합해서 테스트를 수행해야 할때가 있는데, 이런 경우에는 서버에서 모든 코드를 가져와 통합하고 빌드한 뒤에 테스트를 수행하는 것이 좋다.

  - 빌드 스크립트를 이용해 JUnit 테스트를 실행하고 그 결과를 메일 등으로 통보받는 방법을 사용하면 된다.

<br />
<hr />
<br />

## 2.3.2 테스트 결과의 일관성

  - 현재 테스트가 외부 상태에 따라 성공하기도 하고 실패하기도 한다는 문제가 있다.

    (예시) DB 서버가 다운됐다거나 네트워크에 장애로 DB에 접근하지 못하는 예외적인 상황

    정리하면, 별도의 준비 작업 없이는 성공해야 마땅한 테스트가 실패하기도 한다는 문제점이 있다.

  - 반복적으로 테스트를 했을 때 테스트가 실패하기도 하고 성공하기도 한다면 이는 좋은 테스트라고 할 수가 없다. 코드에 변경사항이 없다면 테스트는 항상 동일한 결과를 내야 한다.

  - UserDaoTest의 문제는 이전 테스트 때문에 DB에 등록된 중복 데이터가 있을 수 있다는 점이다. 

  - 가장 좋은 해결책은 addAndGet() 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제해서, 테스트를 수행하기 이전 상태로 만들어주는 것이다. 그러면 테스트를 아무리 여러 번 반복해서 실행하더라도 항상 동일한 결과를 얻을 수 있다.

<br />

### deleteAll()의 getCount() 추가

  - deleteAll
     
     USER 테이블의 모든 레코드를 삭제해주는 간단한 기능

리스트 2-7. deleteAll() 메소드
```Java
public void deleteAll() throws SQLException {
    Connection c = dataSource.getConnection();

    PreparedStatement ps = c.prepareStatement("delete from users");

    ps.executeUpdate();

    ps.close();
    c.close();
}
```

  add() 메소드와 비슷한 구조인데 파라미터 바인딩이 없으므로 더 단순하다.

<br />

  - getCount()
    
    두 번째 추가할 것은 getCount() 메소드로, USER 테이블의 레코드 개수를 돌려준다.

리스트 2-8. getCount() 메소드
```Java
public int getCount() throws SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement("select count(*) from users");

  ResultSet rs = ps.executeQuery();
  rs.next();
  int count = rs.getInt(1);

  rs.close();
  ps.close();
  c.close();

  return count;
}
```

<br />

### deleteAll()의 getCount() 추가

  - deleteAll()과 getCount() 메소드의 기능은 add()와 get()처럼 독립적으로 자동 실행되는 테스트를 만들기가 애매하다. 굳이 테스트를 하자면 USER 테이블에 수동으로 데이터를 넣고 deleteAll()을 실행한 뒤에 테이블에 남은 게 있는지 확인해야 하는데, 사람이 테스트 과정에 참여해야 하니 자동화돼서 반복적으로 실행 가능한 테스트 방법은 아니다.

  - 새로운 테스트를 만들기보다는 차라리 기존에 만든 addAndGet() 테스트를 확장하는 방법을 사용하는 편이 더 나을 것 같다.

  - addAndGet() 테스트의 불편한 점은 실행 전에 수동으로 USER 테이블의 내용을 모두 삭제해줘야 한다.

  - deleteAll()이 잘 작동한다면 addAndGet() 테스트를 위해 매번 USER테이블을 수동으로 삭제하는 수고를 하지 않아도 된다. 하지만 deleteAll()을 넣는 것만으로는 조금 부족하다. deleteAll() 자체도 아직 검증이 되지 않아 다른 테스트에 적용할 수 없다.

  - deleteAll()이 기대한 대로 동작한다면, getCount()로 레코드의 개수를 가져올 경우 0이 나와야 한다.

  - 검증되지 않은 두 메소드를 사용하는 것은 바람직하지 못하다.

  - getCount()에 대한 검증을 위해 add 메소드를 수행하고 나면 레코드 개수가 0에서 1로 바뀌게 되면 이때 getCount()의 결과를 확인하여 동작을 확인할 수 있다.

<br />

리스트 2-9. deleteAll()과 getCount()가 추가된 addAndGet() 테스트
```Java
@Test
public void addAndGet() throws SQLException {
    ...

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    User user = new User();
    user.setId("gyumee");
    user.setName("박성철");
    user.setPassword("springno1");

    dao.add(user);
    assertThat(dao.getCount(), is(1));

    User user2 = dao.get(user.getId());

    assertThat(user2.getName(), is(user.getName()));
    assertThat(user2.getPassword(), is(user.getPassword()));
}
```

<br />

### 동일한 결과를 보장하는 테스트
  
  - 테스트를 반복해서 여러 번 실행해도 DB의 테이블을 삭제하는 작업 없이 계속 성공할 것이다. 

  - 단위 테스트는 코드가 바뀌지 않는다면 매번 실행할 때마다 동일한테스트 결과를 얻을 수 있어야 한다.

  - addAndGet() 테스트를 마치기 직전에 테스트가 변경하거나 추가한 데이터를 모두 원래 상태로 만들어주는 방법이 있다.
    
    - add()를 호출해 데이터 삽입이 완료되면, deleteAll()을 실행하여 등록된 레코드를 삭제하는 방법

    - 만약 addAndGet() 테스트 실행 이전에 다른 이유로 USER 테이블에 데이터가 들어가 있다면 이때는 테스트가 실패할 수 있다.

  - 단위 테스트는 항상 일관성 있는 겨로가가 보장돼야 한다는 점을 잊어선 안된다. DB에 남아 있는 데이터와 같은 외부 환경에 영향을 받지 말아야 하는 것을 물론이고, 테스트를 실행하는 순서를 바꿔도 동일한 결과가 보장되도록 만들어야 한다.

<br />
<hr />
<br />

## 2.3.3 포괄적인 테스트

  - getCount() 메소드 리뷰

     - 기존의 테스트에서 확인할 수 있었던 것은 deleteAll()을 실행했을 때 테이블이 비어 있는 경우(0)과 add()를 한 번 호출한 뒤의 결과(1)

     - 두 개 이상의 레코드를 add() 했을 때 getCount()의 실행 결과는 0과 1 두 가지를 해봤으니 나머지도 당연히 잘될 것이라고 추정할 수도 있겠지만 미처 생각하지 못한 문제가 숨어 있을수도 있으므로 꼼꼼한 테스트를 해보는 것이 좋다.

     - 테스트를 안 만드는 것도 위험한 일이지만, 성의 없이 테스트를 만드는 바람에 문제가 있는 코드인데도 테스트가 성공하게 만드는 건 더 위험하다. 

     - 한 가지 결과만 검증하고 마는 것은 상당히 위험하다. 

<br />

### getCount() 테스트

  - 테스트 기능을 기존의 addAndGet() 메소드에 추가하는 것은 좋은 생각이 아니다. 테스트 메소드는 한 번에 한 가지 검증 목적에만 충실한 것이 좋다.

  - JUnit은 하나의 클래스 안에 여러 개의 테스트 메소드가 들어가는 것을 허용한다. 

    - @Test가 붙어 있고 public 접근자가 있으며 리턴 값이 void형이고 파라미터가 없다는 조건을 지키기만 하면 된다.

  - 테스트 시나리오
     
     - USER 테이블의 데이터를 모두 지우고 getCount()로 레코드의 개수가 0임을 확인

     - 3개의 사용자 정보를 하나씩 추가하면서 매번 getCount()의 결과가 하나씩 증가하는지 확인하는 것

     - 테스트를 만들기 전에 User 클래스에 한 번에 모든 정보를 넣을 수 있도록 초기화가 가능한 생성자를 추가한다.

     - User 오브젝트를 여러 번 만들고 값을 넣어야 하므로 한 번에 설정 가능한 생성자를 만들어 두면 편리하다.

<br />

리스트 2-10. 파라미터가 있는 User 클래스 생성자
```Java
public User(String id, String name, String password){
    this.id = id;
    this.name = name;
    this.password = password;
}

public User(){
    // 자바빈의 규약을 따르는 클래스에 생성자를 명시적으로 추가했을 때는 파라미터가 없는 디폴트 생성자도 함께 저으이해주는 것을 잊지 말자.
}
```

리스트 2-11. getCount() 테스트
```Java
@Test
public void count() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);

    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");
    User user3 = new User("bumjin", "박범진", "springno3");

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
    assertThat(dao.getCount(), is(1));

    dao.add(user2);
    assertThat(dao.getCount(), is(2));

    dao.add(user3);
    assertThat(dao.getCount(), is(3));
    
}
```

  - 세 개의 User 오브젝트를 준비해두고 deleteAll()을 불러 테이블의 내용을 모두 삭제한 뒤에 getCount()가 0임을 확인한다. 그리고 만들어둔 User 오브젝트들을 하나씩 넣으멶서 매번 getCount()가 하나씩 증가되는지를 확인하면 된다.

  - 주의해야 할 점은 두 개의 테스트가 어떤 순서로 실행될지는 알 수 없다는 것이다.

  - JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않는다. 

  - 테스트의 결과가 테스트 실행 순거에 영향을 받는다면 테스트를 잘못 만든 것이다.

  - 예를 들어 addAndGet() 메소드에서 등록한 사용자 정보를 count() 테스트에서 활용하는 식으로 테스트를 만들면 안 된더ㅏ.

  - 모든 테스트는 실행 순서에 상관없이 독립적으로 항상 동일한 결과를 낼 수 있도록 해야 한다.

<br />

### addAndGet() 테스트 보완

  - add() 후에 레코드 개수도 확인하도록 했고, get()으로 읽어와서 값도 모두 비교해봤으니 add()의 기능은 충분히 검증된 것 같다.

  - id 조건으로 해서 사용자를 검색하는 기능을 가진 get()에 대한 테스트는 조금 부족한 감이 있다. get()이 파라미터로 주어진 id에 해당하는 사용자를 가져온 것인지, 그냥 아무거나 가져온 것인지 테스트에서 검증하지는 못했다.

  - User를 하나 더 추가해서 두 개의 User를 add() 하고, 각 User의 id를 파라미터로 전달해서 get()을 실행하도록 만들어 주어진 id에 해당하는 정확한 User 정보를 가져오는지 확이할 수 있다.

리스트 2-12. get() 테스트 기능을 보완한 addAndGet() 테스트
```Java
@Test
public void addAndGet() throws SQLException {
    ...

    UserDao dao = context.getBean("userDao", UserDao.class):
    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
}
```

### get() 예외조건에 대한 테스트

  - get() 메소드에 전달된 id 값에 해당하는 사용자 정보가 없을 때 해결방법

     1. null과 같은 특별한 값을 리턴하는 것

     2. id에 해당하는 정보를 찾을 수 없다고 예외를 던지는 것

  - id에 해당하는 정보가 없다는 의미를 가진 예외 클래스가 하나 필요(예외를 하나 정의할 수 있지만, 스프링이 미리 정의해놓은 예외(EmptyResultDataAccessException)를 가져다 사용

  - UserDao의 get() 메소드에서 쿼리를 실행해 결과를 가져왔을 때 아무것도 없으면 예외를 던지도록 만들면 된다.

  - 테스트 진행 중에 특정 예외가 던져지면 테스트가 성공한 것이고, 예외가 던져지지 않고 정상적으로 작업을 마치면 테스트가 실패하는 상황이다.
    
    - 문제는 예외 발생 여부는 메소드를 실행해서 리턴 값을 비교하는 방법으로 확인할 수 없다는 점이다. 즉, assertThat() 메소드로는 검증이 불가능하다.

  - JUnit은 예외조건 테스트를 위한 특별한 방법을 제공해준다.

<br />

리스트 2-13. get() 메소드의 예외상황에 대한 테스트
```Java
// 테스트 중에 발생할 것으로 기대하는 예외 클래스를 지정해준다.
@Test(expected=EmptyResultDataAccessException.class)   
public void getUserFailure() throws SQLException {
    ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    // 이 메소드 실행 중에 예외가 발생해야 한다. 예외가 발생하지 않으면 테스트가 실패한다.
    dao.get("unknown_id");  
}
```

  - 모든 User 테이터를 지우고 존재하지 않는 id로 get() 메소드를 실행하는 게 전부다.

  - 테스트에서 중요한 것은 @Test 애토네이션의 expected 엘리먼트다. 
     
     - expected는 테스트 메소드 실행 중에 발생하리라 기대하는 예외 클래스를 넣어주면 된다.

     - @Test에 expected를 추가해놓으면 보통의 테스트와는 반대로, 정상적으로 테스트 메소드를 마치면 테스트가 실패하고, expected에서 지정한 예외가 던져지면 테스트가 성공한다.

     - 예외가 반드시 발생해야 하는 경우를 테스트하고 싶을 떄 유용하게 쓸 수 있다.

  - 테스트를 실행하게 되면 테스트는 실패한다.

    1. get()메소드에서 쿼리 결과의 첫 번째 row를 가져오게 하는 rs.next()를 실행할 때 가져올 로우가 없다는 SQLException이 발생할 것이다. 이는 UserDao 코드에 수정이 필요한 것을 의미한다.

<br />

### 테스트를 성공시키기 위한 코드의 수정

리스트 2-14. 테스트를 찾지 못하면 예외를 발생시키도록 수정한 get()메소드
```Java
public User get(String id) throws SQLException{
    ...
    ResultSet rs = ps.executeQuery();

    User user = null;   // User는 null 상태로 초기화
    if(rs.next()){
        // id를 조건으로 한 쿼리의 결과가 있으면 User 오브젝트를 만들고 값을 넣어준다.
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));
    }

    rs.close();
    ps.close();
    c.close();

    if(user == null) throw new EmptyResultDataAccessException(1);

    return user;
}
```

  - 모든 테스트가 성공하면, 새로 추가한 기능도 정상적으로 동작하고 기존의 기능에도 영향을 주지 않았다는 확신을 얻을 수 있다.

<br />

### 포괄적인 테스트

 - DAO의 메소드에 대한 포괄적인 테스트를 만들어두면 안전하고 유용하다.

   - 문제가 발생했을 때 원인을 찾기 힘들어서 고생하게 될지도 모른다.

   - 단순하고 간단한 테스트가 치명적인 실수를 피할 수 있게 해주기도 한다.

 - 개발자가 테스트를 직접 만들 떄 자주 하는 실수

   - 성공하는 테스트만 골라서 만드는 것이다. 

   - 개발자는 머릿속으로 코드가 잘 돌아가는 케이스를 상상하면서 코드를 만드는 경우가 일반적이다. 그래서 테스트를 작성할 때도 문제가 될 만한 상황이나, 입력 값 등은 교모히도 잘 피해서 코드를 만드는 습성이 있다. 

   - 테스트 코드를 통한 자동 테스트뿐 아니라, UI를 통한 수동 테스트를 할 때도 빈번하게 발생하는 문제다.

  - 조금만 신경을 쓰면 자신이 만든 코드에서 발생할 수 있는 다양한 상황과 입력 값을 고려하는 포괄적인 테스트를 만들 수 있다.

    - 스프링의 창시자인 로드 존슨은 "항상 네거티브 테스트를 먼저 만들라"는 조언을 했다. 개발자는 빨리 테스트를 만들어 성공하는 것을 보고 다음 기능으로 나가고 싶어하기 때문에, 긍정적인 경우를 골라서 성공할 만한 테스트를 먼저 작성하게 되기가 쉽다.

    - 테스트를 작성할 때는 부정적인 케이스를 먼저 만드는 습관을 들이는 게 좋다. get() 메소드의 경우라면, 존재하는 id가 주어졌을 때 해당 레코드를 정확히 가져오는가를 테스트하는 것도 중요하지만, 존재하지 않는 id가 주어졌을 때는 어떻게 반응할지를 먼저 결정하고, 이를확인할 수 있는 테스트를 먼저 만들려고 한다면 예외적인 상황을 빠뜨리지 않는 꼼꼼한 개발이 가능하다.

<br />
<hr />
<br />

## 2.3.4 테스트가 이끄는 개발

  - get 메소드의 예외 테스트를 만드는 과정을 다시 돌아보면 한 가지 흥미로운 점을 발견할 수 있다.

    - 새로운 기능을 넣기 위해 UserDao 코드를 수정하고, 검증하기 위해 테스트를 만드는 순서로 진행한 것이 아니다.

    - 테스트를 먼저 만들어 테스트가 실패하는 것을 보고 UserDao를 수정했다.

    - 이러한 순서를 따라서 개발을 진행하는 구체적인 개발 전략이 실제로 존재한다.

<br />

### 기능 설계를 위한 테스트

  - '존재하지 않는 id로 get()메소드를 실행하면 특정한 예외가 던져져야 한다'는 식으로 만들어야 할 기능을 결정하고 UserDao 코드를 수정하는 대신 getUserFailure() 테스트를 만들었다.

  - 어떻게 테스트할까라는 생각하면서 만든 것이 아니라, 추가하고 싶은 기능을 코드로 표현하려고 했기 때문에 가능했다.

  - getUserFailture() 테스트에는 만들고 싶은 기능에 대한 조건과 행위, 결과에 대한 내용이 잘 표현되어 있다.

    <img src="data:image/jpeg;base64,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">

  - 비교해보면 테스트 코드는 마치 잘 작성된 하나의 기능정의서처럼 보인다. 

  - 보통 기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 이 테스트 코드가 일부분 담당하고 있다고 볼 수 있다.

  - 이와 같이 추가하고 싶은 기능을 일반 언어가 아니라 테스트 코드로 표현해서, 마치 코드로 된 설계문서처럼 만들어놓은 것이라고 생각했을 때, 실제 기능을 가진 애플리케이션 코드를 만들고 나면 테스트를 실행해서 설계한 대로 코드가 동작하는지를 빠르게 검증할 수 있다.

  - 테스트가 실패하면 이때는 설계한 대로 코드가 만들어지지 않았음을 바로 알 수 있다. 문제가 되는 부분이 무엇인지에 대한 정보도 테스트 결과를 통해 얻을 수 있다.

  - 코드를 수정하고 테스트를 수행해서 테스트가 성공하도록 애플리케이션 코드를 게속 다듬어간다. 

<br />

### 테스트 주도 개발
  
  - 테스트 주도 개발(TDD, Test Driven Development)

    - 만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법이 있다. 
    
    - 테스트 우선 개발(Test First Development)이라고도 불린다.

  - TDD는 개발자가 테스트를 만들어가며 개발하는 방법이 주는 장점을 극대화한 방법이라고 불 수 있다.

  > "실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다"는 것이 TDD의 기본 원칙이다.

  - 이 원칙을 따랐다면 만들어진 모든 코드는 빠짐없이 테스트로 검증된 것이라고 볼 수 있다.

  - TDD는 아예 테스트를 먼저 만들고 테스트가 성공하도록 하는 코드만 만드는 식으로 진행하기 때문에 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있다.

  - 테스트를 작성하는 시간과 애플리케이션 코드를 작성하는 시간의 간격이 짧아진다. 

  - 코드를 만들고 테스트를 수행할 때까지 걸리는 시간은 0에 가깝고 이미 테스트를 만들어뒀기 때문에 코드를 작성하면 바로바로 테스트를 실행해볼 수 있기 때문이다.

    - 코드에 대한 피드백을 매우 빠르게 받을 수 있게 된다.

    - 매번 테스트가 성공하는 것을 보면서 작성한 코드에 대한 확신을 가질 수 있어, 가벼운 마음으로 다음 단계로 넘어갈 수가 있다. 

    - TDD에서는 테스트 작성하고 이를 성공시키는 코드를 만드는 브이 주기를 가능한 한 짧게 가져가도록 권장한다.

    - 테스트를 반나절 동안이나 만들고 오후 내내 테스트를 통과시키는 코드를 만드는 식의 개발은 좋은 방법이 아니다.

   - 코드를 만들어 테스트를 실행하는 그 사이의 간격이 짧다.

<br />
<hr />
<br />

## 2.3.5 테스트 코드 개선

  - 애플리케이션 코드만이 리팩토링의 대상이 아니다. 필요하다면 테스트 코드도 언제든지 내부구조와 설계를 개선해서 좀 더 깔금하고 이해하기 쉬우며 변경이 용이한 코드로 만들 필요가 있다.

  - 테스트 코드 자체가 이미 자신에 대한 테스트이기 때문에 테스트 결과가 일정하게 유지된다면 리팩토링을 해도 좋다.

  - 중복된 코드는 별도의 메소드로 뽑아내는 것이 가장 손쉬운 방법이다. JUnit 프레임워크는 테스트 메소드를 실행할 때 부가적으로 해주는 작업이 몇 가지 있다. 그중에서 테스트를 실행할 때마다 반복되는 준비 작업을 별도의 메소드에 넣게 해주고, 이를 매번 테스트 메소드를 실행하기 전에 먼저 실행시켜주는 기능이다.

<br />

### @Before

  - 중복되는 코드를 넣을 setUp()이라는 이름의 메소드를 만들고 테스트 메소드에서 제거한 코드를 넣는다. 

  - 로컬 변수를 인스턴스 변수로 변경하여 테스트 메소드에서 접근할 수 있도록 해야한다.

리스트 2-15. 중복 코드를 제거한 UserDaoTest
```Java
public class UserDaoTest {
    private UserDao dao;

    @Before
    public void setUp(){
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        this.dao = context.getBean("userDao", UserDao.class);
    }

    ... 

    @Test
    public void addAndGet() throws SQLException{
        ...
    }

    @Test
    public void count() throws SQLException {
        ...
    }

    @Test(expected=EmptyResultDataAccessException.class)
    public void getUserFailture() throws SQLException {
        ...
    }
}
```

  - JUnit 프레임워크가 테스트 메소드를 실행하는 과정을 알 필요가 있다.
    
    - 프레임워크는 스스로 제어권을 가지고 주도적으로 동작하고, 개발자가 만든 코드는 프레임워크에 의해 수동적으로 실행된다. 그래서 프레임워크에 사용되는 코드만으로는 실행 흐름이 잘 보이지 않기 때문에 프레임워크가 어떻게 사용할지를 잘 이해하고 있어야 한다.

    <br />

  - JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

    1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.

    2. 테스트 클래스의 오브젝트를 하나 만든다.

    3. @Before가 붙은 메소드가 있으면 실행한다.

    4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.

    5. @After가 붙은 메소드가 있으면 실행한다.

    6. 나머지 테스트 메소드에 대해 2~5번을 실행한다.

    7. 모든 테스트의 결과를 종합해서 돌려준다.

  - 실제로는 이보다 더 복잡한데, 간단히 정리하면 JUnit 테스트는 위의 7단계를 거쳐서 진행된다고 볼 수 있다.

  - JUnit은 @Test가 붙은 메소드를 실행하기 전과 후에 각각 @Before와 @After가 붙은 메소드를 자동으로 실행한다.

  - 보통 하나의 테스트 클래스 안에 있는 테스트 메소드들은 공통적인 준비 작업과 적리 작업이 필요한 경우가 많다.

  - @Before, @After가 붙은 메소드에 넣어두면 JUnit은 자동으로 메소드를 실행해주므로 매우 편리한다.

  - 유의 사항
    
    - @Before나 @After 메소드를 테스트 메소드에서 직접 호출하지 않기 때문에 서로 주고받을 정보나 오브젝트가 있다면 인스턴스 변수를 이용해야 한다.

      - UserDaoTest에서는 스프링 컨테이너에서 가져온 UserDao 오브젝트를 인스턴스 변수 dao에 저장해 뒀다가, 각 테스트 메소드에서 사용하게 만들었다.

    - 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다.

      - 한번 만들어진 테스트 클래스의 오브젝트는 하나의 테스트 메소드를 사용하고 나면 버려진다. 테스트 클래스가 @Test 테스트 메소드를 두 개 갖고 있다면, 테스트가 실행되는 중에 JUnit은 이 클래스의 오브젝트를 두 번 만들 것이다.

<img src="https://user-images.githubusercontent.com/63120360/180800463-51f9ddcf-4904-4ffd-b8f0-a8bf003680e0.jpg">