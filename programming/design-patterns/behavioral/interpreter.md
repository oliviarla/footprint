# 인터프리터 패턴

## 접근

* 특정 행동을 수행하기 위해 일정한 문법을 갖는 언어를 처리하기 위해 이를 해석할 수 있는 인터프리터가 필요하다.
* 문법과 구문을 번역하는 인터프리터 클래스를 기반으로 간단한 언어를 정의하고, 언어에 속하는 규칙을 나타내는 클래스를 사용해 언어를 표현한다.

## 개념

* 언어 문법을 해석하고 평가하는 방법을 정의하는 디자인 패턴이다.
* 각 문법 표현식이나 규칙마다 클래스를 두어 언어의 해석 메커니즘을 제공한다.
* 복잡한 표현식을 해석하기 위해 각각의 클래스를 조합하도록 한다.
* 아래와 같은 컴포넌트들로 분리할 수 있다.
  * AbstractExpression : interpret 메서드를 가지는 추상 클래스 혹은 인터페이스이다. 표현식을 위해 생성된 모든 구체 클래스의 공통 인터페이스이다.
  * TerminalExpression : 문법의 가장 기본 단위 표현식(말단 표현식)을 나타내는 구체 클래스로, 인터프리터가 언어를 해석하는 데 사용하는 기본 구성 요소이다. 더이상 분해될 수 없다.
  * NonterminalExpression : 복합적인 표현식을 나타내는 구체 클래스로, 여러 하위 표현식을 사용하여 해석할 수 있도록 한다.
  * Context : 인터프리터에서 사용해야 하는 전역 변수들을 담는 클래스이다. 변수, 자료구조, 상태 정보 등을 담을 수 있으며 이는 인터프리터에서 접근, 변경할 수 있다.
  * Client : Interpreter 객체를 생성하고 interpret 메서드를 호출하여 입력된 명령에 대한 결과를 반환한다. AST는 입력 언어를 구문 분석하고 표현식의 계층적 표현을 구성하여 생성된다.
  * Interpreter : 번역 과정을 조정하는 책임이 있다. Abstract Syntax Tree를 생성하고, 컨텍스트를 관리하고, 입력 표현식을 나타내는 표현식 객체를 만들고, AST를 순회하고 평가하여 표현식을 해석한다. 문법에 따라 구문 분석, 표현식 트리 작성 및 표현식 해석을 위한 로직을 캡슐화한다.
* 컴포넌트들에 대한 예시는 다음과 같다.
  * TerminalExpression: 숫자 5, 10과 같은 상수
  * NonTerminalExpression: 덧셈이나 곱셈 같은 연산자
  * Context: 연산에 필요한 변수나 상수 값
  * Client: "5 + 10"이라는 수식을 입력받아 구문 트리를 생성하고, 결과를 반환

## 장단점

* 장점
  * 문법을 클래스로 표현해 쉽게 언어를 구현할 수 있으며 쉽게 변경하거나 확장할 수 있다.
  * 메서드를 추가해 원하는 기능을 더할 수 있다.
  * 해석하는 로직은 인터프리터의 컴포넌트에 넘기고, 클라이언트는 입력된 표현식만 관리하면 된다.
* 단점
  * 단순하고 간단히 문법을 만드는 데에는 좋지만 효율 측면에서 좋지 않을 수 있다.
  * 문법 규칙의 개수가 많아지면 매우 복잡해지므로 파서나 컴파일러 생성기를 사용하는 것이 좋다.

## 예시

* 아래와 같이 숫자를 더하는 문법을 해석할 수 있도록 인터프리터 패턴을 구현할 수 있다.

```java
interface Expression {
	int interpret(Context context);
}

class NumberExpression implements Expression {
	private int number;

	public NumberExpression(int number) {
		this.number = number;
	}

	@Override
	public int interpret(Context context) {
		return number;
	}
}

class AdditionExpression implements Expression {
	private Expression left;
	private Expression right;

	public AdditionExpression(Expression left, Expression right) {
		this.left = left;
		this.right = right;
	}

	@Override
	public int interpret(Context context) {
		return left.interpret(context) + right.interpret(context);
	}
}

class MultiplicationExpression implements Expression {
	private Expression left;
	private Expression right;

	public MultiplicationExpression(Expression left, Expression right) {
		this.left = left;
		this.right = right;
	}

	@Override
	public int interpret(Context context) {
		return left.interpret(context) * right.interpret(context);
	}
}

class Interpreter {
	private Context context;

	public Interpreter(Context context) {
		this.context = context;
	}

	public int interpret(String expression) {
		Expression expressionTree = buildExpressionTree(expression);
		return expressionTree.interpret(context);
	}

	private Expression buildExpressionTree(String expression) {
		return new AdditionExpression(
			new NumberExpression(2),
			new MultiplicationExpression(
				new NumberExpression(3),
				new NumberExpression(4)
			)
		);
	}
}

public class Client {
	public static void main(String[] args) {
		String expression = "2 + 3 * 4";
		
		Context context = new Context();
		Interpreter interpreter = new Interpreter(context);
		
		int result = interpreter.interpret(expression);
		System.out.println("Result: " + result);
	}
}

```

**출처**

* [**https://www.geeksforgeeks.org/interpreter-design-pattern/**](https://www.geeksforgeeks.org/interpreter-design-pattern/)
