# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높여라

## (아이템28) Parameterized Type(매개변수화 타입)은 invariant(불공변)이다?

제네릭을 사용하면서 ‘타입 파라미터(Type Parameter)와 매개변수화 타입(Parameterized Type)이라는 용어가 등장한다. 

#### Parameter(매개변수)

- 메서드에서 흔히 보는 `(String id)` 같은 값의 자리

```java
public User getUserById(String id)
public List<User> getUserListByName(String Name)
```

#### Type Parameter(타입 파라미터)와 Generic Type(제네릭 타입)

- 타입 파라미터 : 제네릭을 사용할 때 “타입”을 나타내기 위한 값의 자리 `(E, K, V)`
- 제네릭 타입 : “타입 파라미터”로 구성한 껍데기 `List<E>`, `Map<K,V>`

```java
// List.java
public interface List<E> extends Collection<E> {}

// Map.java
public interface Map<K, V> {}
```

#### Parameterized Type(매개변수화 타입)

- 제네릭을 사용 시 Type Parameter에 실제 타입을 채워 사용하는 형태 
`List<String>`, `Map<String, Integer>`

```java
// List<E>
List<String> list = new ArrayList<>();

// Map<K, V> 
Map<String, Integer> map = new HashMap<>();
```

#### 그럼 매개변수화 타입(Parameterized Type)은 불공변(Invariant)이다가 무슨 말일까?

Apple이 Fruit의 하위 클래스라고 하더라도, List<Apple>은 List<Fruit>로 대체해서 쓸 수 없다. 

즉, 제네릭 타입에서는 다형성이 적용되지 않는다. (부모 클래스에 자식 인스턴스를 할당하는…)

```java
class Fruit {}
class Apple extends Fruit {}

Apple apple = new Apple();
Fruit fruit = apple;         // 부모 클래스에 자식 인스턴스를 할당? OK

List<Apple> apples = new ArrayList<>();
List<Fruit> fruits = apples; // 컴파일 오류!
```

#### 왜 안될까? 타입 안정성을 보장하기 위해서 ⇒ 런타임 오류를 방지하기 위해서

제네릭에서 다형성이 적용된다면 아래와 같은 일이 생길 수 있다.

```java
class Fruit {}
class Apple extends Fruit {}
class Banana extends Fruit {}

List<Apple> apples = new ArrayList<>();
List<Fruit> fruits = apples;  // 이게 가능하다고 치고

// 만약 개발자의 실수로 fruits 안에 Banana를 넣었다?
fruits.add(new Banana());     // 실수로 넣어도 컴파일 가능

Apple a = apples.get(0);      // Banana를 Apple로 꺼내게 되면? 런타임 오류 발생!!!!
```

원칙적으로는 개발자가 잘 관리하면 문제는 없지만, 자바는 ‘사람은 실수한다’는 전제를 바탕으로 타입 안정성을 지키기 위해 불공변(invariant) 규칙을 강제한다.
<BR>

## 제네릭에서 다형성을 이용하려면 “한정적 와일드카드”를 이용

#### 예시1 : 내가 만든 Stack 클래스에 pushAll 메서드 추가

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
    
    // ** 신규 메서드 pushAll() **
    // 일련의 원소들 elements를 한번에 스택에 넣는 메서드
    public void pushAll(Iterable<E> elements) {
        for (E e : source)
            push(e);
    }
}
```

> CASE1 : Stack<Integer>, Iterable<Integer>
> 

문제 없음

```java
public class Main {

   public static void main(String[] args){
       Stack<Integer> stack = new Stack<>();
       Iterable<Integer> integerElements = List.of(1,2,3);
       stack.pushAll(integerElements);
   }
}
```

> CASE2 : Stack<Number>, Interable<Integer>
> 

위에서 설명했듯이 Parameterized Type(매개변수화 타입)은 invariant(불공변)이기 때문에 컴파일 오류 발생

```java
// public final class Integer extends Number ... {}
// Number는 Integer의 상위 클래스

public class Main {

   public static void main(String[] args){
       Stack<Number> stack = new Stack<>();
       Iterable<Integer> integerElements = List.of(1,2,3);
       stack.pushAll(integerElements); // 컴파일 오류!
   }
}
```

```java
error: incompatible types: Iterable<Integer> cannot be converted to Iterable<Number>
        stack.pushAll(integerElements);
                          ^
