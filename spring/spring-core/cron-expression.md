# Cron Expression

* Spring Scheduler에서 사용되는 Cron Expression은 아래와 같이 표현할 수 있다.

```
 ┌───────────── second (0-59)
 │ ┌───────────── minute (0 - 59)
 │ │ ┌───────────── hour (0 - 23)
 │ │ │ ┌───────────── day of the month (1 - 31)
 │ │ │ │ ┌───────────── month (1 - 12) (or JAN-DEC)
 │ │ │ │ │ ┌───────────── day of the week (0 - 7)
 │ │ │ │ │ │          (0 or 7 is Sunday, or MON-SUN)
 │ │ │ │ │ │
 * * * * * *
```

* 다음은 각 자리에 들어갈 수 있는 기호들에 대한 설명이다.

<table><thead><tr><th width="92">기호</th><th>Descrption</th><th>Example</th></tr></thead><tbody><tr><td>*</td><td>모든 값을 의미 (매초/분/시간 마다 수행)</td><td><code>0 0 * * * *</code> : 매 시간마다 실행</td></tr><tr><td>?</td><td>day of the month 혹은 day of the week에 지정 가능하며 *과 같은 의미</td><td></td></tr><tr><td>-</td><td>기간 설정</td><td><code>0 0-5 14 * * ?</code> : 매일 14:00시 0분 부터 5분 마다 실행한다. 즉 0,1,2,3,4,5분에 실행된다.</td></tr><tr><td>,</td><td>값을 나열해 사용</td><td><code>0 0,30 * * * *</code> : 0분, 30분 마다 실행된다.</td></tr><tr><td>/</td><td>시작과 반복간격을 지정</td><td><code>30/30 * * * * ?</code> : 30초에 시작해서 30초 마다 실행</td></tr><tr><td>L</td><td><p>마지막 날에 동작</p><p>day of the month, day of the week에서만 사용한다.</p></td><td><code>0 0 0 L * ?</code> : 매월 말일 자정에 실행<br><code>0 15 10 ? * 6L</code> : 매월 마지막 금요일(6) 10:15분에 실행<br><code>0 0 0 L-3 * *</code> : 매월 마지막 날로부터 3일 전인 날짜의 자정에 실행</td></tr><tr><td>W</td><td><p>가장 가까운 평일에 동작<br>지정한 날짜가 토요일이라면 가까운 평일인 금요일에 동작하고, 일요일이라면 월요일에 동작한다.</p><p>day of the month 에서만 사용한다.</p></td><td><code>0 0 0 1W * *</code> : 매월 첫 번째 평일 자정에 실행<br><code>0 0 0 LW * *</code> : 매월 마지막 평일 자정에 실행</td></tr><tr><td>#</td><td><p>몇째주(뒤)의 무슨 요일(앞)을 설정</p><p>day of week에서만 사용한다.</p></td><td><code>0 0 0 * * 6#3</code> : 셋째주 금요일 자정에 실행</td></tr></tbody></table>
