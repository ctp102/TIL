## 스프링은 왜 Checked Exception은 커밋하고 Runtime Exception은 롤백하는 정책을 잡았을까? 
체크 예외: 비즈니스 의미가 있을 때 사용  
언체크 예외: 복구 불가능한 에러  

비즈니스 예외? 이게 뭔데?

## isolation level
아래 목록은 격리 수준이 낮은 순서
1. READ_UNCOMMITTED
   - 트랜잭션의 데이터 변경 내용을 커밋 전에도 다른 트랜잭션이 읽는 것을 허용
   - Dirty Read 현상 발생(롤백된 내용의 데이터를 읽는 경우)
   - Non-Repeatable, Phantom 현상 발생
2. READ_COMMITTED
   - 트랜잭션의 데이터 변경 내용을 커밋 후에 다른 트랜잭션이 읽는 것을 허용
   - 아직 커밋 전이면 트랜잭션 이전 값을 읽고 커밋 후면 변경된 데이터를 읽음
   - Non-Repeatable 현상 발생(같은 트랜잭션 내에서 SELECT문을 2번 조회했는데 결과값이 불일치하는 경우)
   - Phantom 현상 발생
   - Oracle, SQL Server, H2 Database의 Default 값
3. REPEATABLE_READ
   - 트랜잭션 범위 내에서 조회한 내용이 항상 동일함을 보장
   - Phantom Read 현상 발생(Non-Repeatable의 한 종류로 조건 여부와 상관없이 SELECT문을 사용할 때 나타날 수 있는 현상이며 해당 쿼리로 읽히는 데이터의 행이 새로 생기거나 없어진 경우)
   - MySQL InnoDB Default 값
4. SERIALIZABLE
   - 한 트랜잭션에서 사용하는 데이터를 다른 트랜잭션에서 접근 불가

## 트랜잭션 AOP 주의사항
1. 접근제한자 public을 제외한 나머지 접근제한자에는 @Transactional이 적용되지 않는다.
  - 스프링에서 나머지 접근제한자에는 @Transactional을 사용할 가능성이 낮다고 하여 막았다고 한다..(확인 필요)
2. AOP를 적용하면 스프링은 대상 객체 대신에 프록시 객체를 스프링 빈으로 등록한다. 따라서 스프링은 DI시 실제 객체 대신 프록시 객체를 주입하게 된다. 만약 실제 객체에서 트랜잭션을 적용한 메서드 호출을 한다면, 프록시를 거치지 않고 대상 객체를 직접 호출하므로 트랜잭션이 적용되지 않는다. 
3. @PostConstruct, @Transactional을 함께 사용하면 트랜잭션이 적용되지 않는다. 초기화 코드가 먼저 호출되고, 그 다음에 트랜잭션 AOP가 적용되기 때문이다.

## 2개의 트랜잭션 작업 시 주의할 점
- 논리 트랜잭션(내부 트랜잭션) 중 하나라도 롤백이 되면 물리 트랜잭션(외부 트랜잭션)은 무조건 롤백된다.
- 내부 트랜잭션이 롤백되었는데, 외부 트랜잭션이 커밋된다면 `UncheckedRollbackException` 예외가 발생한다.
- `rollbackOnly` 상황에서 커밋이 발생하면 `UncheckedRollbackException` 예외가 발생한다.
  - `rollbackOnly`는 내부 트랜잭션(논리 트랜잭션)에서 롤백이 발생하는 경우 세팅되는 옵션이다.