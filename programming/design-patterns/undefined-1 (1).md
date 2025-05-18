# 커맨드 패턴

## 접근

* 특정 작업을 요청하는 쪽과 처리하는 쪽을 분리해야 한다. 이를 위해 특정 작업 수행을 별도 클래스에 캡슐화한다.
* 예를 들어 조명을 켜고 끄는 요청, 커튼을 열고 닫는 요청을 각각 LightOnCommand, LightOffCommand, OpenCurtainsCommand, CloseCurtainsCommand 클래스로 만들고 구체적인 행동은 클래스 내부에 캡슐화할 수 있다.

## 개념

* 요청에 대한 내용을 하나의 객체로 캡슐화하여 객체를 서로 다른 요청 내역에 따라 매개변수화한다. 이를 통해 요청을 큐에 저장하거나 로깅하거나 취소하는 등 요청 처리 방식을 제어할 수 있다.
* 커맨드 객체를 두어 외부에는 실행(execute) 인터페이스만 노출하고, 행동과 리시버는 내부에만 두고 사용한다.
* 커맨드 객체 내부에서 어떤 것이 실행되든 외부에서는 신경쓸 필요가 없다.
* 여러 커맨드를 묶어 실행하려면 매크로 커맨드를 사용하면 된다.

```java
public class MacroCommand implements Command {
    Command[] commands;
    
    public void execute() {
        for (int i=0; i<commands.length; i++) {
            commands[i].execute();
        }
    }
}
```

* execute 메서드를 가진 인터페이스/추상 클래스를 사용하여 어떤 작업으로 객체를 매개변수화한다는 점에서 전략 패턴과 유사하다고 생각할 수 있다. 하지만 전략 패턴은 같은 목적을 가진 여러 알고리즘을 캡슐화하여 여러 전략 중 하나를 사용하는 형태인 데에 반해 커맨드 패턴은 여러 다른 목적을 가진 요청들을 캡슐화하여 모두 클라이언트로부터 사용된다는 점에서 다르다.

## 장단점

* 장점
  * 실행 취소, 다시 실행, 지연 실행 등 요청 처리를 다양하게 할 수 있다.
  * 커맨드들의 집합을 모아 실행시킬 수 있다.
* 단점
  * 클라이언트와 리시버 간 커맨드라는 레이어가 들어가므로 코드가 복잡해진다.

## 사용 방법

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 클라이언트는 커맨드 객체를 생성해 인보커에 전달한다.
* 커맨드 객체에서는 리시버 객체에게 요청을 보내 작업을 수행하도록 한다.
* 함수형 프로그래밍이 지원되는 언어를 사용하면서 커맨드 인터페이스/추상 클래스에 메서드가 하나뿐일 경우, 커맨드 클래스를 만드는 대신 람다 표현식과 같은 함수 객체를 이용할 수 있다.

```java
receiver.setCommand(() -> curtains.open());
```

## 예시

* spring boot cli에서는 커맨드 패턴을 사용해 요청된 명령에 대한 응답을 반환한다. CommandRunner는 Invoker 역할을 하며 Command들의 목록을 담게 된다.

```java
public static void main(String... args) {
    // ...
    CommandRunner runner = new CommandRunner("spring");
    // ...
    runner.addCommand(new ShellCommand());
    runner.addCommand(new HintCommand(runner));
    int exitCode = runner.runAndHandleErrors(args);
}
```

```java
public class CommandRunner implements Iterable<Command> {
    private final List<Command> commands = new ArrayList<>();
    // ...
    public void addCommand(Command command) {
    	Assert.notNull(command, "Command must not be null");
    	this.commands.add(command);
    }
    
    protected ExitStatus run(String... args) throws Exception {
        // ...
        Command command = findCommand(commandName);
        if (command == null) {
              throw new NoSuchCommandException(commandName);
        }
        beforeRun(command);
        try {
              return command.run(commandArguments);
        } finally {
              afterRun(command);
        }
    }
    // ...
}
```

```java
public class HintCommand extends AbstractCommand {

    private final CommandRunner commandRunner;

    public HintCommand(CommandRunner commandRunner) {
        super("hint", "Provides hints for shell auto-completion");
        this.commandRunner = commandRunner;
    }

    @Override
    public ExitStatus run(String... args) throws Exception {
        // ...
  }
```
