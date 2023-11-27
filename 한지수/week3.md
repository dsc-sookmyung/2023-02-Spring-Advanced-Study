# 3. 템플릿 

템플릿이란 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.

1. 다시보는 초난감 DAO

     1) 예외 처리 기능을 갖춘 DAO

정상적인 JDBC 코드의 흐름을 따르지 않고 중간에 어떤 이유로든 예외가 발생했을 경우에도 사용한 리소스를 반드시 반환하도록 만들어야 하기 때문에, JDBC 코드에는 예외처리 원칙을 반드시 지켜야 한다.

- JDBC 수정 기능의 예외처리 코드

```java
public void deleteAll() throws SQLException() {
	Conenction c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executeUpdate();

	ps.close();
	c.close();
}
```

이 메소드가 정상적으로 처리되면 Connection과 PreparedStatement는 각각 close()를 호출해 리소스를 반환한다. 그런데 PreparedStatement를 처리하는 중에 예외가 발생하면, close() 메소드가 실행되지 않아서 제대로 리소스가 반환되지 못하게 된다.

→ close()되지 못해서 반환되지 못한 connection이 계속 쌓이면 어느 순간 커넥션 풀에 여유가 없어지고 리소스가 모자란다는 심각한 오류를 내며 서버가 중간될 수 있다.

→ JDBC 코드에서는 어떤 상황에서도 가져온 리소스를 반환하도록 try/catch/finally 구문 사용을 권장한다.

```java
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;

	try {
		c = dataSource.getConnection();
		ps = c.prepareStatement("delete from users");
		ps.executeUpdate();
} catch (SQLException e) {
	throw e;
}  finally {
		if (ps != null) {
			try {
		ps.close();
	} catch(SQLException e) {
	}
}
if (c != null) {
		try {
	c.close();
} catch (SQLException e) {
			} 
		}
	}
}
```

finally는 try 블록을 수행한 후에 예외가 발생하든 정상적으로 처리되든 상관없이 반드시 실행되는 코드를 넣을 때 사용한다.

일단 try 블록으로 들어섰다면 반드시 Connection이나 PrepareStatement의 close() 호출을 통해 가져온 리소스를 반환해야 한다.

다만 어느 시점에서 예외가 발생했는지에 따라 close()를 사용할 수 있는 변수가 달라지기 때문에 finally에서는 반드시 c와 ps가 null이 아닌지 먼저 확인 후에 close() 메소드를 호출해야 한다. 이 close()도 SQLException이 발생할 수 있는 메소드이기 때문에 try/catch문으로 처리해줘야 한다.

- JDBC 조회 기능의 예외처리
    
    Connection, PreparedStatement 외에도 ResultSet의 close() 메소드가 반드시 호출되도록 만든다
    
    ```java
    public int getCount() throws SQLException() {
    	Connection c = null;
    	PreparedStatement ps = null;
    	ResultSet rs = null;
    
    	try {
    	c= dataSource.getConnection();
    	ps = c.prepareStatement("select count(*) from users");
    	
    	rs = ps.executeQuery();
    	rs.next();
    	return rs.getInt(1);
    	} catch(SQLException e) {
    			throw e;
    	} finally {
    		if (rs != null) {
    			try {
    				rs.close();
    			} catch (SQLException e) {
    		}
    	}
    		if (ps != null) {
    			try {
    				ps.close();
    				} catch (SQLExcetion e) {
    			}
    		}
    		if (c!= null) {
    			try {
    				c.close();
    				} catch (SQLExcetion e)
    			}
    		}
    	}
    }
    ```
    
    예외 상황에 대한 처리까지 모두 마쳤으니, 서버환경에서도 안정적으로 수행될 수 있으면서 DB 연결 기능을 자유롭게 확장할 수 있는 이상적인 DAO가 완성됐다.
    
1. 변하는 것과 변하지 않는 것
    
    1) JDBC try/catch/finally 코드의 문제점
    
    try/catch/finally 블롲도 적용돼서 완성도 높은 DAO 코드가 된 userDao이지만, 복잡한 블록이 2중으로 중첩되고 모든 메소드마다 반복된다는 문제점이 있다
    
    → Copy & Paste 신공을 사용한다.
    
    : 계속 코드를 복사해서 새로운 메소드를 만들고, 메소드마다 달라지는 일부 코드만 수정하는 방법
    
    but, 복사를 잘못하거나 삭제하면 기능은 잘 동작하는 것처럼 보이지만, 커넥션이 반환되지 않고 쌓이기 때문에 리소스가 꽉 찼다는 에러가 나면서 에러 발생
    
    → 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리하는 작업이 필요하다.
    
    2) 분리와 재사용을 위한 디자인
    
    - 메소드 추출
        
        변하는 부분을 메소드로 빼는 방법이다.
        
        → 메소드 추출 리팩토링을 적용하는 경우와 반대로, 분리시키고 남은 메소드가 재사용이 필요한 부분이고 분리된 메소드는 DAO 로직마다 새롭게 만들어서 확장돼야 하는 부분으로 잘못됐다.
        
    - 템플릿 메소드 패턴의 적용
        
        상속을 통해 기능을 확장해서 사용한다. 변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메소드로 정의해서, 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 방법이다.
        
        → 템플릿 메소드 패턴으로의 접근은 제한이 많다. 가장 큰 문제는, DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 점이다. 또 확장구조가 이미 클래스를 설계하는 시점에서 고정되어 버리므로 관계에 대한 유연성이 떨어진다.
        
    - 전략 패턴의 적용
        
        OCP 관점에서 보면 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식이다.
        
        → 컨텍스트 안에서 이미 구체적인 전략 클래스를 사용하도록 고정되어 있다면, 전략 패턴에도 OCP에도 잘 들어맞는다고 볼 수 없다.
        
    - DI 적용을 위한 클라이언트 / 컨텍스트 분리
        
        Clinet가 구체적인 전략의 하나를 선택하고 오브젝트로 만들어서 Context에 전달한다. Context는 전달받은 그 Strategy 구현 클래스의 오브젝트를 사용한다.
        
        이 구조에서 전략 오브젝트 생성과 컨텍스트로의 전달을 담당하는 책임을 분리시킨 것이 바로 ObjectFactory이며, 이를 일반화한 것이 앞에서 살펴봤던 의존관계 주입이었다. 결국 DI란 이러한 전략 패턴의 장점을 일반적으로 활용할 수 있도록 만든 구조이다.
        
        ```java
        public void jdbcContextStatementStrategy(StatementStrategy stmt) throws SQLException {
        		SQLException {
        			Connection c = null;
        			PreparedStatement ps = null;
        
        			try {
        					c = dataSource.getConnection();
        
        					ps = stmt.makePreparedStatement(c);
        
        					ps.executeUpdate();
        			}	catch (SQLException e) {
        					throw e;
        			} finally {
        					if(ps != null) { try {ps.close();} catch (SQLException e {} }
        					if(c != null) { try {c.close(); catch {SQLException e {} }
        			}
        		}
        ```
        
        이 메소드는 컨텍스트의 핵심적인 내용을 잘 담고있다. 클라이언트로부터 statementStategy 타입의 전략 오브젝트를 제공받고 JDBC try/catch/finally 구조로 만들어진 컨텍스트 내에서 작업을 수행하다, 제공받은 전략 오브젝트는 PreparedStatement 생성이 필요한 시점에 호출해서 사용한다.
        
        다음은 클라이언트에 해당하는 부분을 살펴보자
        
        ```java
        public void deleteAll() throws SQLException {
        	StatementStrategy st = new DeleteAllStatement();
        	jdbcContextWithStatementStrategy(st);
        }
        ```
        
        컨텍스트를 별도의 메소드로 분리했으니 deleteAll() 메소드가 클라이언트가 된다. deleteAll()은 전략 오브젝트(DeleteAllStatement)를 만들고 컨텍스트를 호출하는 책임(jdbcContextWithStatementStrategy())을 지고 있다. 
        
        클라이언트와 컨텍스트는 클래스를 분리하진 않았지만, 의존관계와 책임으로 볼 때 이상적인 클라이언트/컨텍스트 관계를 갖고 있다.
        

