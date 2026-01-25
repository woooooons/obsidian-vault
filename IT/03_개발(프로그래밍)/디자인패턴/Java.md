---
title: 디자인패턴 java
date: 2026-01-25
tags:
  - it
  - 개발
  - 디자인패턴
---
# 1. 생성(Creational) 패턴

## 1-1. Singleton

**의도**: 인스턴스를 하나만 유지하며 전역에서 공유.  
**Java 관점**: private 생성자 + static 인스턴스로 구현. Thread-safe 보장 필요.

```java
public class Config {
    private static volatile Config instance;
    private String value;
    
    private Config() {
        this.value = "default";
    }
    
    public static Config getInstance() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) {
                    instance = new Config();
                }
            }
        }
        return instance;
    }
    
    // Enum 방식 (Joshua Bloch 권장)
    public enum ConfigEnum {
        INSTANCE;
        private String value = "default";
        
        public String getValue() { return value; }
        public void setValue(String value) { this.value = value; }
    }
}
```

## 1-2. Factory Method

**의도**: 객체 생성을 하위 타입에 위임.  
**Java 관점**: 인터페이스 + 팩토리 메서드 또는 추상 클래스 활용.

```java
public interface Notifier {
    void send(String message);
}

public class EmailNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

public class SMSNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

public class NotifierFactory {
    public static Notifier createNotifier(String type) {
        switch (type.toLowerCase()) {
            case "email":
                return new EmailNotifier();
            case "sms":
                return new SMSNotifier();
            default:
                throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}
```

## 1-3. Builder

**의도**: 복잡한 객체를 단계적으로 구성.  
**Java 관점**: 내부 static Builder 클래스로 메서드 체이닝 구현.

```java
public class Server {
    private final String host;
    private final int port;
    private final boolean tls;
    
    private Server(Builder builder) {
        this.host = builder.host;
        this.port = builder.port;
        this.tls = builder.tls;
    }
    
    public static class Builder {
        private String host = "localhost";
        private int port = 8080;
        private boolean tls = false;
        
        public Builder setHost(String host) {
            this.host = host;
            return this;
        }
        
        public Builder setPort(int port) {
            this.port = port;
            return this;
        }
        
        public Builder enableTLS() {
            this.tls = true;
            return this;
        }
        
        public Server build() {
            return new Server(this);
        }
    }
    
    // 사용 예시
    // Server server = new Server.Builder()
    //     .setHost("example.com")
    //     .setPort(443)
    //     .enableTLS()
    //     .build();
}
```

## 1-4. Abstract Factory

**의도**: 관련된 객체 군을 생성하는 인터페이스 제공.  
**Java 관점**: 여러 제품군을 생성하는 팩토리 추상화.

```java
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}
```

---

# 2. 구조(Structural) 패턴

## 2-1. Adapter

**의도**: 서로 다른 인터페이스를 맞춰줌.  
**Java 관점**: 클래스/객체 어댑터 두 방식 가능. 인터페이스 구현으로 래핑.

```java
public interface LegacyPrinter {
    String print(String text);
}

public class ModernPrinter {
    public String printFormatted(String text) {
        return "[FMT] " + text;
    }
}

public class PrinterAdapter implements LegacyPrinter {
    private ModernPrinter modernPrinter;
    
    public PrinterAdapter(ModernPrinter modernPrinter) {
        this.modernPrinter = modernPrinter;
    }
    
    @Override
    public String print(String text) {
        return modernPrinter.printFormatted(text);
    }
}
```

## 2-2. Decorator

**의도**: 기존 객체에 기능을 동적으로 추가.  
**Java 관점**: 추상 데코레이터 클래스로 래핑 체인 구성. Java I/O가 대표적.

```java
public interface Component {
    String operate();
}

public class ConcreteComponent implements Component {
    @Override
    public String operate() {
        return "base";
    }
}

public abstract class Decorator implements Component {
    protected Component component;
    
    public Decorator(Component component) {
        this.component = component;
    }
    
    @Override
    public String operate() {
        return component.operate();
    }
}

public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Component component) {
        super(component);
    }
    
    @Override
    public String operate() {
        return "decorated(" + super.operate() + ")";
    }
}

// 사용 예시
// Component c = new ConcreteDecorator(new ConcreteComponent());
```

