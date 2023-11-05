# 1장 오브젝트와 의존 관계

## 🌿 DAO의 분리

### 🌱 관심사의 분리

객체지향의 세계에서는 오브젝트에 대한 설계와 이를 구현한 코드가 끊임없이 변한다. <br>
→ 그러므로 개발자는 미래의 변화에 대비하는 개발을 실천해야 한다.
<br>

객체지향이 초기에 더 많고 번거로운 작업을 요구하는 이유가 바로 변화에 효과적으로 대처할 수 있다는 기술적인 특징 때문이다!<br>

그렇다면 변화를 어떻게 대비할 것인가? <br>

> 변화의 폭을 최소한으로 줄여주어야 한다! <br>
> → 이것은 **분리와 확장을 고려한 설계를 기반**으로 대비할 수 있다!

모든 변경과 발전은 **한 번에 한 가지 관심사항**에 집중해서 일어난다.
그러므로 우리는 **한 가지 관심이 한 군데에 집중**되게 하도록 설계해야 한다.

> 즉, 관심이 같은 것 끼리는 모으고 관심이 다른 것은 따로 떨어져 있게 하는 것. <br> > **"관심사의 분리"**!!

<br>

관심사의 분리 in 객체지향?

1. 관심이 같은 것 끼리는 하나의 객체 or 친한 객체로 모이게
2. 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리함.

### 🌱 커넥션 만들기의 추출

UserDao를 수정하기

#### 중복 코드의 메소드 추출

현재 Connection을 가져오는 코드가 여러 메소드에서 중복되고 있음.

- getConnection() 메소드로 따로 추출하여 중복을 제거한다!
  - DB 접속 방식 혹은 Driver 클래스와 URL이 바뀌었을 때 능동적인 대처가 가능하다.

→ 즉 관심의 종류에 따라 코드를 구분해놓았기 때문에 한 가지 관심에 대한 변경이 일어날 경우, 그 관심이 집중되는 부분의 코드만 수정하면 된다!

<br>

위 작업에 대한 결과로 기능에 대한 영향 없이 코드 구조를 깔끔하게 변경하였다. <br>
이 과정을 우리는 **리팩토링** 이라고 한다.

> **리팩토링** <br>
> 기존의 코드를 외부의 동작 방식에는 변환 없이 내부 구조를 변경해서 재구성하는 작업 또는 기술을 말함.

### 🌱 DB 커넥션 만들기의 독립

#### 상속을 통한 확장

기존 UserDao 내부 소스코드를 공개하지 않고, 고객 스스로 DB 커넥션 생성 방식을 적용해가면서 UserDao를 사용하게 해보자. <br>
방법은 기존의 UserDao 코드를 한 단계 더 분리하면 된다.

1. getConnection()의 구현 코드를 제거하여 추상 메소드로 만든다.
2. 각각의 고객은 UserDao를 상속한 서브클래스를 생성하여 getConnection()을 원하는 방식대로 구현하여 사용한다.

<br>

- 다음과 같이 DB 커넥션 연결이라는 관심을 이번에는 상속을 통해 서브클래스로 분리해버리는 것이다!

<img width="800" src="./img/week1/inheritance-UserDao-expand.PNG"/>

<br>

<table>
    <tr>
        <th>UserDao의 관심</th>
        <th>UserDao의 서브클래스의 관심</th>
    </tr>
    <tr>
        <td>어떻게 데이터를 등록하고 가져올 것인가?</td>
        <td>DB 연결 기능에 대한 관심을 포함함.</td>
    </tr>
</table>

> **템플릿 메소드 패턴** <br>
> 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤, <br>
> 서브 클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법(패턴)!

<img width="800" src="./img/week1/factory-method-pattern.PNG"/>

<br>

NUserDao와 DUserDao에서 생성하는 Connection 오브젝트의 구현 클래스는 차이가 날 것!

- 각각 NUserDao는 → AConnection (implements Connection)
- DUserDao는 → BConnection 클래스를 구현해서 사용할 것임.

다만 UserDao 클래스는 Connection interface 에만 관심을 두고, Connection에 정의된 메소드를 사용함. **Connection 내부 동작 방식과는 상관 없이 필요한 기능을 인터페이스를 통해 사용**하는 것! <br>
반대로 NUserDao와 DUserDao의 관심 사항은 어떻게 Connection 오브젝트 생성과 연결 기능 제공이 되는지에 대해 관심을 두고 있음. <br>