1. JDBC 전략 패턴의 최적화
    
    1)전략 클래스의 추가 정보
    
    add() 메소드에도 적용한다. 먼저 add() 메소드에서 변하는 부분인 PrepareStatement를 만드는 코드를 Addstatement 클래스로 옮겨 담는다.
    
    ```java
    
    public class AddStatement implements StatementStrategy {
    	public PreparedStatment makePreparedStatement(Connection c) {
    		throws SQLException {
    			PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
    			ps.setString(1, user.getId());
    			ps.setString(2, user.getName());
    			ps.setString(3, user.getPassword());
    
    			return ps;
    			}
    	}
    ```
    
    이렇게 클래스를 분리하고 나면 컴파일 에러가 난다. deleteAll()과는 달리 add()에서는 PreparedStatement를 만들 때 user라는 부가적인 정보가 필요하기 때문이다.
    
    클라이언트로부터 User 타입 오브젝트를 받을 수 있도록 AddStatment의 생성자를 통해 제공받게 만든다.
    
    ```java
    package springbook.user.dao;
    ...
    public class AddStatement implements StatementStrategy {
    	User user;
    	
    	public AddStatement(User user) {
    		this.user = user;
    	}
    	public PreparedStatement makePreparedStatement(Connection c) {
    	...
    	ps.setString(1, user.getId());
    	ps.setString(2, user.getName());
    	ps.setString(3, user.getPassword());
    }
    }
    ```
    
    ```java
    public void add(User user) throws SQLException {
    	StatementStrategy st = new AddStatement(user);
    	jdbcContextWithStatementStrategy(st);
    }
    ```
    
    이렇게 해서 deleteAll() 과  add() 두 군데에서 모두 PreparedStatement를 실행하는 JDBC try/catch/finally 컨텍스트를 공유해서 사용할 수 있다.
    
    2) 전략 클라이언트의 동거
    
    개선할 부분 :
    
    1) DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야한다
    
    → 기존 UserDao 때보다 클래스파일의 개수가 늘어난다.
    
    2) DAO 메소드에서 전달할 부가적인 정보가 있는 경우, 이를오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 번거롭게 만들어야 한다
    
    → 이 오브젝트 시점은 컨텍스트가 전략 오브젝트를 호출할 때이므로 잠시라도 어딘가에 다시 저장해줄 수 밖에 없다.
    
    - 로컬 클래스
        
        클래스 파일이 많아지는 문제 해결> 
        
        StatementStrategy 전략 클래스를 매번 독립된 파일로 만들지 말고 UserDao 클래스 안에 내부 클래스로 정의한다
        
        ```java
        public void add(User user) throws SQLException {
        	class AddStatement implements StatementStrategy {
        		User user;
        
        		public AddStatement(User user) {
        			this.user = user;
        		}
        		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        			PreparedStatement ps = c.prepareStatement("insert into users(id, name, password)values(?,?,?)");
        			ps.setString(1, user.getId());	
        			ps.setString(2, user.getName());	
        			ps.setString(3, user.getPassword());	
        		} 		
        	}
        	StatementStretegy st = new AddStretegy(user);
        	jbdcContentWithStrategyStatement(st);
        }
        ```
        
        AddStatement 클래스를 로컬 클래스로서 add() 메소드 안에 집어넣은 것이다. 마치 로컬 변수를 선언하듯 선언하며, 로컬 클래스는 선언된 메소드 내에서만 사용할 수 있다.
        
        또한 내부 메소드는 자신이 정의된 메소드의 로컬 변수에 직접 접근할 수 있기 때문에, 생성자를 통해 오브젝트를 전달해줄 필요가 없다. 다만 내부 클래스에서 외부의 변수를 사용할 때는 외부변수는 반드시 final로 선언해줘야 한다.
        
        ```java
        public void add(final User user) throws SQLException {
        	class AddStatement implements StatementStrategy {
        		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        			PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) valuse(?,?,?)");
        			ps.setString(1, user.getId());
        			ps.setString(2, user.getName());
        			ps.setString(3, user.getPassword());
        
        			return ps;
        			)
        		}
        		StatementStrategy st = new AddStatement();
        		jdbcContextWithStatementStrategy(st);
        	}
        ```
        
        로컬클래스로 만들어주면 메소드마다 추가해야 했던 클래스파일을 하나 줄일 수 있고, 내부 클래스의 특징을 이용해 로컬 변수를 바로 가져다 사용할 수 있다.
        
    
    - 익명 내부 클래스
        
        익명 내부 클래스는 선언과 동시에 오브젝트를 생성한다. 이름이 없기 때문에 클래스 자신의 타입을 가질 수 없고, 구현한 인터페이스 타입의 변수에만 저장할 수 있다.
        
        ```java
        StatementStrategy st = new StatementStrategy() { //익명 내부 클래스는 구현하는 인터페이스를 생성자처럼 이용해서 오브젝트로 만든다.
        	public PreparedStatement makePreparedStatement(Connection c) throws SPQException {
        		PreparedStatement ps = c.prepareStatement("id, name, password") values(?,?,?)");
        		ps.setString(1, user.getId());
        		ps.setString(2, user.getName());
        		ps.setString(3, user.getPassword());
        
        		return ps;
        	}
        }
        ```
        
        만들어둔 익명 내부 클래스의 오브젝트는 딱 한 번만 사용할 테니 굳이 변수에 담아두지 말고 jdbcContextWithStrategy() 메소드의 파라미터에서 바로 생성하는 편이 낫다. 이렇게 정리하면 더욱 간결해진다.
        
        ```java
        public void add(final User user) throws SQLException {
        	jdbcContextWithStatementStrategy { new StatementStrategy()
        	
        			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        				PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
        				ps.setString(1, user.getId());
        				ps.setString(2, user.getName());
        				ps.setString(3, user.getPasswrod());
        		
        				return ps;
        			}
        		}
        	);
        }
        ```
        
