# item 37) ordinal 인덱싱 대신 EnumMap을 사용하라

## ordinal 인덱싱

* 아래는 LifeCycle 상수의 ordinal 인덱스 값을 배열의 인덱스로 사용하여, 특정 LifeCycle에 해당되는 Plant들을 배열 내부 Set에 추가한다.
* 배열이 제네릭과 호환되지 않아 비검사 형변환 수행해야 한다.
* 배열의 각 인덱스에 해당하는 의미를 모르므로 출력 결과에 직접 레이블을 달아주어야 한다.
* 정수는 열거 타입과 달리 타입 안전하지 않으므로 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

```java
public static void usingOrdinalArray(List<Plant> garden) {
    Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
    }

    for (Plant plant : garden) {
        plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
    }

    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
```

## EnumMap

* 열거 타입을 키로 사용하도록 설계하여, 실질적인 열거 타입 상수를 값으로 매핑하도록 한다.
* 내부 구현 방식은 배열으로 되어있지만, 내부로 감춰져 Map의 타입 안전성과 배열의 성능을 가진다.

```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

for (LifeCycle lifeCycle : LifeCycle.values()) {
    plantsByLifeCycle.put(lifeCycle,new HashSet<>());
}

for (Plant plant : garden) {
    plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}
```

* 스트림과 EnumMap을 함께 사용하는 예제이다.
* 생애주기에 속하는 Plant 객체가 있을 경우에만 EnumMap의 키가 생성된다.

```java
public static void streamEnumMap(List<Plant> garden) {
    Map plantsByLifeCycle = garden.stream().collect(Collectors.groupingBy(plant -> plant.lifeCycle,
                    () -> new EnumMap<>(LifeCycle.class),Collectors.toSet()));
    System.out.println(plantsByLifeCycle);
}
```

## 중첩 EnumMap

* 이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵
* 실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(values())
                .collect(Collectors.groupingBy(t -> t.from,
                        () -> new EnumMap<>(Phase.class),
                        Collectors.toMap(t -> t.to,
                                t -> t,
                                (x,y) -> y,
                                () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }

}
```