```

#### 해결책 : 한정적 와일드카드

자바는 이런 상황에 대처할 수 있는 “한정적 와일드카드 타입”이라는 특별한 Parameterized Type(매개변수화 타입)을 지원한다.

```java
public class Stack<E> {
    public void pushAll(Iterable<E> source) {
        for (E e : source) push(e);
    }
}
```

코드가 이렇게 짜져있다면, 제네릭은 불공변이므로 오직 Iterable<E>만 받을 수 있다.

CASE2에서 Integer가 Number의 하위 클래스이더라도 Iterable<Number>만 받을 수 있다. 

```java
public class Stack<E> {
    public void pushAll(Iterable<? extends E> source) { // 변경
        for (E e : source) push(e);
    }
}

public class Main {
   public static void main(String[] args){
       Stack<Number> stack = new Stack<>();
       Iterable<Integer> integerElements = List.of(1,2,3);
       stack.pushAll(integerElements); // 컴파일 OK!
   }
}
```

E의 하위 타입도 받을 수 있게 하려면 이렇게 변경하면 된다.

E의 하위 타입의 Iterable ⇒ `Iterable<? extends E>`

CASE2도 문제 없이 컴파일 된다.

#### 예시2 : 내가 만든 Stack 클래스에 popAll 메서드 추가

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
    public void pushAll(Iterable<E> elements);
    
    // ** 신규 메서드 popAll() **
    // Stack안의 모든 원소를 주어진 컬렉션으로 옮겨 담는 기능
    public void popAll(Collection<E> destination) {
        while (! this.isEmpty())
            destination.add(this.pop);
    }
}
```

> CASE3 : Stack<Number>, Collection<Number>
> 

문제 없음

```java
public class Main {

   public static void main(String[] args){
       Stack<Number> stack = new Stack<>();               // E : Number
       Iterable<Number> numberElements = List.of(1,2,3);
       stack.pushAll(numberElements);
       
       Collection<Number> destination = new ArrayList<>();// E : Number
       stack.popAll(destination);
   }
}
```

> CASE4 : Stack<Number>, Collection<Object>
> 

위에서 설명했듯이 Parameterized Type(매개변수화 타입)은 invariant(불공변)이기 때문에 컴파일 오류 발생

```java
public void popAll(Collection<E> destination) {
    while (! this.isEmpty())
        destination.add(this.pop);
}

public class Main {
   public static void main(String[] args){
       Stack<Number> stack = new Stack<>();               // E : Number
       Iterable<Number> numberElements = List.of(1,2,3);
       stack.pushAll(numberElements);

       Collection<Object> destination = new ArrayList<>(); // E : Object
       stack.popAll(destination); // 컴파일 오류! (Number != Object)
   }
}
```

```java
error: method popAll in class Stack<Number> cannot be applied to given types;
  required: Collection<Number>
  found: Collection<Object>
  reason: actual argument Collection<Object> cannot be converted to Collection<Number> by method invocation conversion
```

#### 해결책 : 한정적 와일드카드

```java
public void popAll(Collection<? super E> destination) {
    while (! this.isEmpty())
        destination.add(this.pop);
}
```

E의 상위 타입도 받을 수 있도록 코드를 수정한다.

E의 상위 타입의 Collection → `Collection<? super E>`

CASE4도 문제 없이 컴파일된다.

결론적으로, '한정적 와일드카드'를 이용하면 제네릭에도 다형성을 적용할 수 있게 되므로, API의 유연성을 높일 수 있다!!!
<BR>


## 한정적 와일드카드 형태 결정하기 : 펙스(PECS)

한정적 와일드카드는 2종류

```java
public void pushAll(Iterable<? extends E> source) 
public void popAll(Collection<? super E> destination)
```

펙스(PECS, producer-extends, consumer-super) 원칙에 따라 extends인지 super인지를 결정한다.

- 생산자 : 데이터를 꺼내서 읽는 역할
    - <? extends E>를 사용
    - E의 하위 타입에서 꺼내서 E처럼 사용 가능
- 소비자 : 데이터를 넣는 역할
    - <? super E>를 사용
    - E의 상위 타입에 E를 안전하게 넣을 수 있음
<BR>

## 비한정적 와일드카드  <?>

`<? extends T>`, `<? super T>` 같은 한정적 와일드카드 말고, 한정 없는 와일드카드 `<?>`를 사용할 수도 있다.