2. 컨텍스트와 DI
    
    1) JDBC Context의 분리
    
    jdbcContextWithStatementStrategy()를 다른 UserDao 클래스 밖으로 독립시켜서 모든 DAO가 사용할 수 있게 한다 
    
    - 클래스 분리
        
        JdbcContext에 UserDao에 있던 컨텍스크 메소드를 workWithStatementStrategy() 라는 이름으로 옮긴다. 이 경우 JdbcContext가 DataSource에 의존하고 있으므로 DataSource 타입 빈을 DI 받을 수 있게 해줘야 한다.
        
        ```java
        package springbook.user.dao;
        ...
        public class JdbcContext {
        	private DataSource;
        	
        	public void setDataSource(DataSource dataSource) {
        		this.dataSource = dataSource;
        	}
        	
        	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        		Connection c = null;
        		PreparedStatement ps = null;
        	}
        	
        	try {
        		c = this.dataSource.getConnection;
        		ps = stmt.makePreparedStatment(c);
        		ps.executeUpdate();
        	} catch (SQLException e) {
        			throw e;
        	} finally {
        			if (ps != null) { try {ps.close();} catch (SQLExcletion e) {}}
        			if (c != null) { try {c.close();} catch (SQLExcletion e) {}}
        	}
        }
        ```
        
        userDao가 분리된 JdbcContext를 DI 받아서 사용할 수 있게 만든다.
        
        ```java
        public class UserDao {
        	...
        	private JdbcContext(JdbcContext jdbcContext) {
        		this.jdbcContext = jdbcContext;
        	}
        	public void add(final User user) throws SQLException {
        		this.jdbcContext.workWithStateStrategy(
        			new StatementStrategy() {...}
        			);
        	}
        	public void deleteAll() throws SQLException {
        		this.jdbcContext.workWithStatementStrategy (
        			new StatementStrategy() {...}
        		);
        	}
        }
        ```
        
    - 빈 의존관계 변경
        
        새롭게 작성된 오브젝트 간의 의존관계를 살펴보고 이를 스프링 설정에 적용해본다.
        
        UserDao는 이제 JdbcContext에 의존한다. 그런데 JdbcContext는 인터페이스인 DataSource와 달리 구체 클래스다. 스프링의 DI는 기본적으로 인터페이스를 사이에 두고 의존 클래스를 바꿔서 사용하도록 하는게 목적이다. 
        
        하지만 이 경우는 인터페이스를 구현하도록 만들지 않았고,  UserDao와 JdbcContext는 인터페이스를 사이에 두지 않고 DI를 적용하는 특별한 구조가 된다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f72b6313-c847-41f1-936a-bdcf0fc40280/e995934b-24e9-4877-8ba8-6229a93261b5/Untitled.png)
        
        스프링의 빈 설정은 클래스 레벨이 아니라 런타임 시에 만들어지는 오브젝트 레벨의 의존관계에 따라 정의된다. 기존에는 userDao 빈이 dataSource 빈을 직접 의존했지만 이제는 jdbcContext 빈이 그 사이에 끼게 된다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f72b6313-c847-41f1-936a-bdcf0fc40280/5d8f5181-ae5d-4f04-a837-03ef1c7884fd/Untitled.png)
        
        그림의 빈 의존관계에 따라 test-applicationContext.xml 파일을  아래와 같이 수정한다.
        
        ```java
        <xml version = "1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schma/beans"
        	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        	xsi:schemaLocation="http://www.springframe.org.schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        	
        	<bean id = "userDao" class="springbook.user.doa.UserDao">
        		<property name = "dataSource" ref = "dataSource" />
        		<properety name = "jdbcContext" ref="jdbcContext" />
        	</bean>
        	<bean id="jdbcContext" class="springbook.user.dao.JdbcContext">
        		<property name = "dataSource" ref="dataSource" />
        	</bean>
        
        	<bean id ="dataSource" class = "org.springframework.jdbc.datasource.SimpleDriverDataSource">
        		...
        	</bean>
        </bean>
        ```
        
        이제 JdbcContext를 UserDao로부터 완전히 분리하고DI를 통해 연결될 수 있도록 설정을 마쳤다.
        
    
    2) JDBC Context의 특별한 DI
    
    - 스프링 빈으로 DI
        
        인터페이스를 사용하지 않고 DI를 적용하는 것
        
        → 반드시 인터페이스를 사용할 필요는 없다. 
        
        인터페이스를 사용하지 않았다면 온전한 DI라고 볼 수 없지만. 스프링의 DI는 넓게 보자면 객체의 생성과 관계 설정에 대한 제어권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC라는 개념을 포괄한다.
        
        인터페이스를 사용해서 클래스를 자유롭게 변경할 수 있게 하지는 않았지만 JdbcContext를 UserDao와 DI 구조로 만들어야 할 이유를 생각해보자
        
        1: JdbcContext가 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이 되기 때문이다.
        
        2: JdbcContext가 DI를 통해 다른 빈에 의존하고 있기 때문이다. DI를 위해서는 주입되는 오브젝트와 주입받는 오브젝트 양쪽 모두 스프링 빈으로 등록돼야 한다.
        
        강력한 결합을 허용하고 싶다면, 인터페이스를 두지 않고, 싱글톤으로 만드는 것과 JdbcContext에 대한 DI 필요성을 위해 스프링의 빈으로 등록해서 UserDao에 DI 되도록 만들어도 좋다. 다만 가장 마지막 단계에서 고려해볼 사항이여야 한다.
        
    - 코드를 이용하는 수동 DI
        
        JdbcContext를 스프링의 빈으로 등록해서 UserDao에 DI하는 대신, UserDao 내부에서 직접 DI를 적용하는 방법이다.
        
        싱글톤으로 만드는 대신, DAO 마다 하나의 JdbcContext 오브젝트를 갖고 있게 하는 것이다. 
        
        JdbcContext를 스프링 빈으로 등록하지 않았으므로 다른 누군가가 JdbcContext의 생성과 초기화를 책임져야 한다. JdbcContext의 제어권은 UserDao가 갖는 것이 적당한데, 자신이 사용할 오브젝트를 직접 만들고 초기화하는 전통적인 방법을 사용하는 것이다.
        
        JdbcContext가 다른 빈을 인터페이스를 통해 간접적으로 의존하고 있으므로, 의존오브젝트를 DI를 통해 제공박기 위해서라도 자신도 빈으로 등록해야 한다. 
        
        이 경우에는, JdbcContext에 대한 제어권을 갖고 생성과 관리를 담당하는 UserDao에게 DI까지 맡긴다.
        
        JdbcContext를 UserDao에서 코드를 통해 DI 해주는 방식으로 변경하는 것이다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f72b6313-c847-41f1-936a-bdcf0fc40280/cf5e4108-337b-4606-9e38-886aa2b474c2/Untitled.png)
        
        스프링의 설정 파일에 userDao와 dataSource 두 개만 빈으로 정의한다. 그리고 userDao 빈에 DataSource 타입 프로퍼티를 지정해서 dataSource 빈을 주입받도록한다.
        
        ```java
        <beans>
        	<bean id="userDao" class="springbook.user.dao.UserDao">
        		<property name="dataSource" ref="dataSource" />
        	</bean>
        
        	<bean id="dataSource" class ="org.springframework.jdbcsource.SimpleDriverDataSource">
        	...
        	</bean>
        </beans>
        ```
        
        설정파일만 보자면 UserDao가 직접 DataSource를 의존하고 있는 것 같지만, 내부적으로는 JdbcContext를 통해 간접적으로 DataSource를 사용하고 있을 뿐이다.
        
        ```java
        public class UserDao {
        	...
        	private JdbcContext jdbcContext;
        	
        	public void setDataSource(DataSource dataSource) {
        		this.jdbcContext = new JdbcContext();
        		this.jdbcContext.setDataSource(dataSource);
        		this.dataSource = dataSource;
        	}
        }
        ```
        
        UserDao의 메소드에서 는 JdbcContext가 외부에서 빈으로 만들어져 주입된 것인지, 내부에서 직접 만들고 초기화 된 것인지 구분할 필요가 없으며, 필요에 따라 JdbcContext를 사용하면 된다.
        
        이 방법의 장점은 굳이 인터페이스를 두지 않아도 될 만큼 긴밀한 관계를 갖는 DAO 클래스와 JdbcContext를 어색하게 따로 빈으로 분리하지 않고 내부에서 직접 만들어 사용하면서도 다른 오브젝트에 대한 DI를 적용할 수 있다는 점이다.
        
    
