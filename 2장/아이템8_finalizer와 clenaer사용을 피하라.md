# Effective Java

**Finalizer와 Cleaner는 사용하지 마라**

Finalizer 매서드는 객체가 가비지 컬렉션될 때 호출되는 메서드입니다.

그러나 **`finalize`** 메서드는 예측할 수 없고, 신뢰할 수 없으며, 성능에 부정적인 영향을 미칠 수 있기 때문에 사용이 권장되지 않습니다. Java 9부터는 **`finalize`** 메서드가 더 이상 권장되지 않으며, Java 18에서는 완전히 제거되었습니다.

```jsx
public class FinalizeTest {
    private static int counter = 0;
    private int id;

    public FinalizeTest(int id) {
        this.id = id;
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("Finalize called for object with id: " + id);
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new FinalizeTest(i);
        }

        // Suggest the JVM to run the garbage collector
        System.gc();

        // Wait for a while to give the garbage collector time to run
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

이 코드는 100개의 **`FinalizeTest`** 객체를 생성하고, 각 객체는 고유한 **`id`**를 가집니다. **`finalize`** 메서드는 객체가 가비지 컬렉션될 때 호출되며, 객체의 **`id`**를 출력합니다.

**`System.gc()`** 메서드를 호출하여 JVM에 가비지 컬렉션을 요청합니다. 그러나 이 요청이 즉시 처리된다는 보장은 없습니다. **`Thread.sleep(5000)`**은 가비지 컬렉션이 완료될 시간을 주기 위해 5초 동안 대기합니다.

이 코드를 실행하면, 객체가 가비지 컬렉션될 때 **`finalize`** 메서드가 호출되고, 각 객체의 **`id`**가 출력됩니다. 그러나 객체가 생성된 순서대로 **`finalize`** 메서드가 호출된다는 보장은 없습니다. 가비지 컬렉션의 순서는 JVM의 구현에 따라 다를 수 있습니다.

![image](https://github.com/user-attachments/assets/e814b9a7-4578-4c6b-a95c-478d4ec84880)

**`finalize` 메서드를 사용하지 말아야 하는 이유는 여러 가지가 있습니다. 다음은 그 주요 이유들입니다:**

1. **예측 불가능한 실행 시점**:
    - **`finalize`** 메서드는 객체가 가비지 컬렉션될 때 호출되지만, 정확히 언제 호출될지는 알 수 없습니다. 이는 프로그램의 동작을 예측하기 어렵게 만듭니다.
2. **성능 문제**:
    - **`finalize`** 메서드를 사용하는 객체는 가비지 컬렉션 과정에서 추가적인 오버헤드를 발생시킵니다. 이는 성능 저하를 초래할 수 있습니다.
3. **자원 회수 지연**:
    - **`finalize`** 메서드가 호출될 때까지 자원이 해제되지 않을 수 있습니다. 이는 파일 핸들, 네트워크 소켓 등과 같은 중요한 자원의 회수를 지연시킬 수 있습니다.
4. **불확실한 실행 보장**:
    - JVM이 종료될 때 **`finalize`** 메서드가 호출되지 않을 수 있습니다. 이는 중요한 정리 작업이 수행되지 않을 위험을 초래합니다.
5. **복잡성 증가**:
    - **`finalize`** 메서드를 올바르게 구현하는 것은 어렵고, 잘못 구현하면 메모리 누수나 다른 심각한 문제를 일으킬 수 있습니다.
6. **대체 기술의 존재**:
    - **`java.lang.ref.Cleaner`**와 같은 대체 기술이 존재합니다. **`Cleaner`**는 더 명확하고 예측 가능한 자원 해제를 제공합니다.

```jsx
package effective;

import java.lang.ref.Cleaner;

public class CleanerTest {
    private static final Cleaner cleaner = Cleaner.create();

    static class State implements Runnable {
        private int id;

        State(int id) {
            this.id = id;
        }