## 2-3. Facade

**의도**: 복잡한 서브시스템을 단순 API로 제공.  
**Java 관점**: 단일 클래스가 여러 서브시스템 조율.

```java
public class AuthService {
    public void authenticate() {
        System.out.println("Authenticating...");
    }
}

public class DatabaseService {
    public void query() {
        System.out.println("Querying database...");
    }
}

public class LoggingService {
    public void log(String message) {
        System.out.println("Log: " + message);
    }
}

public class SystemFacade {
    private AuthService auth;
    private DatabaseService db;
    private LoggingService logger;
    
    public SystemFacade() {
        this.auth = new AuthService();
        this.db = new DatabaseService();
        this.logger = new LoggingService();
    }
    
    public void processRequest() {
        logger.log("Processing request");
        auth.authenticate();
        db.query();
        logger.log("Request completed");
    }
}
```

## 2-4. Proxy

**의도**: 객체에 대한 접근을 제어하는 대리자 제공.  
**Java 관점**: 동일 인터페이스 구현, 실제 객체 호출 전/후 처리.

```java
public interface Service {
    void execute();
}

public class RealService implements Service {
    @Override
    public void execute() {
        System.out.println("Executing real service");
    }
}

public class ServiceProxy implements Service {
    private RealService realService;
    private boolean hasAccess;
    
    public ServiceProxy(boolean hasAccess) {
        this.hasAccess = hasAccess;
    }
    
    @Override
    public void execute() {
        if (hasAccess) {
            if (realService == null) {
                realService = new RealService();
            }
            realService.execute();
        } else {
            System.out.println("Access denied");
        }
    }
}
```

---

# 3. 행위(Behavioral) 패턴

## 3-1. Strategy

**의도**: 알고리즘을 교체 가능하게 추상화.  
**Java 관점**: 인터페이스 기반 전략 객체를 Context에 주입.

```java
public interface Strategy {
    int execute(int a, int b);
}

public class AddStrategy implements Strategy {
    @Override
    public int execute(int a, int b) {
        return a + b;
    }
}

public class MultiplyStrategy implements Strategy {
    @Override
    public int execute(int a, int b) {
        return a * b;
    }
}

public class Context {
    private Strategy strategy;
    
    public Context(Strategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }
    
    public int executeStrategy(int a, int b) {
        return strategy.execute(a, b);
    }
}
```

## 3-2. Observer

**의도**: 상태 변경 시 여러 구독자에게 알림.  
**Java 관점**: Subject가 Observer 목록 관리, 변경 시 notify 호출.

```java
public interface Observer {
    void update(String message);
}

public class ConcreteObserver implements Observer {
    private String name;
    
    public ConcreteObserver(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}

public class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}
```

## 3-3. Command

**의도**: 요청을 객체로 캡슐화해 호출자와 실행자를 분리.  
**Java 관점**: Command 인터페이스, Invoker가 명령 큐 관리.

```java
public interface Command {
    void execute();
    void undo();
}

public class PrintCommand implements Command {
    private String message;
    
    public PrintCommand(String message) {
        this.message = message;
    }
    
    @Override
    public void execute() {
        System.out.println(message);
    }
    
    @Override
    public void undo() {
        System.out.println("Undo: " + message);
    }
}

public class Invoker {
    private List<Command> commandQueue = new ArrayList<>();
    
    public void addCommand(Command command) {
        commandQueue.add(command);
    }
    
    public void executeAll() {
        for (Command command : commandQueue) {
            command.execute();
        }
        commandQueue.clear();
    }
}
```

## 3-4. State

**의도**: 객체의 상태 변화에 따라 행동을 변경.  
**Java 관점**: State 인터페이스, Context가 현재 상태 객체 보유.