3. 템플릿과 콜백
    
    전략패턴의 컨텍스트를 템플릿이라 부르고, 익명 내부 클래스로 만들어지는 오브젝트를 콜백이라고 부른다.
    
    템플릿 : 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀을 가리킨다
    
    콜백 : 실행되는 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 말한다.
    
    1) 템플릿/콜백의 동작원리
    
    여러 개의 메소드를 가진 일반적인 언터페이스를 사용할 수 있는 전략 패턴의 전략과 달리 템플릿/콜백 패턴의 콜백은 보통 단일 메소드 인터페이스를 사용한다.하나의 템플릿에서 여러 가지 종류의 전략을 사용해야 한다면 하나 이상의 콜백 오브젝트를 사용할 수도 있다. 콜백은 일반적으로 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어진다고 보면 된다. 
    
    콜백 인터페이스의 메소드에는 보통 파라미터가 있다. 이 파라미터는 템플릿의 작업 프름 중에 만들어지는 컨텍스트 정보를 전달받을 때 사용된다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f72b6313-c847-41f1-936a-bdcf0fc40280/4725e3c4-ced8-4d3a-92a9-cab8d115edfb/Untitled.png)
    
    DI 방식의 전략 패턴 구조라고 생각하고 보면 간단하다. 클라이언트가 템플릿 메소드를 호출하면서 콜백 오브젝트를 전달하는 것은 메소드 레벨에서 일어나는 DI다. 템플릿이 사용할 콜백 인터페이스를 구현한 오브젝트를 메소드를 통해 주입해주는 DI 작업이 클라이언트가 템플릿의 기능을 호출하는 것과 동시에 일어난다. 
    
    일반적인 DI라면 템플릿에 인스턴스 변수를 만들어두고 사용할 의존 오브젝트를 수정자 메소드로 받아서 사용하지만, 템플릿/콜백 방식에서는 매번 메소드 단위로 사용할 오브젝트를 새롭게 전달받는다. 콜백 오브젝트가 내부 클래스로서 자신을 생성한 클라이언트 메소드 내의 정보를 직접 참조한다는 것도 템플릿/콜백의 고유한 특징이다. 클라이언트와 콜백이 강하게 결합된다는 면에서도 일반적인 DI와 조금 다르다. 템플릿/콜백 방식은 전략 패턴과 DI의 장점을 익명 내부 클래스 사용 전략과 결합한 독특한 활용법이라고 이해할 수 있다.
    
    2) 편리한 콜백의 재활용
    
    템플릿/콜백 방식에는 DAO 메소드에서 매번 익명 내부 클래스를 사용하기 때문에 상대적으로 코드를 작성하고 읽기가 조금 불편하다는 단점이 있다.
    
    - 콜백의 분리와 재활용
        
        JDBC의 try/catch/finally에 적용했던 방법을 UserDao의 메소드에도 적용해서 복잡한 익명 내부 클래스의 사용을 최소화한다.
        
        SQL 문장만 파라미터로 받아서 바꿀 수 있게 하고 메소드 내용 전체를 분리해 별도의 메소드로 만들어본다.
        
        ```java
        public void deleteAll() throws SQLException {
        	executeSql("delete from users");
        }
        private void executeSql(final String query) throws SQLException {
        	this.jdbcContext.workWithStatementStrategy(
        		new StatementStrategy() {
        			public PreparedStatement makePreparedStatement(Connnection c throws SQLException) {
        				return c.prepareStatement(query);
        			} 
        		}
        	);
        }
        ```
        
        바뀌지 않는 모든 부분을 빼내서 executeSql() 메소드로 만들었다. 바뀌는 부분인 SQL 문장만 파라미터로 받아서 사용하게 만들었다. SQL을 담은 파라미터를 final로 선언해서 익명 내부 클래스인 콜백 안에서 직접 사용할 수 있게 하는 것만 주의하면 된다.
        
    - 콜백과 템플릿의 결합
        
        이렇게 재사용가능한 콜백을 담고 있는 메소드라면 DAO가 공유할 수 있는 템플릿 클래스 안으로 옮겨도 된다. 
        
        ```java
        public class JdbcContext {
        	...
        	public void executeSql(final String query) throws SQLException {
        		workWithStatementStrategy(
        			new StatementStrategy() {
        				public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        					return c.prepareStatement(query);
        				} 
        			}
        		);
        	}
        ```
        
        executeSql() 메소드가 JdbcContext로 이동했으니 UserDao의 메소드에서도 jdbcContext를 통해 executeSql() 메소드를 호출하도록 수정해야 한다.
        
        ```java
        public void deleteAll() throws SQLException {
        	this.jdbcContext,executeSql("delete from users");
        }
        ```
        
        결국 JdbcContext 안에 클라이언트와 템플릿, 콜백이 모두 함께 공존하면서 동작하는 구조가 됐다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f72b6313-c847-41f1-936a-bdcf0fc40280/b21b1f3f-2251-4914-a05c-a5b0a2e718da/Untitled.png)
        
        일반적으로 성격이 다른 코드들은 가능한 한 분리하는 편이 낫지만, 이 경우는 하나의 목적을 위해 서로 긴밀하게 연관되어 동작하는 응집력이 강한 코드들이기 때문에 한 군데 모여있는게 유리하다. 구체적인 구현과 내부의 전략 패턴, 코드에 의한 DI, 익명 내부 클래스 등의 기술은 최대한 감취두고, 외부에는 꼭 필요한 기능을 제공하는 단순한 메소드만 노출해주는 것이다.
        
    
    3) 템플릿/콜백의 응용
    
    - 테스트와 try/catch/finally
        
        파일 하나를 열어서 모든 라인의 숫자를 더해주는 코드에 대한 테스트를 작성한다.
        
        ```java
        public class CalcSumTest {
            @Test
            public void sumOfNumber() throws IOException {
                Calculator calculator = new Calculator();
                int sum = calculator.calcSum(getClass().
                        getResource("numbers.txt").getPath());
                assertThat(sum, is(10));
            }
        }
        ```
        
        테스트의 구현 코드를 작성한다.
        
        ```java
        public class Calculator {
            public Integer calcSum(String filePath) throws IOException {
                BufferedReader br = new BufferedReader(new FileReader(filePath));
                Integer sum = 0;
                String line = null;
                while ((line = br.readLine()) != null) {
                    sum += Integer.valueOf(line);
                }
                br.close();
                return sum;
        
            }
        }
        ```
        
        어떤 경우에서도 파일이 열렸으면 반드시 닫아주고, 예외상황이 발생하면 로그를 남기는 기능을 추가한다. 
        
        calcSum() 메서드에 try/catch/finally를 적용한다.
        
        ```java
        public class Calculator {
            public Integer calcSum(String filePath) throws IOException {
                BufferedReader br = null;
                try {
                    br = new BufferedReader(new FileReader(filePath));
                    Integer sum = 0;
                    String line = null;
                    while ((line = br.readLine()) != null) {
                        sum += Integer.valueOf(line);
                    }
                    return sum;
                } catch (IOException e) {
                    System.out.println(e.getMessage());
                    throw e;
                } finally {
                    if (br != null) {
                        try {
                            br.close();
                        } catch (IOException e) {
                            System.out.println(e.getMessage());
                        }
                    }
                }
            }
        }
        ```
        
    - 중복의 제거와 템플릿/콜백 설계
        
        템플릿이 파일을 열고 각 라인을 읽어올 수 있는 BufferedReader를 만들어서 콜백에게 전달해주고, 콜백이 각 라인을 읽어서 알아서 처리한 후 최종 결과만 템플릿에게 돌려준다. 이것을 인터페이스의 메소드로 표현한다.
        
        ```java
        public interface BufferedReaderCallback {
            Integer doSomethingWithReader(BufferedReader br) throws IOException;
        }
        ```
        
        템플릿 부분을 메소드로 분리한다. 템플릿에서는 BufferedReaderCallback 인터페이스 타입의 콜백 오브젝트를 받아서 적절한 시점에 실행해준다. 콜백이 돌려준 결과는 최종적으로 모든 처리를 마친 후에 다시 클라이언트에 돌려준다.
        
        ```java
        private Integer fileReadTemplate(String filePath, BufferedReaderCallback callback)
                    throws IOException {
                BufferedReader br = null;
                try {
                    br = new BufferedReader(new FileReader(filePath));
                    return callback.doSomethingWithReader(br);
                } catch (IOException e) {
                    System.out.println(e.getMessage());
                    throw e;
                } finally {
                    if (br != null) {
                        try {
                            br.close();
                        } catch (IOException e) {
                            System.out.println(e.getMessage());
                        }
                    }
                }
            }
        ```
        
        ```java
        
        public Integer calcSum(String filePath) throws IOException {
        	BufferedReaderCallback sumCallback = new BufferedReaderCallback() {
                @Override
                public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                    Integer sum = 0;
                    String line = null;
                    while ((line = br.readLine()) != null) {
                       sum += Integer.valueOf(line);
                     }
                     return sum;
                    }
                };
                return fileReadTemplate(filePath, sumCallback);
            }
        ```
        
        파일에 있는 숫자의 곱을 구하는 메소드를 이 템플릿/콜백을 이용해서 만든다.
        
        테스트에서 공통 파일을 다루기 때문에 @Before에서 처리한다.
        
        ```java
        public class CalcSumTest {
            Calculator calculator;
            String numFilePath;
            @Before
            public void setUp(){
                calculator = new Calculator();
                numFilePath = getClass().getResource("numbers.txt").getPath();
            }
            @Test
            public void multiplyOfNumbers() throws IOException {
                assertThat(this.calculator.calcMultiply(this.numFilePath),is(24));
            }
            @Test
            public void sumOfNumber() throws IOException {
                assertThat(this.calculator.calcSum(this.numFilePath), is(10));
            }
        }
        ```
        
        테스트를 성공시키는 코드를 만든다. 앞에서 만든 sumCallback과 거의 비슷하지만 각 라인의 숫자를 더하는 대신 곱하는 기능을 담은 콜백을 사용하도록 만든다.
        
        ```java
        public Object calcMultiply(String filePath) throws IOException {
             BufferedReaderCallback multiplyCallback = new BufferedReaderCallback() {
                 @Override
                 public Integer doSomethingWithReader(BufferedReader br) throws IOException {
                    Integer multiply = 1;
                    String line = null;
                    while ((line = br.readLine()) != null) {
                       multiply *= Integer.valueOf(line);
                    }
                   return multiply;
                 }
             };
        	   return fileReadTemplate(filePath, multiplyCallback);
        }
        ```
        
    - 템플릿/콜백의 재설계
        
        앞서 구현한 코드를 살펴보면, 덧셈, 곱셈의 콜백 부분이 상당히 유사함을 알 수 있다. 템플릿과 콜백을 찾아낼 때는, 변하는 코드의 경계를 찾고 그 경계를 사이에 두고 주고받는 일정한 정보가 있는지 확인한다. 여기서 바뀌는 코드는 실제로 네번째 줄 뿐이다
        
        ```java
        public interface LineCallback {
            Integer doSomethingWithLine(String line, Integer value) throws IOException;
        }
        ```
        
        새로 만든 LineCallback 인터페이스를 경계로 템플릿을 새로 만든다. while 루프 안에 콜백을 넣어서 콜백을 여러 번 반복적으로 호출한다.
        
        ```java
        private Integer lineReadTemplate(String filePath, LineCallback callback, Integer initValue)
                    throws IOException {
                BufferedReader br = null;
                try {
                    br = new BufferedReader(new FileReader(filePath));
                    Integer res = initValue;
                    String line = null;
                    while ((line = br.readLine()) != null) {
                        res = callback.doSomethingWithLine(line, res);
                    }
                    return res;
                } catch (IOException e) {
                    System.out.println(e.getMessage());
                    throw e;
                } finally {
                    if (br != null) {
                        try {
                            br.close();
                        } catch (IOException e) {
                            System.out.println(e.getMessage());
                        }
                    }
                }
            }
        ```
        
        이렇게 수정한 템플릿을 사용하는 코드를 만든다.
        
        ```java
        public Object calcMultiply(String filePath) throws IOException {
                LineCallback multiplyCallback = new LineCallback() {
                    @Override
                    public Integer doSomethingWithLine(String line, Integer value) throws IOException {
                        return value * Integer.valueOf(line);
                    }
                };
                return lineReadTemplate(filePath, multiplyCallback, 1);
        }
        
        	public Integer calcSum(String filePath) throws IOException {
        	        LineCallback sumCallback = new LineCallback() {
        	            @Override
        	            public Integer doSomethingWithLine(String line, Integer value) throws IOException {
        	                return value + Integer.valueOf(line);
        	            }
        	        };
        	        return lineReadTemplate(filePath, sumCallback, 0);
        	}
        	
        
        ```
        
        여타 로우레벳의 파일 처리 코드가 템플릿으로 분리되고 순수한 계산 로직만 남아있기 때문에 코드의 관심이 무엇인지 명확하게 보인다. Calculator 클래스와 메소드는 데이터를 가져와 계산한다는 핵심 기능에 충실한 코드만 갖고 있게 됐다.
        
    - 제네릭스를 이용한 인터페이스
        
        자바 언어에 타입 파라미터라는 개념을 도입한 제네릭스를 이용하면, 다양한 오브젝트 타입을 지원하는 인터페이스나 메소드를 정의할 수 있다. 
        
        먼저 콜백 인터페이스를 수정한다. 콜백 메소드의 리턴 앖과 파라미너 값의 타입을 제네릭 타입 파라미터 T로 선언한다.
        
        ```java
        public interface LineCallback<T> {
            T doSomethingWithLine(String line, T value) throws IOException;
        }
        ```
        
        템플릿인 lineReadTemplate() 메소드도 타입 파라미터를 사용해 제네릭 메소드로 만든다. 콜백의 타입 파라미터와 초기값인 initVal의 타입, 템플릿의 결과 값 타입을 모두 동일하게 선언해야 한다.
        
        ```java
        public class Calculator {
            private <T> T lineReadTemplate(String filePath, 
                                           LineCallback<T> callback, T initValue)
                    throws IOException {
                BufferedReader br = null;
                try {
                    br = new BufferedReader(new FileReader(filePath));
                    T res = initValue;
                    String line = null;
                    ...
            }
            ...
        }
        ```
        
        파일의 모든 라인의 내용을 하나의 문자열로 길게 연결하는 기능을 가진 메소드를 추가한다. 콜백을 정의할 때 사용할 타입을 지정하면 된다.
        
        ```java
         public String concatenate(String filePath) throws IOException {
                LineCallback<String> concatenateCallback = new LineCallback<String>() {
                    @Override
                    public String doSomethingWithLine(String line, String value) throws IOException {
                        return value + Integer.valueOf(line);
                    }
                };
                return lineReadTemplate(filePath, concatenateCallback, "");
            }
        ```
        
        concatenate()메서드에 대한 테스트를 만든다.
        
        ```java
        @Test
        public void concatenateStrings() throws IOException {
           assertThat(calculator.concatenate(this.numFilePath), is("1234"));
           }
        }
        ```
        

