# 어노테이션 프로세서

## 개념

* 소스 코드 레벨에서 컴파일 시 특정 어노테이션이 붙어있는 것을 참조하여 새로운 코드를 생성하거나 수정해주기 위한 기능이다.
* 어노테이션 프로세서를 사용하는 툴로는 롬복과 AutoService 등이 존재한다.
* 인터페이스나 클래스를 상속받아 메서드를 재정의할 때 사용되는 `@Override` 역시 어노테이션 프로세서를 사용한다.
* java agent를 사용해 런타임에 바이트 코드를 조작하는 것과 달리 컴파일 시 조작하기 때문에 런타임에 수행해야 하는 비용이 없어진다.
* 기존 클래스 코드 변경하는 기능은 제공하지 않기 때문에 편법이 필요하다.
* 롬복은 컴파일 시점에 어노테이션 프로세서를 사용해 소스코드의 Abstract Syntax Tree를 조작한다.

## Annotation Processor 만들기

* 여러 라운드에 거쳐 소스 및 컴파일 된 코드를 처리할 수 있다.
* 라운드는 Spring Security의 필터 체인과 비슷한 개념이다.
* 아래는 `@Magic` 이라는 어노테이션이 붙은 인터페이스가 존재하면 해당 인터페이스를 구현하여 `"rabbit!" 이라는 문자열이 반환되는 pullOut 메서드를 가진 클래스`를 생성하도록 하는 프로세서이다.

```java
@AutoService(Processor.class)
public class MagicMojaProcessor extends AbstractProcessor {

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        return Set.of(Magic.class.getName());
    }
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Magic.class);

        for (Element element : elements) {
            Name elementName = element.getSimpleName();
            
            // 인터페이스에만 어노테이션이 적용 가능하다.
            if (element.getKind() != ElementKind.INTERFACE) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Magic annotation can not be used on " + elementName);
            } else {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "Processing " +
                elementName); 
            }
            
            TypeElement typeElement = (TypeElement) element;
            ClassName className = ClassName.get(typeElement);
            
            // 메서드 생성
            MethodSpec pullOut = MethodSpec.methodBuilder("pullOut")
                .addModifiers(Modifier.PUBLIC)
                .returns(String.class)
                .addStatement("return $S", "Rabbit!")
                .build();

            // 타입 생성
            TypeSpec magicMoja = TypeSpec.classBuilder("MagicMoja")
                .addModifiers(Modifier.PUBLIC)
                .addSuperinterface(className)
                .addMethod(pullOut)
                .build();

            Filer filer = processingEnv.getFiler();            
            try {
                JavaFile.builder(className.packageName(), magicMoja)
                    .build()
                    .writeTo(filer);
            } catch (IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "FATAL ERROR: " + e);
            } 
        }

        return true; // 다른 프로세서가 프로세싱하지 않도록 true를 반환한다.
    }
}
```

### `@AutoService`

* 서비스 프로바이더 레지스트리 생성기이다.
* Processor를 적용하기 위해서는 /resource/META-INF/javax.annotation.processor.Processor 파일에 Processor 구현체의 클래스패스를 등록해야 한다.
* 이 파일을 직접 생성해 프로세서를 등록하려면 기존 프로젝트를 먼저 빌드하고, 이후에 파일을 생성해 다시 빌드해야 하는 귀찮은 과정이 있다.
* AutoService 를 사용하면 파일을 자동으로 생성해 프로세서를 등록해주므로 편리하다.

### JavaPoet

* 자바 소스코드 생성하는 유틸리티이다.
* `JavaFile` 클래스를 제공하여 새로운 클래스를 생성할 수 있으며, 이 클래스를 프로세서의 Filer를 통해 파일로 저장할 수 있다.
* Intellij 에서 어노테이션 프로세서 기능을 활성화시키는 이유가 바로 이 어노테이션 프로세서로 만들어낸 클래스 파일을 읽을 수 있기 위함이다.
