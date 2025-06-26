# 아이템 16.public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라

## 🔑 요약

> public 클래스는 절대 필드를 직접 공개하지 말고
→ **private 필드 + 접근자 메서드(getter/setter)**를 통해 캡슐화하는 것이 좋다
> 

---

## ❌ 잘못된 예시: 필드를 직접 노출한 클래스

```java
class Point {
    public double x;
    public double y;
}

```

### 🔍 문제점

- 내부 표현을 바꿀 수 없음 (API 고정)
    - 외부에서 `x`, `y` 에 직접 접근이 가능해서 getter/setter든 다른 방법이든 이 필드를 바꾸면, 이 클래스를 사용하는 모든 코드가 영향을 받기 때문에 public 클래스는 내부 표현을 바꾸기 어려움
- 불변 보장 불가능
    - 외부에서 직접 값을 바꿀 수 있기 때문에
- 멀티스레드 환경에서 안전하지 않음
    - 여러 스레드가 한 번에 접근하면 값이 꼬일 수 있기 때문에
- 외부에서 필드 직접 수정 가능 → 캡슐화 위반

## ✅ 개선된 예시: getter/setter 메서드를 활용한 캡슐화

```jsx
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}

```

### ✅ 장점

- 내부 구현 변경 가능
    - 변경하고 싶다면 메소드를 만들어서 변경하면 된
- 유효성 검사 삽입 가능
- 멀티스레드 환경 대응 가능

### 🤷🏻‍♀️ 멀티스레드 환경에 대응이 가능하다는 게 무슨 말인가

**✅ public 클래스에서 필드를 직접 public으로 공개하면**

👉 외부에서 값을 **직접 바꿔버리기 때문에**

👉 내부에서 **동기화, 유효성 검사, 로깅, 접근 제한 등 아무 제어도 못 한다.**

**✅ getter/setter를 사용하면**

👉 값을 읽거나 쓸 때 반드시 **메서드를 거치게 되고**

👉 그 안에서 **동기화, 검사, 로직 추가**가 가능하기 때문에

👉 **멀티스레드 환경에서도 안정성을 확보할 수 있다.**

## ⚠️ 예외적 허용: 불변 클래스에서의 public 필드 노출

```jsx
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}

```

### 💡 조건

- 필드는 `final`이며 **불변성 보장**
- 생성자에서 유효성 검사를 수행
- 외부에서 변경 불가

## 📌 핵심 요약

| **상황** | **필드 노출 가능 여부** | **설명** |
| --- | --- | --- |
| `public 클래스` | ❌ 불가 | 반드시 getter/setter 사용 |
| `package-private / private 클래스` | ✅ 가능 | 내부 전용이므로 유연하게 사용 가능 |
| `불변 객체(public final class)` | ⚠️ 제한적으로 허용 | 단, 유효성 검사 필수 |

---

## 🧠 결론

- public 클래스에서는 캡슐화를 철저히 지켜야 함
- 직접 필드 노출은 API 유연성을 심각하게 제한함
- **getter/setter**를 통해 정보 은닉 및 유지보수성 확보