1. 스프링의 JdbcTemplate
    
    스프링이 제공하는 jdbc코드용 기본 템플릿은 JdbcTemplate이다. 앞에서 만들었던 JdbcContext와 유사하지만 훨씬 강력하고 편리한 기능을 제공해준다.
    
    JdbcContext를 JdbcTemplate로 변경한다.
    
    ```java
    public class UserDao {
    	...
    	private JdbcTempate jdbcTemplate;
    	
    	public void setDataSource(DataSource dataSource) {
    		this.jdbcTemplate = new JdbcTemplate(dataSource);
    
    		this.dataSource = dataSource;
    	}
    }
    ```
    

1) update()

JdbcTemplate의 콜백과 템플릿 메소드를 사용하도록 deleteAll() 메소드를 수정한다.

```java
 public void deleteAll() throws SQLException{
        String query = "delete from users";
        this.jdbcTemplate.update(new PreparedStatementCreator() {
            @Override
            public PreparedStatement createPreparedStatement(Connection con) 
                    throws SQLException {
                return con.prepareStatement(query);
            }
        });
    }
```

JdbcTemplate의 내장 콜백을 사용하는 메소드를 호출하도록 수정한다.

```java
public void deleteAll() throws SQLException{
        this.jdbcTemplate.update("delete from users");
}
```