### 🌱 상속 사용의 문제점

만약 이미 UserDao가 다른 목적을 위해 상속을 사용하고 있다면?

- 다른 목적으로 상속 적용이 어려움.

<br>

또한, 상속 관계는 생각보다 밀접함.

- 슈퍼클래스 내부 변경은 모든 서브 클래스 변경을 야기함.
  - 이걸 막기 위해 슈퍼 클래스의 변화를 막는 제약을 가하게 될 수도..

<br>

확장된 기능인 DB 커넥션 생성 코드를 다른 DAO 클래스에 적용할 수 없다는 것도 큰 단점임.

- UserDao외의 다른 DAO 클래스들이 계속 만들어진다면? DB 커넥션 생성 코드 (getConnection() 메서드)는 모든 Dao 클래스에서 중복되어 나타날 것!

## 🌿 DAO의 확장

### 🌱 클래스의 분리

관심사를 메소드가 아닌 아예 독립적인 클래스로 분리시킨다!

- 한 번만 SimpleConnectionMaker 오브젝트를 생성하여 저장해두고 계속 사용한다!

<img width="800" src="./img/week1/separated-class.PNG"/>

<br>

But, 다른 문제 발생!

- UserDao의 코드가 `SimpleConnectionMaker`라는 특정 클래스에 종속되어버리면서 connection 기능 확장장이 불가능해졌다!

<br>

우리가 해결해야 할 문제

- UserDao가 바뀔 수 있는 정보, 즉 DB 커넥션을 가져오는 클래스에 대해 너무 많이 알고 있음.
  - 그러므로 고객이 DB 커넥션을 자유롭게 확장하기 위해 다른 클래스를 구현하려면 어쩔 수 없이 UserDao 자체를 다시 수정해야 함.

### 🌱 인터페이스의 도입

추상화 작업을 해보자!

- 추상화: 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업.

  - 인터페이스로 추상화해놓은 최소한의 통로를 통해 접근? 오브젝트를 만들 때 사용할 클래스가 무엇인지 몰라도 됨!

    <img width="800" src="./img/week1/interface.PNG"/>

### 🌱 관계설정 책임의 분리

하지만 여전히 UserDao에는 어떤 Connection 구현 크래스를 사용할지 결정하는 코드가 남아있다.

- UserDao의 클라이언트에서 UserDao를 사용하기 전에, 먼저 UserDao가 어떤 ConnectionMaker의 구현 클래스를 사용할지를 결정하도록 만들어주자!

<br>

UserDao 오브젝트가 사용할 구현 클래스를 **메소드 파라미터** 혹은 **생성자 파라미터**를 이용하여 전달해준다!

> 객체 지향의 **다형성 특징을 사용**한다!

클래스 사이의 관계 X, 오브젝트 사이의 다이내믹한 관계가 만들어지는 것.

- 클래스 사이의 관계: 코드에 다른 클래스 이름이 나타나면서 만들어짐.
- 오브젝트 사이의 관계
  - 코드에서는 특정 클래스를 전혀 알지 못함.
  - 다만 해당 클래스가 구현한 인터페이스를 사용하여 해당 클래스를 인터페이스 타입으로 받아 사용하는 것!

#### 책임을 클라이언트로 떠넘기자!

다음과 같이 userDao client가 구체적인 클래스를 생성하여 UserDao의 생성자로 넣어주게 되는 것이다!

- UserDao와 ConnectionMaker 구현 클래스와의 런타임 오브젝트 의존 관계 설정 책임을 담당하게 되는 것.
  <img width="800" src="./img/week1/userDao-client.PNG"/>

<br>

UserDaoTest가 추가된 구조는 다음과 같다.

<img width="800" src="./img/week1/structure-with-UserDaoTest.PNG"/>

### 🌱 원칙과 패턴

#### 개방 폐쇄 원칙

클래스와 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다!

#### 높은 응집도와 낮은 결합도

- 응집도가 높다는 것?
  - 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있따는 것.
  - 하나의 공통 관심사는 한 클래스에!
- 낮은 결합도?
  - 책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태를 유지해야 한다.
  - 느슨한 결합
    - 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공. <br>
      나머지는 서로 독립적이고 알 필요도 없게 만들어 주는 것.

