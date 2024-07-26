## JAR 파일이 실행될 수 있는 이유 (With Spring)
[참고 링크](https://elsboo.tistory.com/27)
1. gradle의 bootJar 명령어를 실행하면 jar 파일이 생성됨
2. jar 파일은 BOOT-INF, META-INF, org/springframework/boot/loader로 구성
   ### BOOT-INF
    * 우리가 개발한 소스 코드 영역
    * `classes`: JVM이 읽을 수 있도록 컴파일 된 (바이트 코드) 클래스 파일이 존재
    * `lib`: 코드 실행에 필요한 디펜던시 라이브러리들이 존재
    * `classpath.idx`: 스프링 부트가 lib에 있는 외부 라이브러들을 찾을 수 있도록 클래스 경로 명시
    * `layers.idx`: 도커에서 jar 이미지를 만들 때 쓰이는 정보
   ### META-INF
    * 스프링 부트와 우리가 개발한 소스 코드를 연결해 주는 영역
    * `MANIFEST.MF`: jar 파일이 실행될 때 `JarLauncher`가 호출되는데, 여기에서 우리가 개발한 Start-Class (`MainApplication`)를 호출하기 위해 필요한 경로 설정 파일
    * 즉 jar 실행 명령어를 입력하면, `MANIFEST.MF`의 Main-Class인 JarLauncher를 호출하고, JarLauncher가 MANIFEST.MF의 Start-Class인 MainApplication을 호출한다.
    * 여기에서 `프레임워크와 라이브러리의 차이`가 나온다. 흐름 (호출) 통제권이 프레임워크에 있다. 라이브러리는 호출을 개발자가 한다.
   ### org/springframework/boot/loader
    * 스프링 부트 영역
    * spring-boot-loader (gradle이 빌드할 때 넣어줌) 를 통해 jar 안의 jar (nested jar)도 로드되도록 함
3. JarLauncher의 main 메서드 실행, 최상위 부모인 Launcher의 launch 메서드 실행
    ```java
   // JarLauncher
    public static void main(String[] args) throws Exception {
        (new JarLauncher()).launch(args);
    }
    ```
4. main 메서드 클래스명과 클래스 로더를 생성함. 이 기능 구현은 자식 클래스인 ExecutableArchiveLauncher에서 구현됨
    ```java
   // Launcher
    protected void launch(String[] args) throws Exception {
        if (!this.isExploded()) {
            Handlers.register();
        }

        try {
            // ExecutableArchiveLauncher
            ClassLoader classLoader = this.createClassLoader((Collection)this.getClassPathUrls());
            String jarMode = System.getProperty("jarmode");
            String mainClassName = this.hasLength(jarMode) ? JAR_MODE_RUNNER_CLASS_NAME : this.getMainClass();
            this.launch(classLoader, mainClassName, args);
        } catch (UncheckedIOException var5) {
            UncheckedIOException ex = var5;
            throw ex.getCause();
        }
    }
    ```
5. 스레드에 클래스 로더를 지정하며, 런타임 시점에 `리플렉션`을 사용하여 메인 클래스에서 main 메서드를 찾아 호출
    ```java
   // Launcher
    protected void launch(ClassLoader classLoader, String mainClassName, String[] args) throws Exception {
        Thread.currentThread().setContextClassLoader(classLoader);
        Class<?> mainClass = Class.forName(mainClassName, false, classLoader);
        Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
        mainMethod.setAccessible(true);
        mainMethod.invoke((Object)null, args);
    }
   ```
정리하자면 jar 파일 실행 - JarLauncher main 메서드 - Launcher launch 메서드 - 클래스 로더 생성 후 스레드에 클래스 로더를 지정하고 런타임 시점에 리플렉션을 사용하여 메인 클래스에서 main 메서드를 찾게 된다. 이 과정이 이루어지기 위해서는 우리가 작성한 파일이 지정되어 있는 META-INF 파일이 필요하다.
