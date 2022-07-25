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