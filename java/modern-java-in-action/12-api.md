# 12장: 날짜와 시간 API

## 기존 자바의 날짜와 시간 API

* 자바 1.0부터 제공된 Date의 경우 특정 시점을 날짜가 아닌 밀리초 단위로 표현하며, 1900년을 기준으로 하는 오프셋, 달을 나타낼 때 0부터 시작하는 인덱스 등으로 인해 유용성이 떨어졌다.
* 2017년 9월 21일을 나타내려면 아래와 같이 작성해야하므로 직관적이지 않다.

```java
Date date = new Date(117, 8, 21);
```

* 자바 1.1에서 Calendar 클래스가 제공되었지만 여전히 달을 나타낼 때 0으로 시작하는 인덱스 문제가 있었고 오히려 DateFormat같은 일부 기능은 Date 클래스에만 작동했다.
* DateFormat은 thread safe 하지 않아 두 스레드가 동시에 하나의 포매터로 날짜를 파싱하면 문제가 발생할 수 있다.
* Date와 Calendar는 가변 클래스이기 때문에 다른 곳에서 의도치 않게 수정이 가능하므로 유지보수가 어려워질 수 있다.

## 새로운 날짜와 시간 API

### LocalDate

* 시간을 제외한 날짜를 표현하는 불변 객체
* 어떤 시간대 정보도 포함하지 않는다.
* 아래와 같이 생성하고 값을 조회할 수 있다.

```java
LocalDate date = LocalDate.of(2017, 9, 21);
LocalDate date = LocalDate.parse("2017-09-21");
LocalDate date = LocalDate.now();

int year = date.getYear(); // 2017
Month month = date.getMonth(); // SEPTEMBER
int month = date.getMonthValue(); // 9
int day = date.getDayOfMonth(); // 21
DayOfWeek dow = date.getdayOfWeek(); // THURSDAY
int len = date.lengthOfMonth(); // 31 (달별 일 수)
boolean leap = date.isLeapYear(); // false (윤년 여부)
```

* TemporalField를 이용해서 조회하는 방법도 있다.

```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);
```

### LocalTime

* 시간을 나타내는 클래스
* 아래와 같이 생성하고 값을 조회할 수 있다.

```java
LocalTime time = LocalTime.of(13, 45, 20);
LocalTime time = LocalTime.parse("13:45:20");

int hour = time.getHour();
int minute = time.getMinute();
int second = time.getSecond();
```

### LocalDateTime

* LocalDate와 LocalTime을 쌍으로 가져 날짜와 시간을 모두 표현할 수 있다.
* 아래와 같이 생성할 수 있다.

```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

* LocalDate 혹은 LocalTime으로 추출할 수 있다.

```java
LocalDate date1 = dt1.toLocalDate();
LocalTime time1 = dt1.toLocalTime();
```

### Instant

* 인간 관점이 아닌 기계적인 관점에서 시간을 표현하는 클래스
* unix epoch time(1970년 1월 1일 0시 0분 0초 UTC) 기준으로 특정 지점까지의 시간을 초로 나타낸다.
* 나노초 단위의 정밀도를 제공한다.
* 아래와 같이 생성할 수 있다. 두번째 인자를 통해 나노초 단위로 시간을 보정할 수 있다.

```java
Instant i = Instant.ofEpochSecond(3);
Instant i = Instant.ofEpochSecond(3, 0);
Instant i = Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후의 1억 나노초
```

### Duration

* 두 시간 객체 사이의 지속 시간을 나타내는 클래스
* 정적 팩토리 메서드인 between 메서드에 두 시간 객체를 입력하여 Duration 객체를 생성할 수 있다. 단, 두 객체의 타입은 같아야 한다.
* of나 ofXXX 메서드도 제공하여 다양한 방식으로 객체를 만들 수 있다.

```java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d1 = Duration.between(instant1, instant2);

Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);
```

* 초, 나노초 단위로 시간을 표현하기 때문에 년,월,일 단위로는 Duration을 만들 수 없다.

### Period

* 두 시간 객체 사이의 날짜를 나타내는 클래스
* 년,월,일 단위로 시간을 표현할 수 있다.
* of나 ofXXX 메서드도 제공하여 다양한 방식으로 객체를 만들 수 있다.

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
```

## 날짜 조정, 파싱, 포매팅

### LocalDate

### TemporalAdjusters











