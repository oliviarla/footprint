# 메멘토 패턴

## 접근

* 객체를 이전의 상태로 복구해야 할 때 등을 대비해 객체의 상태를 저장하는 클래스를 따로 두어 핵심적인 객체의 캡슐화를 유지하도록 한다.

## 개념

* 객체의 구현 세부 사항을 공개하지 않으면서 이전 상태를 저장하고 복원할 수 있게 해주는 디자인 패턴이다.
* 상태를 실제로 소유하고 있는 원본 객체에 상태 스냅샷 생성을 위임한다.
* 객체 상태 복사본은 메멘토에 저장하여 메멘토의 상태를 원본 객체 외에는 접근하지 못하도록 한다.
* 대신 인터페이스를 통해 외부에서 스냅샷의 데이터를 가져올 수 있도록 한다.
* 시스템의 상태 저장 시 직렬화를 사용하는 것이 좋다.
* 오리지네이터 클래스는 자신의 상태에 대한 스냅샷을 생성할 수 있으며 스냅샷을 통해 상태를 복원할 수 있다.
* 메멘토는 오리지네이터의 상태 스냅샷을 저장하는 객체이다. 메멘토는 보통 불변이며 오리지네이터 클래스 내부의 중첩 클래스로 선언한다. 이를 통해 메멘토의 비공개 필드나 메서드에 접근할 수 있다.
* 케어테이커는 메멘토들의 스택을 저장해 기록을 추적할 수 있다.

<figure><img src="../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 저장된 상태를 별도 객체에 보관하여 핵심 객체의 데이터를 캡슐화된 상태로 유지할 수 있다.
  * 복구 기능을 구현하기에 좋다.
  * 원본 클래스와 스냅샷 클래스를 분리하여 원본 클래스의 코드를 단순화할 수 있다.
* 단점
  * 상태를 저장하고 복구하는 데에 시간이 오래 걸릴 수 있다.
  * 케어테이커 내부에서 더 이상 유효하지 않은 메멘토들을 없앨 수 있도록 오리지네이터의 수명주기를 추적해야 한다.

## 예시

* 아래와 같이 Text의 특정 상태에 대해 주기적으로 memento를 생성하고 이를 사용해 복구할 수 있다.

```java
public class TextEditor {
    private String documentText;

    public TextEditor(String initialText) {
        this.documentText = initialText;
    }

    public void insertText(String text, int position) {
        StringBuilder newDocumentText = new StringBuilder(documentText);
        newDocumentText.insert(position, text);
        documentText = newDocumentText.toString();
        System.out.println("Inserted text: " + text);
    }

    public void deleteText(String text, int startPosition, int endPosition) {
        StringBuilder newDocumentText = new StringBuilder(documentText);
        newDocumentText.delete(startPosition, endPosition);
        documentText = newDocumentText.toString();
        System.out.println("Deleted text: " + text);
    }

    public String getDocumentText() {
        return documentText;
    }

    public Memento createMemento() {
        return new Memento(documentText);
    }

    public void restoreMemento(Memento memento) {
        this.documentText = memento.getDocumentText();
    }
    
    public static class Memento {
        private String documentText;
    
        public Memento(String documentText) {
            this.documentText = documentText;
        }
    
        public String getDocumentText() {
            return documentText;
        }
    }
}

public class Client {

    public static void main(String[] args) {
        TextEditor editor = new TextEditor("This is the initial text.");

        Memento memento1 = editor.createMemento(); // Save state after initial text
        editor.insertText("Hello, ", 5); // Insert text

        Memento memento2 = editor.createMemento(); // Save state after inserting text
        editor.deleteText("world", 12, 16); // Delete text

        System.out.println("Current document text: " + editor.getDocumentText());

        // Restore state to before inserting text
        editor.restoreMemento(memento1);
        System.out.println("Restored document text (before inserting text): " + editor.getDocumentText());

        // Restore state to before deleting text
        editor.restoreMemento(memento2);
        System.out.println("Restored document text (before deleting text): " + editor.getDocumentText());
    }
}
```

**출처**

* [https://medium.com/@mehar.chand.cloud/memento-design-pattern-use-case-undo-in-text-editor-c121cff48a6e](https://medium.com/@mehar.chand.cloud/memento-design-pattern-use-case-undo-in-text-editor-c121cff48a6e)
