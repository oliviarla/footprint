# Spring Data Couchbase

## Cache Abstraction 어노테이션 사용하기

#### build.gradle 작성

* Spring Data Couchbase 의존성을 추가해준다.

```groovy
implementation 'org.springframework.data:spring-data-couchbase'
```

#### docker로 Couchbase 서버 실행하기

* 캐싱 데이터를 저장할 Couchbase 서버를 docker를 이용해 간단히 실행한다.

```sh
docker run -d --name couchbase-local -p 8091-8096:8091-8096 -p 11210-11211:11210-11211 couchbase
```

#### Cache Configuration 작성

* Spring Data Couchbase를 사용하기 위해서 가장 첫번째 해야할 일은 couchbase 서버와 연결하는 것이다.
* docker compose파일로 띄운 couchbase 서버에 연결하기 위해 아래와 같이 Configuration을 작성한다.
* `@Cacheable`과 같은 어노테이션을 사용해 캐싱하려면, Configuration에 `@EnableCaching` 어노테이션을 명시해주어야 한다.
* CouchbaseCacheManager 빌더에서 CacheName과 CacheConfiguration을 등록해주어야 `@Cacheable` 어노테이션에서 사용할 수 있다.

```java
@EnableCaching
@Configuration
public class CouchbaseCacheConfig {
    @Value("${couchbase.url}")
    private String url;
    
    @Value("${couchbase.username}")
    private String username;
    
    @Value("${couchbase.password}")
    private String password;
    
    @Value("${couchbase.bucket}")
    private String bucket;
    
    
    @Bean(destroyMethod = "disconnect")
    public Cluster cluster() {
      return Cluster.connect(url, username, password);
    }
    
    @Bean
    public CouchbaseCacheManager couchbaseCacheManager() {
      return CouchbaseCacheManager.builder(new SimpleCouchbaseClientFactory(cluster(), bucket, null))
          .withCacheConfiguration("post",
              CouchbaseCacheConfiguration.defaultCacheConfig()
                  .entryExpiry(Duration.ofSeconds(180L))).build();
    }
}
```

#### 캐시 어노테이션 사용하기

* CouchbaseCacheManager 빈 등록 시 추가해준 CacheConfiguration의 이름을 명시하면, 해당 CacheConfiguration의 속성을 사용해 캐싱된다.

```java
@Cacheable(cacheNames = "post")
public List<Post> getTodaysPosts(TOPIC topic) {
  return postRepository.getTodaysPosts(topic);
}
```

* 별도의 키를 지정하거나 KeyGenerator를 사용하지 않는다면 `post::NEWS` 키에 메서드의 반환 값이 저장된다.
* 캐시에 데이터가 없는 상태일 때 요청이 오면 캐시에 데이터를 저장하게 되고, 캐시에 데이터가 있는 상태라면 캐시의 데이터를 조회해 곧바로 반환하게 된다.
