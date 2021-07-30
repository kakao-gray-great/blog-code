
## JDBC Transaction의 이해

```java=
Connection connection = dataSource.getConnection();

try (connection) {
    connection.setAutoCommit(false);
    // excute some SQL statements...
    connection.commit();
} catch(SQLException e) {
    connection.rollback();
}
```

1. 데이터베이스를 연결합니다.
2. setAutoCommit의 default value는 true이다. 이것을 setAutoCommit을 false로 지정한다. true로 설정하게 되면 하나의 쿼리마다 자동시작 ~ 자동커밋이 일어난다.우리는 여러 개의 쿼리 문장이 하나의 작업으로 수행되게 할 것이기 때문에 각각의 쿼리가 자동으로 작동되지 못하도록 해야 한다.
3. 쿼리가 실행된 후 commit 한다.
4. 예외가 발생했을 경우 롤백을 한다.

@Transactional도 이와 다르지 않다.


## @Transactional Annotation

```java=
public ProductService() {

    @Transactional
    public Long saveProduct(Product product) {
        // excute some SQL...
        Long productId = productMapper.save(product);
        
        return productId;
    }
}
```

* Spring Configuration에 @EnableTransactionManagement annotation을 붙인다. (Spring boot에서는 자동)

```java=
@Configuration
@EnableTransactionManagement
public class AppConfig {
    
    @Bean
    public PlatformTransactionManager manager() {
        return manager;
    }
}
```

* Spring Confiuration에 Transaction Manager를 지정한다.

* Spring은 @Transactional annotation이 달린 public 메서드에 대해서 내부적으로 데이터베이스 Transaction 코드를 실행해준다.

위 과정을 따르게 되면 아래와 같이 변환된다.

```java=
public ProductService() {

    public Long saveProduct(Product product) {
        Connection connection = dataSource.getConnection();
        
        try (connection) {
            connection.setAutoCommit(false);
            
            // excute some SQL...
            Long productId = productMapper.save(product);
            
            connection.commit();
        } catch(SQLException e) {
            connection.rollback();
        }
        
        return productId;
    }
}
```

## 스프링이 Transaction 코드를 넣는 법

@Transactional은 Spring AOP를 기반으로 동작한다. 

@Transactional이 적용되면 Transaction 처리를 Dynamic Proxy 객체에 대신 위임한다.
Dynamic Proxy 객체는 target이 상속하고 있는 인터페이스를 상속 후 추상 메서드를 구현하며 내부적으로 target 메서드 호출 전후로 transaction 처리를 수행한다.

Controller는 target 메서드를 호출하는 것으로 생각하지만 실제로는 proxy의 메서드를 호출하게 된다.

Target 객체가 인터페이스를 상속하고 있지 않다면, CGLib를 사용하여 인터페이스 대신 target 객체를 상속하는 proxy 객체를 만든다.


### 참고

https://jeong-pro.tistory.com/228
https://hwannny.tistory.com/98