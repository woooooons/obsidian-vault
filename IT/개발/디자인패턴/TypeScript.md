---
title: 디자인패턴 TypeScript
date: 2025-12-03
tags:
  - it
  - 개발
  - 디자인패턴
---
# 1. 생성(Creational) 패턴

## 1-1. Singleton

**의도**: 전역에서 하나의 인스턴스만 사용.  
**TS 특성**: `private constructor`로 제어.

```ts
class Config {
  private static instance: Config;
  private constructor(public value: string) {}

  static getInstance() {
    if (!Config.instance) {
      Config.instance = new Config("default");
    }
    return Config.instance;
  }
}

const c1 = Config.getInstance();
const c2 = Config.getInstance();
```

---

## 1-2. Factory Method

**의도**: 객체 생성 로직을 캡슐화.  
**TS 특성**: interface + class 조합.

```ts
interface Notifier {
  send(msg: string): void;
}

class Email implements Notifier {
  send(msg: string) { }
}

class SMS implements Notifier {
  send(msg: string) { }
}

function createNotifier(type: string): Notifier {
  switch (type) {
    case "email": return new Email();
    case "sms": return new SMS();
    default: throw new Error("invalid type");
  }
}
```

---

## 1-3. Builder

**의도**: 복잡한 객체를 단계적으로 생성.  
**TS 특성**: 체이닝 패턴이 자연스럽게 사용됨.

```ts
class Server {
  constructor(
    public host?: string,
    public port?: number,
    public tls?: boolean
  ) {}
}

class ServerBuilder {
  private server = new Server();

  setHost(h: string) {
    this.server.host = h;
    return this;
  }

  setPort(p: number) {
    this.server.port = p;
    return this;
  }

  enableTLS() {
    this.server.tls = true;
    return this;
  }

  build() {
    return this.server;
  }
}
```

---

# 2. 구조(Structural) 패턴

## 2-1. Adapter

**의도**: 기존 인터페이스를 새 인터페이스에 맞게 변환.

```ts
interface IPrinter {
  print(str: string): string;
}

class ModernPrinter {
  printFormatted(str: string) {
    return `[FMT] ${str}`;
  }
}

class PrinterAdapter implements IPrinter {
  constructor(private printer: ModernPrinter) {}

  print(str: string) {
    return this.printer.printFormatted(str);
  }
}
```

---

## 2-2. Decorator

**의도**: 객체 기능을 계층적으로 확장.  
**TS 특성**: class wrapping 또는 함수 데코레이터 활용 가능.

```ts
interface Component {
  operate(): string;
}

class Base implements Component {
  operate() { return "base"; }
}

class Decorator implements Component {
  constructor(private comp: Component) {}

  operate() {
    return `decorated(${this.comp.operate()})`;
  }
}
```

---

## 2-3. Facade

**의도**: 복잡한 시스템을 단순 API로 제공.

```ts
class AuthService {
  check() { }
}

class DBService {
  query() { }
}

class AppFacade {
  constructor(
    private auth = new AuthService(),
    private db = new DBService()
  ) {}

  process() {
    this.auth.check();
    this.db.query();
  }
}
```

---

# 3. 행위(Behavioral) 패턴

## 3-1. Strategy

**의도**: 알고리즘을 교체할 수 있도록 추상화.  
**TS 특성**: 클래스 또는 함수 전략 모두 적용 가능.

```ts
interface Strategy {
  calc(a: number, b: number): number;
}

class Add implements Strategy { calc(a, b) { return a + b; } }
class Multiply implements Strategy { calc(a, b) { return a * b; } }

class Context {
  constructor(private strategy: Strategy) {}

  setStrategy(s: Strategy) { this.strategy = s; }

  execute(a: number, b: number) {
    return this.strategy.calc(a, b);
  }
}
```

---

## 3-2. Observer

**의도**: 상태 변화 시 구독자에게 자동 알림.

```ts
interface Observer {
  update(msg: string): void;
}

class Subject {
  private observers: Observer[] = [];

  subscribe(o: Observer) {
    this.observers.push(o);
  }

  notify(msg: string) {
    this.observers.forEach(o => o.update(msg));
  }
}
```

---

## 3-3. Command

**의도**: 요청을 객체로 캡슐화해 실행을 분리.

```ts
interface Command {
  execute(): void;
}

class PrintCommand implements Command {
  constructor(private msg: string) {}
  execute() { }
}

class Invoker {
  private queue: Command[] = [];

  add(cmd: Command) {
    this.queue.push(cmd);
  }

  run() {
    for (const cmd of this.queue) cmd.execute();
  }
}
```

---

## 3-4. State

**의도**: 상태에 따라 행동을 변경.

```ts
interface State {
  handle(): string;
}

class OnState implements State {
  handle() { return "ON"; }
}

class OffState implements State {
  handle() { return "OFF"; }
}

class Device {
  constructor(private state: State) {}

  setState(s: State) { this.state = s; }

  run() {
    return this.state.handle();
  }
}
```

---

# 4. TS 개발자가 실무에서 자주 쓰는 패턴

1. **Strategy 패턴**
    
    - validation 방식 교체, 인증 방식 교체, 검색 알고리즘 교체 등
        
2. **Factory 패턴**
    
    - DI 컨테이너 구성, 환경별 객체 생성
        
3. **Decorator 패턴**
    
    - NestJS / class-validator 데코레이터 시스템과 자연스럽게 연결
        
4. **Facade 패턴**
    
    - 서비스 계층 통합, 도메인 단위 API 설계
        
5. **Observer 패턴**
    
    - RXJS, 이벤트 기반 구조, 소켓 기반 시스템
        
6. **Adapter 패턴**
    
    - 외부 라이브러리/SDK 인터페이스 통일
        

---

# 5. 확장 가이드

원하면 아래 내용도 추가로 정리해줄 수 있다.

- NestJS/Express 기반 실전 패턴 설계
    
- TypeScript에서 Clean Architecture + 디자인 패턴 조합
    
- 선택 패턴별 unittest 예시
    
- 프런트엔드(React/Vue)용 디자인 패턴 버전
    

---

## 참고 자료
참고 가능한 신뢰 자료(일반적 설명 수준이며 코드 구현은 본문 기반으로 참고 가능)
- [https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)
    
- [https://www.typescriptlang.org/docs/handbook/intro.html](https://www.typescriptlang.org/docs/handbook/intro.html)
    
- [https://github.com/torokmark/design_patterns_in_typescript](https://github.com/torokmark/design_patterns_in_typescript)