JdbcTemplate는 add()메소드에 대한 편리한 메소드도 제공한다. PreparedStatement를 만들 때 사용하는 SQL은 동일하며 파라미터는 순서대로 넣어주면 된다.

```java
this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
		user.getId(), user.getName(), user.getPassword)
```

2) queryForInt()

getCount() 메서드에 JdbcTemplate을 사용하도록 수정한다.

콜백을 두 개 등장시키는데,  첫 번쨰 PreparedStatementCreator 콜백은 템플릿으로부터 Connection을 받고 PreparedStatement를 돌려준다. 두 번째 ResultSetExtractor는 템플릿으로부터 ResultSet을 받고 거기서 추출한 결과를 돌려준다.

콜백을 만드느라 익명 내부클래스가 두 번이나 등장하는데, 원래 getCount() 메소드에 있던 코드 중에 변하는 부분만 콜백으로 만들어져서 제공된다고 생각한다. 콜백이 만들어낸 결과는 템플릿을 거쳐야만 클라이언트인 getCount() 메소드로 넘어오는 것이다. 

또한 ResultSetExtractor는 ResultSet에서 추출할 수 있는 값의 타입이 다양하기 때문에 제네릭스 타입 파라미터를 갖는다. 

JdbcTemplate는 이런 기능을 가진 콜백을 내장하고 있는 queryForInt() 라는 편리한 메소드를 제공한다. Integer 타입의 결과를 가져올 수 있는 SQL문장만 전달해주면 된다.

