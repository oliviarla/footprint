# Set

## 특징

* 고유한 원소들을 저장하는 자료구조
* 수학의 집합 개념과 유사하다.
* 중복된 값을 저장할 수 없다.
* 저장되는 순서를 유지할 수 없다. 따라서 index 기반 탐색이 불가능하다.
* 값을 추가하거나 삭제할 때&#x20;
* 종류
  * Hash-Based Set
    * 해시 테이블로 표현되는 Set이다.
    * 각 원소는 해시코드에 기반한 버킷에 저장된다.
  * Tree-Based Set
    * 이진 탐색 트리로 표현되는 Set이다.
    * 각 원소는 트리의 노드에 저장된다.

## HashSet

* 사실상 내부 구현은 HashMap을 사용하도록 되어 있다.
* Key로는 Set의 요소들을 가지며, Value로는 Dummy Object(쓸모없는 임시 값)을 가진다.

<pre class="language-java"><code class="lang-java">public class HashSet&#x3C;E>
    extends AbstractSet&#x3C;E>
    implements Set&#x3C;E>, Cloneable, java.io.Serializable
{
    // ...
<strong>    private transient HashMap&#x3C;E,Object> map;
</strong><strong>    // ...
</strong></code></pre>





## TreeSet







## LinkedHashSet