```java
public static void printList(List<?> list) { // 비한정적 와일드카드
    for (Object o : list) {
        System.out.println(o);
    }
}
```

파라미터 list는 어떤 타입의 List든 다 받을 수 있다. 

- List<String>, List<Integer>. List<Object>

이러한 비한정적 와일드카드`<?>`와 타입 매개변수`<E>`는 공통되는 부분이 있어서, 둘 다 사용해도 괜찮을 때가 많다.

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

특별한 쓰임이 없다면 2번째(와일드카드) 방식이 낫다.

- 1번째(타입 매개변수)와 동일하게 어떤 리스트든 이 메서드에 넘기면 원소를 교환해줄 수 있고
- 신경써야 할 타입 매개변수 E도 없기 때문

#### 그럼 어떤 때에 타입 매개변수를 사용하면 좋을까?  원소 추가가 필요할 때

비한정적 와일드카드 타입은 “원소 추가”가 불가능하다.

```java
public static void swap(List<?> list, int i, int j) {
    //list.set() ?
    list.set(i, list.set(j, list.get(i)); // 컴파일 오류!
}
```

```java
error: incompatible types: Object cannot be converted to CAP#1
list.set(i, list.set(j, list.get(i)));
^
where CAP#1 is a fresh type-variable:
CAP#1 extends Object from capture of ?
```

비한정적 타입 매개변수는 null 외에 어떤 값도 넣을 수 없다.

만약 비한정적 매개변수를 사용하고 싶다면, private 도우미 메서드를 따로 작성해야 한다.

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)); 
}
```

# LETS 코드 예시

```
    SDS                LGL
   CELLO ------------> LETS
 - CNTINF          receiveData(DTO)
 - ACKANS
 - GENGES
```

> LetsSdsRequestDto
> 

```json
{
    messageTypeId : "CNTINF"
}
```

> SdsEdiTypeContext.java
> 

```java

//public class CNTINF implements SdsEdiType {}
//public class CGAINF implements SdsEdiType {}
//public class FREINV implements SdsEdiType {}

@Component
public class SdsEdiTypeContext {
    // 한정적 와일드타입 Class<? extends SdsEdiType> 사용
    // => Parameterized Type(파라미터화 타입)은 invariant(불공변) 이기 때문....
    private final Map<String, Class<? extends SdsEdiType>> sdsEdiTypeMap = new HashMap<>();

    public SdsEdiTypeContext(SdsOrderMapper sdsOrderMapper) {
        // 송수신 EDI 타입 전체
        sdsEdiTypeMap.put("CNTINF", CNTINF.class);
        sdsEdiTypeMap.put("CGAINF", CGAINF.class);
        sdsEdiTypeMap.put("FREINV", FREINV.class);
        sdsEdiTypeMap.put("ACKANS", ACKANS.class);
        sdsEdiTypeMap.put("GENRES", GENRES.class);
        ...
    }

    /**
     * @Method Name : getMessageTypeClass
     * @Method Desc : messageTypeId에 대한 ClassName을 반환한다
     * @param messageTypeId (수신 인터페이스 타입 코드)
     */
    public Class<? extends SdsEdiType> getMessageTypeClass(String messageTypeId) {
        Class<? extends SdsEdiType> messageType = MapUtils.getObject(sdsEdiTypeMap, messageTypeId);
        if (messageType == null) {
            throw new IllegalArgumentException("No messageType for messageTypeId: " + messageTypeId);
        }
        return messageType;
    }
```

![{1B0C235D-3320-432E-A846-35A19144B297}.png](attachment:45460236-a273-49cf-9c9a-74ad464c875b:1B0C235D-3320-432E-A846-35A19144B297.png)

> LetsSdsSftpController.java
> 

```java
    /**
     * @Method Name : receiveData
     * @Method Desc : SFTP 서버에서 messageTypeId에 해당하는 파일을 읽어 DB에 저장한다
     * @param request
     */
    @PostMapping(value = "/receiveData")
    public ResponseEntity<Void> receiveData(@RequestBody LetsSdsRequestDto request) {
        Assert.notNull(request.getMessageTypeId(), AssertMessageUtils.getNotNull("messageTypeId"));

        String messageTypeId = request.getMessageTypeId();
        letsSdsSftpService.receiveData(messageTypeId);

        return new ResponseEntity<>(HttpStatus.OK);
    }
```

> LetsSdsSftpService.java
> 

```java
    /**
     * @Method Name : receiveData
     * @Method Desc : SFTP 서버에서 XML 수신 파일을 읽어 DB에 저장한다
     * @param messageTypeId (수신 인터페이스 타입 코드)
     */
    @Transactional
    public void receiveData(String messageTypeId) {
        Assert.notNull(messageTypeId, AssertMessageUtils.getNotNull("messageTypeId"));

        Class<? extends SdsEdiType> messageType = sdsEdiTypeContext.getMessageTypeClass(messageTypeId);
        // Class<? extends SdsEdiType> messageType = CNTINF.class

        receiveData(messageTypeId, messageType);
        // receiveData("CNTINF", CNTINF.class);
    }

