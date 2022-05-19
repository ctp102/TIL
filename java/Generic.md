# Generic

## 제네릭 메서드
제네릭 메서드는 static이 가능하다.  
static 메서드는 호출 시점에 타입이 결정되므로 이는 당연하다.  
하지만, 제네릭 클래스에서 static 필드는 JVM이 메모리에 올리는 시점에 타입이 결정되기 때문에 사용이 불가능하다.  

-----
제네릭 메서드는 일반 클래스/제네릭 클래스에서 모두 사용이 가능하다. 또한 제네릭 클래스에 속한 제네릭 메소드여도, 그 타입 변수를 공유하지 않는다.
```java
class Student<T>{
    T name; // 1-1

    public T getName(T name){ this.name = name; return name; }  // 1-2
    
    public <T> T getId(T id){return (T)id;} // 2 제네릭 클래스의 T와 다름  
    
    public <S> T toT1(S id){return (T)id; }  // 3
    
    public static <S> T toT2(S id){return (T)id;}  // 4 에러!!   
}
```