```java
 public int getCount() {
     return this.jdbcTemplate.query(new PreparedStatementCreator() {
         @Override
         public PreparedStatement createPreparedStatement(Connection con)
                 throws SQLException {
             return con.prepareStatement("select count(*) from users";);
         }
    }, new ResultSetExtractor<Integer>() {
    @Override
    public Integer extractData(ResultSet rs)
                    throws SQLException, DataAccessException {
                rs.next();
                return rs.getInt(1);
        }
     });
  }
```

queryForInt()를 사용하면 이중 콜백을 사용하는 제법 복잡해보이는 메소드를 아래와 같이 한 줄로 바꿀 수 있다.

```java
public int getCount() {
        return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

3) queryForObject()

get() 메소드에 JdbcTemplate를 적용한다.

첫 번째 파라미터는 PreparedStatement를 만들기 위한 SQL이고, 두 번쨰는 여기에 바인딩할 값들이다. 뒤에 다른 파라미터가 있으므로 가변인자 대신 Object 타입 배열을 사용해야 한다. 배열 초기화 블록을 사용해서 SQL의 ?에 바인딩할 id 값을 전달한다. 

quertForObject()는 SQL을 실행하면 한 개의 로우만 얻을 것이라고 기대한다. 그리고 ResultSet의 next()를 실행해서 첫 번째 로우로 이동시킨 후에 RowMapper 콜백을 호출한다. 이미 RowMapper가 호출되는 시점에서 ResultSet은 첫 번째 로우를 가리키고 있으므로 다시 rs.next()를 호출할 필요는 없다. RowMapper에서는 현재 ResultSet이 가리키고 있는 로우의 내용을 User오브젝트에 그대로 담아서 리턴해주기만 하면 된다. RowMapper가 리턴한 User 오브젝트는 queryForObject() 메소드의 리턴 값으로 get() 메소드에 전달된다.

quertForObject()를 이용할 때는 조회 결과가 없는 예외상황에 대해 특별히 해 줄 것은 없다. 이미 queryForObject()는 SQL을 실행해서 받은 로우의 개수가 하나가 아니라면 예외를 던지도록 만들어져 있다. 이때 던져지는 예외가 바로 EmptyResultDataAccessException이다.

```java
public User get(String id) {
        return this.jdbcTemplate.queryForObject("select * from users where id = ?";,
                new Object[]{id},
                new RowMapper<User>() {
                    @Override
                    public User mapRow(ResultSet rs, int rowNum)
                            throws SQLException {
                        User user = new User();
                        user.setId(rs.getString("id"));
                        user.setName(rs.getString("name"));
                        user.setPassword(rs.getString("password"));
                        return user;
                    }
                }
        );
  }