        @Override
        public void run() {
            System.out.println("Cleaner called for object with id: " + id);
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public CleanerTest(int id) {
        this.state = new State(id);
        this.cleanable = cleaner.register(this, state);
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new CleanerTest(i);
        }

        // Suggest the JVM to run the garbage collector
        System.gc();

        // Wait for a while to give the garbage collector time to run
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
![image](https://github.com/user-attachments/assets/6778671b-e73f-41e1-a5f1-7baddfda9508)


**`Cleaner`**는 **`finalize`**보다 더 예측 가능하고 안전한 자원 해제 방법을 제공하지만, 여전히 몇 가지 이유로 사용을 권장하지 않는 경우가 있습니다. 다음은 **`Cleaner`**를 사용하지 말아야 하는 이유들입니다:

1. **명시적 자원 해제 권장**:
    - 자원 해제는 명시적으로 수행하는 것이 가장 좋습니다. 예를 들어, **`try-with-resources`** 구문을 사용하면 자원을 명확하고 예측 가능하게 해제할 수 있습니다. 이는 코드의 가독성과 유지보수성을 높입니다.
2. **지연된 자원 해제**:
    - **`Cleaner`**는 가비지 컬렉션이 발생할 때 자원을 해제합니다. 이는 자원 해제가 지연될 수 있음을 의미합니다. 중요한 자원(예: 파일 핸들, 데이터베이스 연결 등)의 경우, 즉시 해제되지 않으면 시스템 자원 고갈 등의 문제가 발생할 수 있습니다.
3. **성능 오버헤드**:
    - **`Cleaner`**를 사용하면 추가적인 성능 오버헤드가 발생할 수 있습니다. 이는 특히 자주 생성되고 해제되는 객체의 경우 성능 저하를 초래할 수 있습니다.
4. **예측 불가능한 타이밍**:
    - **`Cleaner`**는 가비지 컬렉션이 발생할 때 실행되므로, 정확히 언제 자원이 해제될지 예측하기 어렵습니다. 이는 프로그램의 동작을 예측하기 어렵게 만들 수 있습니다.

**명시적 자원해제**

- **try-finally 예제**

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TryFinallyExample {
    public static void main(String[] args) {
        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader("example.txt"));
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

- **try-with-resources 예제**

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TryWithResourcesExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("example.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**실제 코드 찾아보기**

```java
 @Transactional(readOnly = true)
    public Map<String, Object> getSealImg(Map<String, Object> parameter) {
        Assert.notNull(parameter, AssertMessageUtils.getNotNull("parameter"));
        Map<String, Object> resultMap = new HashMap<String, Object>();

        List<Map<String, Object>> resultList = jobOrderMapper.getSealImg(parameter);

        if(CollectionUtils.isEmpty(resultList)) {
            resultMap.put("resultMsg", "불러올 이미지가 없습니다.");
        }
        else {
            for(Map<String, Object> result : resultList) {
                try {
                    String str = "";
                    Clob clob = (Clob)MapUtils.getObject(result, "IMG_FILE");
                    String buffer = new String();
                    BufferedReader br = new BufferedReader(clob.getCharacterStream());
                    while((str = br.readLine()) != null) {
                        buffer += str;
                    }
                    Object obj = buffer;
                    result.put("IMG_FILE", obj);
                }
                catch(Exception e) {
                    e.printStackTrace();
                }
            }
            resultMap.put("list", resultList);
        }

        return resultMap;
```

**개선된 코드**

```java
public Map<String, Object> getSealImg(Map<String, Object> parameter) {
    Assert.notNull(parameter, AssertMessageUtils.getNotNull("parameter"));
    Map<String, Object> resultMap = new HashMap<>();

    List<Map<String, Object>> resultList = jobOrderMapper.getSealImg(parameter);

    if (CollectionUtils.isEmpty(resultList)) {
        resultMap.put("resultMsg", "불러올 이미지가 없습니다.");
    } else {
        for (Map<String, Object> result : resultList) {
            try {
                Clob clob = (Clob) MapUtils.getObject(result, "IMG_FILE");
                StringBuilder buffer = new StringBuilder();
                try (BufferedReader br = new BufferedReader(clob.getCharacterStream())) {
                    String str;
                    while ((str = br.readLine()) != null) {
                        buffer.append(str);
                    }
                }
                result.put("IMG_FILE", buffer.toString());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        resultMap.put("list", resultList);
    }

    return resultMap;
}
```

**Q) Bean 등록사용하는 객체들은 자원해제 관리를 별도로 할 필요가 없을까?**

Spring 프레임워크에서 **`@Service`** 어노테이션이 붙은 클래스는 일반적으로 애플리케이션 컨텍스트에 의해 관리되는 빈(Bean)으로 등록됩니다. 이러한 빈은 Spring 컨테이너가 생성하고 관리하며, 애플리케이션의 생명 주기 동안 자동으로 관리됩니다. **`@Autowired`** 어노테이션을 사용하여 이러한 빈을 주입받을 수 있습니다.

Spring 컨테이너가 관리하는 빈은 일반적으로 자원 회수를 명시적으로 할 필요가 없습니다. 이는 Spring 컨테이너가 빈의 생명 주기를 관리하고, 애플리케이션 종료 시점에 필요한 자원 해제를 자동으로 처리하기 때문입니다. 그러나 특정 상황에서는 자원 회수를 명시적으로 처리해야 할 수도 있습니다.

### **Spring 컨테이너가 관리하는 빈의 자원 회수**

1. **Spring 컨테이너가 관리하는 빈**:
    - **`@Service`**, **`@Component`**, **`@Repository`**, **`@Controller`** 등으로 정의된 빈은 Spring 컨테이너가 생성하고 관리합니다.
    - 이러한 빈은 일반적으로 애플리케이션의 시작 시점에 생성되고, 애플리케이션 종료 시점에 컨테이너에 의해 자동으로 소멸됩니다.
2. **자원 회수 필요성**:
    - 대부분의 경우, Spring 컨테이너가 관리하는 빈은 자원 회수를 명시적으로 처리할 필요가 없습니다.
    - 그러나 빈이 파일, 네트워크 연결, 데이터베이스 연결 등과 같은 외부 자원을 사용하는 경우, 이러한 자원을 명시적으로 해제해야 할 수 있습니다.
