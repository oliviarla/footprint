# 컴포지트 패턴

## 접근

* 도메인을 트리 형태로 표현할 수 있고 전체를 포괄하는 최상위 객체가 나머지 하위 객체들을 관리하는 형태일 때, 모든 객체들에 동일한 행동을 갖는 메서드를 추가한다.
* 이를 통해 최상위 객체의 메서드에서는 나머지 하위 객체의 메서드들을 호출한 결과를 종합해 반환하면 된다.

## 개념

* 객체를 트리 구조로 구성해 **부분-전체 계층 구조**를 구현한다. 클라이언트에서 개별 객체와 복합 객체를 같은 방법으로 다룰 수 있다.
* 부분-전체 계층 구조를 생성하여 부분들이 계층을 이루고 있지만 모든 부분을 묶어서 전체를 다룰 수 있도록 한다.
* **구성 요소**는 복합 객체이거나 리프(Leaf) 객체이다. **복합(Composite) 객체**에는 여러 구성 요소(Component)들을 담을 수 있다. 루트 구성 요소가 가장 위에 있고 점점 아래로 계층을 이루는 트리 모양이다.
* 예를 들어 Menu 라는 복합 객체에는 구성 요소인 MenuItem들과 또 다른 복합 객체인 Menu가 존재할 수 있는 형태이다.
* 복합 객체와 리프 객체는 인터페이스가 동일해야 하며 이 덕분에 클라이언트는 복합 객체든 리프 객체든 똑같은 명령을 호출할 수 있다.
* 단, 인터페이스를 동일하게 하다보니 의미 없는 메서드가 생길 수 있다. 보통 이 경우 아무 것도 하지 않거나 null, false, UnsupportedException 등을 반환한다.

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

* 재귀적인 합성을 통해 여러 객체들을 정리한다는 관점에서 데코레이터 패턴과 유사하다. 데코레이터 패턴은 래핑된 객체에 책임을 추가해 확장하는 반면 컴포지트 패턴은 래핑된 객체들을 요약하기만 한다.

## 장단점

* 장점
  * 투명성을 확보하여 자식들을 관리하는 기능과 리프 노드로의 기능을 하나의 클래스에 담아 클라이언트가 복합 객체와 리프 노드를 같은 방식으로 다룰 수 있도록 한다.
  * 클라이언트는 복합 객체를 사용하는지 리프 객체를 사용하고 있는지 신경쓰지 않고 메소드 하나만 호출하면 전체 구조를 조회할 수 있다.
  * 객체들을 모아서 관리할 때 유용하다.
* 단점
  * 단일 책임 원칙을 위반한다.
  * 기능이 너무 다른 클래스들에는 공통 인터페이스를 제공하기 어려울 수 있으며, 컴포넌트 인터페이스를 과도하게 일반화하여 이해하기 어렵게 만들 수 있다.

## 사용 방법

* 필요한 메서드들을 모두 Component라는 추상 클래스 혹은 인터페이스에 정의해둔다.
* Component를 상속받는 Leaf, Composite 클래스를 정의한다. 각 구현 클래스에서는 반드시 지원해야 하는 메서드만 정의한다.

## 예시

### 메뉴판 안의 메뉴

* 아침 메뉴판, 점심 메뉴판, 저녁 메뉴판이 각각 있고 점심 메뉴판에는 디저트 메뉴판이, 저녁 메뉴판에는 술 메뉴판이 포함되는 것을 표현하려면 컴포지트 패턴을 사용해야 한다.
* 먼저 추상 클래스를 선언해 복합 객체와 리프 객체를 공통화한다. 공통적으로 처리할 수 있는 메서드는 구현해두고 처리할 수 없으면 UnsupportedOperationException를 반환하도록 해둔다. 혹은 인터페이스로 두어도 된다.

```java
public abstract class MenuComponent {
    
    public void add(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }
    
    public void remove(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }
    
    public MenuComponent getChild(int i) {
        throw new UnsupportedOperationException();
    }
    
    public String getName() {
        throw new UnsupportedOperationException();
    }
    
    public String getDescription() {
        throw new UnsupportedOperationException();
    }
    
    public double getPrice() {
        throw new UnsupportedOperationException();
    }
    
    public boolean isVegeterian() {
        throw new UnsupportedOperationException();
    }
    
    public void print() {
        throw new UnsupportedOperationException();
    }
}
```

* 리프 객체가 될 수 있는 클래스를 구현한다. 하위 메뉴가 없고 메뉴의 특징, 설명을 담는다.&#x20;
* 점심 메뉴인 엔초비파스타나 디저트 메뉴인 브라우니가 이 메뉴 아이템에 해당한다.

```java
public class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegeterian;
    double price;
    
    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }
    
    public String getName() {
        return name;
    }
    
    public String getDescription() {
        return description;
    }
    
    public double getPrice() {
        return price;
    }
    
    public boolean isVegeterian() {
        return vegetarian;
    }
    
    
    public void print() {
        System.out.print("  " + getName());
        if (isVegiterian()) {
            System.out.print("(v)");
        }
        System.out.println(", " + getPrice());
        System.out.println("    -- " + getDescription());
    }
}
```

* 복합 객체를 나타내는 클래스로, 하위 메뉴가 존재할 수 있다.
* 이 클래스를 통해 메뉴판 객체를 만들 수 있으며 아침 메뉴판에는 메뉴들만 있고, 점심 메뉴판에는 메뉴들과 함께 디저트 메뉴판이 들어갈 수 있다.

```java
public class Menu extends MenuComponent {
    List<MenuComponent> menuComponents = new ArrayList<>();
    String name;
    String description;
    
    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public void add(MenuComponent menuComponent) {
        menuComponents.add(menuComponent);
    }
    
    public void remove(MenuComponent menuComponent) {
        menuComponents.remove(menuComponent);
    }
    
    public MenuComponent getChild(int i) {
        return menuComponents.get(i);
    }
    
    public String getName() {
        return name;
    }
    
    public String getDescription() {
        return description;
    }
    
    public void print() {
        System.out.println("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("-----------------------------------");
        
        for (MenuComponent menuComponent : menuComponents) {
            menuComponent.print();
        }
    }
}
```

* 사용자가 하나의 최상위 메뉴 컴포넌트에 `print()` 메서드를 호출하면, 내부적으로 하위 컴포넌트들을 순회하여 모든 메뉴가 출력된다.

```java
MenuComponent breakfastMenu = new Menu("pancakes", "breakfast menus");
MenuComponent lunchMenu = new Menu("pastas", "lunch menus");
MenuComponent dinnerMenu = new Menu("steaks", "dinner menus");
MenuComponent dessertMenu = new Menu("cake", "dessert menus");
MenuComponent drinkMenu = new Menu("alcohol", "drink menus");

MenuComponent allMenus = new Menu("all menus", "all menus");

allMenus.add(breakfastMenu);
allMenus.add(lunchMenu);
allMenus.add(dinnerMenu);

breakfastMenu.add(new MenuItem(...));
breakfastMenu.add(new MenuItem(...));

lunchMenu.add(new MenuItem(...));
lunchMenu.add(dessertMenu);

dinnerMenu.add(new MenuItem(...));
dinnerMenu.add(drinkMenu);

allMenus.print(); // 모든 메뉴 출력
```
