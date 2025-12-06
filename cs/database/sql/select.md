# SELECT

## JOIN

### LATERAL JOIN

* 서브쿼리가 앞에 있는 테이블의 각 행(row)에 접근할 수 있도록 해주는 조인 방식
* **중첩된 질의**가 필요할 때 유용하게 사용할 수 있다.
* 예를 들어 users의 게시물 중 가장 최신 게시물 1개씩만 조회하려고 한다면 다음과 같은 쿼리를 작성할 수 있다.

<pre class="language-sql"><code class="lang-sql">SELECT u.id, u.name, p.title
FROM users u
LEFT JOIN LATERAL (
    SELECT *
    FROM posts p
<strong>    WHERE p.user_id = u.id
</strong>    ORDER BY p.id DESC
    LIMIT 1
) p ON TRUE;
</code></pre>

