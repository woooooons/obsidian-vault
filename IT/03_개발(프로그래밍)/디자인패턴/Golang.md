---
title: 디자인패턴 golang
date: 2025-12-03
tags:
  - it
  - 개발
  - 디자인패턴
---
# 1. 생성( Creational ) 패턴

## 1-1. Singleton

**의도**: 인스턴스를 하나만 유지하며 전역에서 공유.  
**Go 관점**: 전역 변수 + `sync.Once`로 안전하게 초기화.

```go
package singleton

import "sync"

var (
    instance *Config
    once     sync.Once
)

type Config struct {
    Value string
}

func GetInstance() *Config {
    once.Do(func() {
        instance = &Config{Value: "default"}
    })
    return instance
}
```

## 1-2. Factory Method

**의도**: 객체 생성을 하위 타입에 위임.  
**Go 관점**: 인터페이스 + 생성용 함수.

```go
package factory

type Notifier interface {
    Send(msg string)
}

type Email struct{}
func (e Email) Send(msg string) {}

type SMS struct{}
func (s SMS) Send(msg string) {}

func NewNotifier(kind string) Notifier {
    switch kind {
    case "email": return Email{}
    case "sms": return SMS{}
    default: return nil
    }
}
```

## 1-3. Builder

**의도**: 복잡한 객체를 단계적으로 구성.  
**Go 관점**: 필드 체이닝 또는 구조체 조립용 별도 Builder 객체.

```go
type Server struct {
    Host string
    Port int
    TLS  bool
}

type Builder struct {
    server Server
}

func (b *Builder) SetHost(h string) *Builder {
    b.server.Host = h
    return b
}

func (b *Builder) SetPort(p int) *Builder {
    b.server.Port = p
    return b
}

func (b *Builder) EnableTLS() *Builder {
    b.server.TLS = true
    return b
}

func (b *Builder) Build() Server {
    return b.server
}
```

---

# 2. 구조( Structural ) 패턴

## 2-1. Adapter

**의도**: 서로 다른 인터페이스를 맞춰줌.  
**Go 관점**: 타입 래핑으로 인터페이스 충족.

```go
type LegacyPrinter interface {
    Print(s string) string
}

type ModernPrinter struct{}
func (m ModernPrinter) PrintFormatted(s string) string { return "[FMT]" + s }

type Adapter struct {
    printer ModernPrinter
}

func (a Adapter) Print(s string) string {
    return a.printer.PrintFormatted(s)
}
```

## 2-2. Decorator

**의도**: 기존 객체에 기능을 동적으로 추가.  
**Go 관점**: 인터페이스 기반으로 래핑.

```go
type Component interface {
    Operate() string
}

type Base struct{}
func (b Base) Operate() string { return "base" }

type Decorator struct {
    inner Component
}
func (d Decorator) Operate() string {
    return "decorated(" + d.inner.Operate() + ")"
}
```

## 2-3. Facade

**의도**: 복잡한 서브시스템을 단순 API로 제공.  
**Go 관점**: 단일 struct가 여러 모듈을 내부에서 호출.

```go
type Auth struct{}
func (a Auth) Check() {}

type DB struct{}
func (d DB) Query() {}

type Facade struct {
    auth Auth
    db   DB
}

func (f Facade) Process() {
    f.auth.Check()
    f.db.Query()
}
```

---

# 3. 행위( Behavioral ) 패턴

## 3-1. Strategy

**의도**: 알고리즘을 교체 가능하게 추상화.  
**Go 관점**: 함수 타입 또는 인터페이스로 전략 교체.

```go
type Strategy interface {
    Do(a, b int) int
}

type Add struct{}
func (Add) Do(a, b int) int { return a + b }

type Multiply struct{}
func (Multiply) Do(a, b int) int { return a * b }

type Context struct {
    S Strategy
}

func (c Context) Execute(a, b int) int {
    return c.S.Do(a, b)
}
```

## 3-2. Observer

**의도**: 상태 변경 시 여러 구독자에게 알림.  
**Go 관점**: slice에 등록된 observer 인터페이스 순환 호출.

```go
type Observer interface {
    Update(msg string)
}

type Subject struct {
    observers []Observer
}

func (s *Subject) Attach(o Observer) {
    s.observers = append(s.observers, o)
}

func (s *Subject) Notify(msg string) {
    for _, o := range s.observers {
        o.Update(msg)
    }
}
```

## 3-3. Command

**의도**: 요청을 객체로 캡슐화해 호출자와 실행자를 분리.  
**Go 관점**: 인터페이스 기반 실행 커맨드.

```go
type Command interface {
    Execute()
}

type PrintCommand struct {
    Msg string
}
func (c PrintCommand) Execute() {}

type Invoker struct {
    queue []Command
}

func (i *Invoker) Add(c Command) {
    i.queue = append(i.queue, c)
}

func (i *Invoker) Run() {
    for _, cmd := range i.queue {
        cmd.Execute()
    }
}
```

## 3-4. State

**의도**: 객체의 상태 변화에 따라 행동을 변경.  
**Go 관점**: 상태를 인터페이스로, Context가 상태 객체를 교체.

```go
type State interface {
    Handle() string
}

type On struct{}
func (On) Handle() string { return "ON" }

type Off struct{}
func (Off) Handle() string { return "OFF" }

type Device struct {
    state State
}

func (d *Device) SetState(s State) {
    d.state = s
}

func (d Device) Operate() string {
    return d.state.Handle()
}
```

---

# 4. Go 개발자가 실무에서 가장 많이 쓰는 패턴

1. **Strategy**: 알고리즘 선택, 로깅/인증 방식 교체 등
    
2. **Factory**: 설정 기반 객체 생성
    
3. **Decorator**: HTTP middleware 구조와 자연스럽게 호환
    
4. **Facade**: 복잡한 모듈을 단일 서비스로 정리
    
5. **Command**: 작업 큐, 비동기 task 관리
    
6. **Singleton**: 설정, 커넥션 풀 관리
    
7. **Adapter**: 외부 라이브러리 인터페이스 맞추기
    

---

# 5. 필요 시 확장 가능한 정리 구조

- 생성 패턴만 따로 프로젝트 구조화
    
- 구조/행위 패턴 각각 예제 코드를 독립 패키지로 구성
    
- 실제 웹/백엔드 환경(Gin, Echo, gRPC, Clean Architecture 등)과 연계하여 패턴 적용 사례 확장 가능
    

---
## 참고 자료
참고 가능한 신뢰 자료(일반적 설명 수준이며 코드 구현은 본문 기반으로 참고 가능)
- _Go Patterns Repository_ ([https://github.com/tmrts/go-patterns](https://github.com/tmrts/go-patterns))
    
- _Refactoring Guru: Design Patterns_ ([https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns))
