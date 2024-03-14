---
description: 자바의 새로운 날짜, 시간 클래스를 알아본다.
---

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

### 기존 클래스로부터 새로운 클래스 생성

* 자바의 새로운 날짜와 시간 클래스들은 모두 불변이다. 대신 기존 객체에 값을 변경하여 새로운 객체를 만들어내는 메서드를 제공하여 편리하게 사용할 수 있다.
* `withXXX` 메서드를 제공하여 기존의 LocalDate 객체에서 특정 값만 변경하여 새로운 LocalDate 객체를 반환한다.
  * 내부의 변수를 바꾸는 것이 아닌 새로운 객체를 반환하는 것이므로 새로운 변수에 할당해주어야 한다.
  * 첫번째 인수로 TemporalField를 입력할 수도 있다.

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
```

* 지정된 시간을 추가하거나 뺄 수도 있다.

```
LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeek(1); // 2017-09-28
LocalDate date3 = date2.minusYear(6); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

### TemporalAdjusters

* 다음주 일요일, 돌아오는 평일 등 복잡한 날짜 조정 기능을 정적 메서드로 제공하는 클래스이다.

| method                       | description                                |
| ---------------------------- | ------------------------------------------ |
| dayOfWeekInMonth             |  서수 요일에 해당하는 날짜를 반환                        |
| firstDayOfMonth              |   현재 달의 첫 번쨰 날짜를 반환                        |
| firstDayOfNextMonth          |  다음 날의 첫 번째 날짜를 반환                         |
| firstDayOfNextYear           |   내년의 첫 번째 날짜를 반환                          |
| firstDayOfYear               |   올해의 첫 번째 날짜를 반환                          |
| firstInMonth                 |   현재 달의 첫 번째 요일에 해당하는 날짜를 반환               |
| lastDayOfMonth               |   현재달의 마지막 날짜를 반환                          |
| lastDayOfNextMonth           |   다음 달의 마지막 날짜를 반환                         |
| lastDayOfNextYear            |   내년의 마지막 날짜를 반환                           |
| lastDayOfYear                |   올해의 마지막 날짜를 반환                           |
| lastInMonth                  |   현재 달의 마지막 요일에 해당하는 날짜 반환                 |
|  next / previous             |  현재 달에서 현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환 |
|  nextOrSame / previousOrSame |  현재 날짜 이후로 지정한 요일이 처음 / 이전으로 나타나는 날짜를 반환   |

* 커스텀 TemporalAdjuster도 구현 가능하다.

```java
public class NextWorkingDay implements TemporalAdjuster {
    @Override
    public Temporal adjustInto(Temporal temporal) {
        DayOfWeek today = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK)); // 현재 날짜 읽기
        int dayToAdd = 1;
        if (today == DayOfWeek.FRIDAY) {
            dayToAdd = 3;
        } else if (today == DayOfWeek.SATURDAY) {
            dayToAdd = 2;
        }
        return temporal.plus(dayToAdd, ChronoUnit.DAYS);
    }
}
```

### DateTimeFormatter

* 정적 팩토리 메서드와 상수를 이용해 포매터를 만들 수 있다.
* 아래와 같이 포매터를 이용해 날짜 객체를 문자열으로 만들 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
date.format(DateTimeFormatter.BASIC_ISO_DATE); //20140318
```

* 혹은 문자열을 파싱해 객체로 만들 수 있다.

```java
LocalDate parse = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate parse2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

* 원하는 패턴을 입력하여 포매터를 사용할 수 있다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate localDate = LocalDate.of(2014, 3, 18);
String formattedDate = localDate.format(formatter);
LocalDate parse1 = LocalDate.parse(formattedDate, formatter);
```

* Locale 정보를 넣어 국가별 최적화된 포매터도 만들어 사용할 수 있다. `parseCaseInsensitive()`를 통해 정해진 형식과 정확히 일치하지 않는 입력을 해석할 수 있도록 한다.

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
                .appendText(ChronoField.DAY_OF_MONTH)
                .appendLiteral(". ")
                .appendText(ChronoField.MONTH_OF_YEAR)
                .appendLiteral(" ")
                .appendText(ChronoField.YEAR)
                .parseCaseInsensitive()
                .toFormatter(Locale.ITALIAN);
```

## 시간대

### ZoneId

* 새로운 날짜와 시간 API의 편리함 중 하나는 시간대를 간단하게 처리할 수 있다는 점이다.
* 기존 TimeZone을 대체할 수 있는 ZoneId 클래스가 새롭게 등장했다.
* 새로운 클래스를 이용하면 서머타임같은 복잡한 사항이 자동으로 처리된다.
* ZoneRules 클래스에 40여개의 시간대가 있다.
* 기존의 TimeZone 객체에서 toZoneId 메서드를 사용해 ZoneId 객체를 얻을 수 있다.

```java
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

* LocalDateTime을 비롯한 클래스들을 ZonedDateTime으로 변경할 수 있다.

```java
ZoneId krZone = ZoneId.of("Asia/Tokyo");
LocalDateTime dt = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt = dt.atZone(krZone);
```

### ZoneOffset

* UTC / GMC 기준으로 시간대를 표현하기 위해 ZoneOffset 클래스를 사용할 수 있다.

```java
ZoneOffset nyoffset = ZoneOffset.of("-5:00");
LocalDateTime dt = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime odt = OffsetDateTime.of(dt, nyoffset);
```

### ISO-8601 외의 캘린더 시스템

* 전세계적으로 통용되는 ISO-8601 캘린더 시스템 외에 ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate를 제공한다.