```java
public interface State {
    String handle();
}

public class OnState implements State {
    @Override
    public String handle() {
        return "Device is ON";
    }
}

public class OffState implements State {
    @Override
    public String handle() {
        return "Device is OFF";
    }
}

public class Device {
    private State state;
    
    public Device() {
        this.state = new OffState();
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    public String operate() {
        return state.handle();
    }
}
```

## 3-5. Template Method

**의도**: 알고리즘 골격을 정의하고 일부 단계를 서브클래스에 위임.  
**Java 관점**: 추상 클래스에서 템플릿 메서드 정의, 하위 클래스가 구현.

```java
public abstract class DataProcessor {
    // 템플릿 메서드
    public final void process() {
        readData();
        processData();
        writeData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    
    protected void writeData() {
        System.out.println("Writing data...");
    }
}

public class CSVProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV data");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
}

public class JSONProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading JSON data");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing JSON data");
    }
}
```

## 3-6. Iterator

**의도**: 컬렉션 내부 구조를 노출하지 않고 순회.  
**Java 관점**: Java Collections Framework가 기본 제공. 커스텀 구현 가능.

```java
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

public class CustomCollection<T> implements Iterable<T> {
    private List<T> items = new ArrayList<>();
    
    public void add(T item) {
        items.add(item);
    }
    
    @Override
    public Iterator<T> iterator() {
        return new Iterator<T>() {
            private int index = 0;
            
            @Override
            public boolean hasNext() {
                return index < items.size();
            }
            
            @Override
            public T next() {
                return items.get(index++);
            }
        };
    }
}
```

---

# 4. Java 개발자가 실무에서 가장 많이 쓰는 패턴

1. **Singleton**: Spring Bean 컨테이너(기본 스코프), 로깅, 설정 관리
    
2. **Factory/Abstract Factory**: 객체 생성 로직 캡슐화, DI 컨테이너 기반
    
3. **Builder**: 복잡한 DTO, 설정 객체 생성 (Lombok @Builder 활용)
    
4. **Strategy**: 결제 방식, 정렬 알고리즘, 비즈니스 로직 교체
    
5. **Template Method**: Spring의 JdbcTemplate, RestTemplate 등
    
6. **Decorator**: Java I/O (BufferedReader, InputStreamReader), Spring AOP
    
7. **Proxy**: Spring AOP, JPA Lazy Loading, 원격 프록시(RMI)
    
8. **Observer**: Event Listener, Spring ApplicationEvent
    
9. **Adapter**: 레거시 시스템 통합, 외부 API 래핑
    
10. **Facade**: 복잡한 서브시스템을 단순 서비스 인터페이스로 제공
    

---

# 5. Spring Framework와의 연계

- **Singleton**: Spring Container의 기본 빈 스코프
- **Factory**: BeanFactory, ApplicationContext
- **Proxy**: AOP, 트랜잭션 관리(@Transactional)
- **Template Method**: JdbcTemplate, RestTemplate, TransactionTemplate
- **Dependency Injection**: 전체적으로 Factory + Strategy 패턴 조합

---

# 6. 필요 시 확장 가능한 정리 구조

- 각 패턴별 독립 패키지 구성 (creational, structural, behavioral)
- Spring Boot 프로젝트와 통합 예제
- JPA, Hibernate와 패턴 연계 (Proxy, Decorator)
- Microservices 아키텍처에서의 패턴 활용 (API Gateway = Facade, Circuit Breaker = Proxy)
- 테스트 코드에서 Mock 객체 = Proxy/Decorator 패턴

---

## 참고 자료

신뢰 가능한 참고 자료 (일반적 설명 수준, 코드 구현은 본문 기반)

- _Design Patterns: Elements of Reusable Object-Oriented Software_ (GoF)
- _Refactoring Guru: Design Patterns_ ([https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns))
- _Head First Design Patterns_ (O'Reilly)
- _Spring Framework Documentation_ ([https://spring.io/guides](https://spring.io/guides))