```

4) query()

- 기능 정의와 테스트 작성
    
    RowMapper를 좀 더 사용하여, 현재 등록되어 있는 모든 사용자 정보를 가져오는 getAll() 메소드를 추가한다.
    
    UserDaoTest 안에 픽스처로 준비해둔 user1, user2, user3을 차례로 추가하면서 getAll() 이 돌려주는 리스트의 크기와 리스트에 담긴 User 오브젝트의 내용을 픽스처와 비교한다. 이때 Id 순서대로 정렬된다는 점을 주의해야 한다. 그래서 user3은 가장 마지막에 추가되지만 getAll()의 결과에선 가장 첫 번째여야 한다. User의 값을 비교하는 코드가 반복되기 떄문에 별도의 메소드로 분리한다.
    
    ```java
     @Test
     public void getAll() {
            dao.deleteAll();
    
            dao.add(user1);
            List<User> listUsers1 = dao.getAll();
            assertThat(listUsers1.size(), is(1));
            checkSameUser(user1, listUsers1.get(0));
    
            dao.add(user2);
            List<User> listUsers2 = dao.getAll();
            assertThat(listUsers2.size(), is(2));
            checkSameUser(user1, listUsers2.get(0));
            checkSameUser(user2, listUsers2.get(1));
    
            dao.add(user3);
            List<User> listUsers3 = dao.getAll();
            assertThat(listUsers3.size(), is(3));
            checkSameUser(user3, listUsers3.get(0));
            checkSameUser(user1, listUsers3.get(1));
            checkSameUser(user2, listUsers3.get(2));
        }
    
        private void checkSameUser(User pUser1, User pUser2) {
            assertThat(pUser1.getId(), is(pUser2.getId()));
            assertThat(pUser1.getName(), is(pUser2.getName()));
            assertThat(pUser1.getPassword(), is(pUser2.getPassword()));
        }
    
    ```
    
- query 템플릿을 이용하는 getAll() 구현
    
    이 테스트를 성공시키는 getAll() 메소드를 JdbcTemplate의 query() 메소드를 사용하여 만들어본다.
    
    첫 번째 파라미터에는 실행할 SQL 쿼리를 넣는다. 바인딩할 파라미터가 있다면 두 번째 파라미터에 추가할 수도 있다. 파라미터가 없다면 생략할 수 있다. 마지막 파라미터는 RowMapper 콜백이다. quert() 템플릿은 SQL을 실행해서 얻은 ResultSet의 모든 로우를 열람하면서 RowMapper 콜백을 호출한다. SQL 쿼리를 실행해 DB에서 가져오는 로우의 개수만큼 호출될 것이다. RowMapper는 현재 로우의 내용을 User 타입 오브젝트는 템플릿이 미리 준비한 List<User> 컬렉션에 추가된다. 모든 로우에 대한 작업을 마치면 모든 로우에 대한 User오브젝트를 담고 있는List<User> 오브젝트가 리턴된다.
    
    ```java
    public List<User> getAll() {
            String query = "select * from users order by id";
            return this.jdbcTemplate.query(query,
                    new RowMapper<User>() {
                        @Override
                        public User mapRow(ResultSet rs, int rowNum)
                                throws SQLException {
                            User user = new User();
                            user.setId(rs.getString("id"));
                            user.setName(rs.getString("name"));
                            user.setPassword(rs.getString("password"));
                            return user;
                        
    		                }
    			        );
    }
    ```
    

- 테스트 보완
    
    의도적으로 예외적인 조건에 대해 먼저 테스트를 만들어보는 것도 좋다.
    
    getAll()의 쿼리를 실행했는데 아무런 데이터가 없는 경우, queryForObject() 처럼 예외를 던지지는 않는다. 대신 크기가 0인 List<T> 오브젝트를 돌려준다. getAll()은 query()가 돌려주는 결과를 그대로 리턴하도록 만든다. 테스트에는 검증 코드를 추가한다.
    
    ```java
    public void getAll() {
         dao.deleteAll();
    
         List<User> users0 = dao.getAll();
         assertThat(users0.size() ,is(0));
    		 ...
    		
    ```
    

5) 재사용 가능한 콜백의 분리

- DI를 위한 코드 정리
    
    필요 없어진 DataSource 인스턴스 변수는 제거하고 수정자 메소드는 남겨둔다.
    
    JdbcTemplate를 직접 스프링 빈으로 등록하는 방식을 사용하고 싶다면 setDataSource를 setJdbcTemplate로 바꿔주기만 하면 된다.
    
    ```java
    private JdbcTemplate jdbcTemplate;
    
    public void setDataSource(DataSource dataSource) {
    	this.jdbcTemplate = new JdbcTemplate(dataSource
    }
    ```
    

- 중복 제거
    
    UserDao용 RowMapper 콜백을 메소드에서 분리해 중복을 없애고  재사용되게 만들어야 한다. RowMapper 콜백은 하나만 만들어서 공유한다. userMapper라는 이름을 인스턴스 변수를 만들고 사용할 매핑 용 콜백 오브젝트를 초기화하도록 만든다. 
    
    ```java
    public class UserDao {
        private RowMapper<User> userMapper =
                new RowMapper<User>() {
                    @Override
                    public User mapRow(ResultSet rs, int rowNum)
                            throws SQLException {
                        User user = new User();
                        user.setId(rs.getString("id"));
                        user.setName(rs.getString("name"));
                        user.setPassword(rs.getString("password"));
                        return user;
                    }
          };
    ```
    
    인스턴스 변수에 저장해둔 userMapper 콜백 오브젝트는 get()과 getAll() 에서 사용하면 된다.
    
    ```java
    public User get(String id) {
            String query = "select * from users where id = ?";
            return this.jdbcTemplate.queryForObject(query,
                    new Object[]{id}, this.userMapper);
        }
    
        public List<User> getAll() {
            String query = "select * from users order by id";
            return this.jdbcTemplate.query(query, this.userMapper);
        }
    ```
    
- 템플릿/콜벡 패턴과 UserDao
    
    최종적으로 완성된 UserDao클래스이다. 템플릿/콜백 패턴과 DI를 이용해 예외 처리와 리소스 관리, 유연한 DataSource 활용 방법까지 제공하면서도 군더더기 하나 없는 깔끔하고 간결한 코드로 정리할 수 있게 됐다.
    
    UserDao에는 User 정보를 DB에 넣거나 가져오거나 조작하는 방법에 대한 핵심적인 로직만 담겨 있다. User라는 자바 오브젝트와 USER 테이블 사이에 어떻게 정보를 주고받을지, DB와 커뮤니케이션하기 위한 SQL 문장이 어떤 것인지에 대한 최적화된 코드를 갖고 있다. 만약 사용할 테이블과 필드 정보가 바뀌면 UserDao의 거의 모든 코드가 함께 바뀌므로 응집도가 높다고 볼 수 있다.
    
    반변 JDBC API를 사용하는 방식, 예외처리, 리소스의 반납, DB연결을 어떻게 가져올지에 관한 책임과 관심은 모두 JdbcTemplate에게 있다. 따라서 변경이 일어난다고 해도 UserDao코드에는 아무런 영향을 주지 않는다. 그런 면에서 책임이 다른 코드와는 다른 낮은 결합도를 유지하고 있다. 다만 JdbcTemplate이라는 템플릿 클래스를 직접 이용한다는 면에서 특정 템플릿/콜백 구현에 대한 강한 결합을 가지고 있다.
    
    ```java
    public class UserDao {
    	  public void setDataSource(DataSource dataSource) {
            this.jdbcTemplate = new JdbcTemplate(dataSource);
        }
    
    		 private JdbcTemplate jdbcTemplate;
    
    		 private RowMapper<User> userMapper =
                new RowMapper<User>() {
                    @Override
                    public User mapRow(ResultSet rs, int rowNum)
                            throws SQLException {
                        User user = new User();
                        user.setId(rs.getString("id"));
                        user.setName(rs.getString("name"));
                        user.setPassword(rs.getString("password"));
                        return user;
                }
    	    };
    
    			 public void add(final User user) {
    	        String id = user.getId();
    	        String name = user.getName();
    	        String password = user.getPassword();
    	        String query = "insert into users(id, name, password) value (?,?,?)";
    	        this.jdbcTemplate.update(query, id, name, password);
    		    }
    
    		    public User get(String id) {
    	        String query = "select * from users where id = ?";
    	        return this.jdbcTemplate.queryForObject(query,
                    new Object[]{id}, this.userMapper);
    		    }
        
    				public void deleteAll() {
    	        String query = "delete from users";
    	        this.jdbcTemplate.update(query);
    		    }
    
    		    public int getCount() {
    	        String query = "select count(*) from users";
    	        return this.jdbcTemplate.queryForObject(query, Integer.class);
    		    }
    
    				public List<User> getAll() {
    	        String query = "select * from users order by id";
    	        return this.jdbcTemplate.query(query, this.userMapper);
    		    }
    		}
    ```