    // 타입 파라미터 T를 사용 => 와일드카드로 변경 가능 (T 관리 필요 없어짐)
    // private void receiveData(String messageTypeId, Class<? extends SdsEdiType> sdsEdiType) {
    private <T extends SdsEdiType> void receiveData(String messageTypeId, Class<T> sdsEdiType) {
        Assert.notNull(sdsEdiType, AssertMessageUtils.getNotNull("sdsEdiType"));

        ChannelSftp channelSftp = null;
        try {
            channelSftp = SftpClient.loginSftp(host, port, username, password);
            channelSftp.cd(SDS_RECEIVE_WORKING_DIR);
            @SuppressWarnings("unchecked")
            Vector<ChannelSftp.LsEntry> receivedFiles = channelSftp.ls(".");
            for (ChannelSftp.LsEntry receivedFile : receivedFiles) {
                if (receivedFile.getAttrs().isDir()
                        || !receivedFile.getFilename().toLowerCase().endsWith(SDS_FILE_EXTENSION)) {
                    continue;
                }
                String filename = receivedFile.getFilename();
                convertXmlToObject(channelSftp, filename, sdsEdiType, messageTypeId);
            }
        } catch (JSchException e) {
            throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Failed to Log In\n" + e.getMessage(), null);
        } catch (SftpException e) {
            throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Failed to Change Directory\n" + e.getMessage(), null);
        } catch (Exception e) {
            throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Unrecognized Exception\n" + e.getMessage(), null);
        } finally {
            try {
                SftpClient.logoutSftp(channelSftp);
            } catch (JSchException e) {
                throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Failed to Log Out\n" + e.getMessage(), null);
            }
        }
    }

    // 타입 파라미터 T를 사용 => 와일드카드로 변경 가능 (T 관리 필요 없어짐)
    // private void convertXmlToObject(ChannelSftp channelSftp, String filename, Class<? extends SdsEdiType> sdsEdiType, String messageTypeId) {
    private <T extends SdsEdiType> void convertXmlToObject(ChannelSftp channelSftp, String filename,
            Class<T> sdsEdiType, String messageTypeId) {
        Assert.notNull(channelSftp, AssertMessageUtils.getNotNull("channelSftp"));
        Assert.notNull(filename, AssertMessageUtils.getNotNull("filename"));
        Assert.notNull(sdsEdiType, AssertMessageUtils.getNotNull("sdsEdiType"));

        try (InputStream inputStream = channelSftp.get(filename)) {
            byte[] fileBytes = readInputStreamToByteArray(inputStream);
            try (InputStream validateStream = new ByteArrayInputStream(fileBytes)) {
                if (!isMessageTypeMatching(messageTypeId, validateStream, filename)) {
                    return;
                }
            }
            try (InputStream unmarshalStream = new ByteArrayInputStream(fileBytes)) {
                JAXBContext jaxbContext = JAXBContext.newInstance(sdsEdiType);
                Unmarshaller unmarshaller = jaxbContext.createUnmarshaller();
                SdsEdiType dataObject = (SdsEdiType) unmarshaller.unmarshal(unmarshalStream);
                sdsEdiTypeContext.saveReceivedData(dataObject);
                channelSftp.rename(filename, SDS_RECEIVE_DONE_DIR + "/" + filename);
                log.info("[SDS] File Received Successfully (" + filename + ")");
            }
        } catch (IOException | SftpException | ParserConfigurationException | SAXException e) {
            throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Failed to Read File (" + filename + ").\n" + e.getMessage(), null);
        } catch (JAXBException e) {
            throw new ServiceException("SDS_SFTP_ERROR", "[SDS] Failed to Convert File to Java Object (" + filename + ").\n" + e.getMessage(), null);
        }
    }

...
```