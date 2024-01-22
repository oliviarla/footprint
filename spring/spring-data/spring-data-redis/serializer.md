# Serializer

* Spring Data Redis에서 Jedis 또는 Lettuce 라이브러리를 사용해 데이터를 저장할 때 byte array 타입을 사용한다. 사용자가 입력한 다양한 클래스 타입을 byte array형태로 변환해 Redis 서버에 저장하기 위해 Serializer가 필요하다.

## 용도 별 Serializer 지정 방법

* 사용자는 RedisTemplate을 생성할 때 setter를 사용해 용도에 따른 Serializer를 지정할 수 있다.
  * &#x20;DefaultSerializer: 외부적으로 setXXXSerializer로 지정되지 않았을 때, DefaultSerializer가 사용된다.
  * KeySerializer: Redis 캐시 데이터의 key를 어떻게 Serialize할 지 지정할 수 있다. 보통은 key를 String으로 지정하기 때문에 StringRedisSerializer를 지정한다.
  * ValueSerializer: Redis 캐시 데이터의 value를 어떻게 Serialize할 지 지정할 수 있다. 상황에 따라 자바 직렬화를 사용하거나 Json 형태로 변환해 저장할 수 있다.
  * HashKeySerializer: Redis Hashes 데이터의 key를 어떻게 Serialize할 지 지정할 수 있다.
  * HashValueSerializer: Redis Hashes 데이터의 key를 어떻게 Serialize할 지 지정할 수 있다.
  * StringSerializer:  항상 String 데이터를 다루는 API에서 어떻게 Serialize할 지 지정할 수 있다. 극소수의 API에서만 사용된다.

## Serializer 종류

### **JdkSerializationRedisSerializer**

* redis의 기본 Serializer이다.
* Serializable에 의해 직렬화된 byte가 redis에 저장된다.
  *   Serializable을 구현하지 않으면 다음 에러가 발생한다.

      ```
      DefaultSerializer requires a Serializable payload but received an object of type
      ```

### StringRedisSerializer

* 보통 key 값을 직렬화하기 위해 사용한다.
  * `spring session data redis` 에서도 기본적으로 이 Serializer를 사용해 key, hashkey를 저장한다.
* String 자료형인 경우에만 사용할 수 있다.
* Serializable에 의해 직렬화되는 것이 아니기 때문에 깔끔하게 String 내용만 저장된다.
  * Serializable 사용 시 `\xac\xed\x00\x05t\x00\x03key` 가 키값으로 저장된다.
  * StringRedisSerializer 사용 시 `key` 가 키값으로 저장된다.
* String 타입이 아닌 객체를 저장하려면, ObjectMapper로 직접 `JSON String ↔ 객체` 변환해주는 코드를 작성해도 된다.
  * 이 경우 클래스 정보를 넣지 않기 때문에 다른 서버에서 활용도 할 수 있다.

### GenericJackson2JsonRedisSerializer

* redis value값을 class type 상관없이 값을 불러 올수있다.

```java
redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
```

* 다음과 같은 형태로 클래스 타입과 함께 저장된다.

```json
{
    "@class": "jam2in.redis.demo.domain.Title",
    "id": 1,
    "description": "Mission Impossible",
    "director": {
        "@class": "jam2in.redis.demo.domain.Director",
        "name": "Christopher McQuarrie",
        "nationality": "US",
        "sex": "M"
    }
}
```

* LocalDateTime 등 지원되지 않는 클래스가 있다면, 커스텀 ObjectMapper를 생성자에 입력해주어야 한다. 이 때 activateDefaultTyping을 넣어주어야 커스텀 ObjectMapper가 클래스 타입을 함께 JSON String으로 변환한다.

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.registerModule(new JavaTimeModule()); // LocalDateTime도 ObjectMapper에서 처리할 수 있도록 모듈 등록
objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
objectMapper.activateDefaultTyping(objectMapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);

return new GenericJackson2JsonRedisSerializer(objectMapper);
```

### Jackson2JsonRedisSerializer

* 아래와 같이 특정 클래스 대상으로만 지정하면 정상적으로 Serialize 가능하다.
* Object 클래스를 대상으로 지정하면 Redis로부터 데이터를 객체로 변환할 때 LinkedHashMap으로 먼저 변환된다. 이렇게 LinkedHashMap으로 변환되면, 커스텀 클래스로 바로 형변환할 수 없는 문제가 있다.

```java
redisTemplate.setHashValueSerializer(new Jackson2JsonRedisSerializer<>(Title.class));
```

* 다음과 같은 형태로 저장된다.

```json
{
    "id": 1,
    "description": "Mission Impossible",
    "director": {
        "name": "Christopher McQuarrie",
        "nationality": "US",
        "sex": "M"
    }
}
```