#### 전략 패턴

자신의 기능 맥락(context)에서, 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴.

## 🌿 제어의 역전(IoC)

### 🌱 오브젝트 팩토리

현재 UserDaoTest는 UserDao의 기능 동작 테스트 뿐 만 아니라, ConnectionMaker의 구현 클래스를 결정하는 역할까지 떠맡음.

한 가지씩 역할을 맡아야 하기 때문에,

- UserDao와 ConnectionMaker 구현 클래스의 오브젝트를 만드는 것
- 두 개의 오브젝트가 연결되어 사용될 수 있또록 관계를 맺어주는 것

이 기능들을 분리해주자!

#### 팩토리

팩토리 클래스의 역할: 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 것.

- 여기서는 UserDaoTest 내부의 UserDao, ConnectionMaker 관련 생성 작업을 맡는다!

```java

/**
 * 다음과 같이 팩토리의 메소드는 UserDao 타입의 오브젝트를
 * 어떻게 만들고, 어떻게 준비시킬지를 결정함.
**/

public class DaoFactory{
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

#### 설계도로서의 팩토리

- 어떤 오브젝트가 어떤 오브젝트를 사용하는지 보임.

<img width="800" src="./img/week1/structure-with-ObjectFactory.PNG"/>

<br>

역할과, 관계를 분석해보자

<table>
    <tr>
        <th>UserDao</th>
        <th>ConnectionMaker</th>
        <th>DaoFactory</th>
    </tr>
    <tr>
        <td>어떻게 데이터를 등록하고 가져올 것인가?</td>
        <td>어떻게 DB에 연결할 것인가?</td>
        <td>오브젝트들을 어떻게 구성하고, 그 관계를 정의할 것인가?</td>
    </tr>
</table>

### 🌱 오브젝트 팩토리의 활용

- UserDao가 아닌 다른 DAO의 생성 기능을 DaoFactory에 넣게 된다면?
  - 다음과 같이 ConnectionMaker 구현 클래스를 선정하고 생성하는 코드가 중복되게 됨.

```java
public class DaoFactory{
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker(); // 중복
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }

    public AccountDao accountDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();// 중복
        AccountDao accountDao = new AccountDao(connectionMaker);
        return accountDao;
    }

    public MessageDao messageDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();// 중복
        MessageDao messageDao = new MessageDao(connectionMaker);
        return messageDao;
    }
}
```

#### 중복 제거하기

```java
public class DaoFactory{
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    public AccountDao accountDao() {
        return new AccountDao(connectionMaker());
    }

    public MessageDao messageDao() {
        return new MessageDao(connectionMaker());
    }

    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```

### 🌱 제어권의 이전을 통한 제어관계 역전

제어의 역전? 프로그램 제어 흐름 구조가 뒤바뀌는 것!

- 오브젝트는 자신이 사용할 오브젝트를 스스로 선택하지 않음. + 당연히 생성하지도 않음.
- 자신이 어떻게 만들어지고, 어떻게 사용되는지도 알지 못함.
  - 모든 제어 권한을 다른 대상에게 위임하기 때문.

→ main() 과 같은 엔트리 포인트를 제외하고, 모든 오브젝트는 이렇게 **위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어짐**.

## 🌿 스프링의 IoC

### 🌱 오브젝트 팩토리를 이용한 스프링의 IoC

#### 애플리케이션 컨텍스트와 설정정보

### 🌱 애플리케이션 컨텍스트의 동작방식

### 🌱 스프링 IoC의 용어 정리

## 🌿 싱글톤 레지스트리와 오브젝트 스코프

### 🌱 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

### 🌱 싱글톤과 오브젝트의 상태

### 🌱 스프링 빈의 스코프

## 🌿 의존관계 주입(DI)

### 🌱 제어의 역전과 의존관계 주입

### 🌱 런타임 의존관계 설정

### 🌱 의존관계 검색과 주입

### 🌱 의존관계 주입의 응용

### 🌱 메소드를 이용한 의존관계 주입

## 🌿 XML을 이용한 설정

### 🌱 XML 설정

### 🌱 XML을 이용하는 애플리케이션 컨텍스트

### 🌱 DataSource 인터페이스로 변환

### 🌱 프로퍼티 값의 주입
