https://blogs.oracle.com/java/post/detaching-graalvm-from-the-java-ecosystem-train
https://gemini.google.com/app/854b5076ca9a53de
오라클 블로그 게시물 "Detaching GraalVM from the Java Ecosystem Train"의 요약은 다음과 같습니다.

### 요약

오라클은 2022년 GraalVM과 자바 개발을 일치시키겠다는 발표 이후, GraalVM에 대한 자사 로드맵을 변경한다고 밝혔습니다. 주요 내용은 다음과 같습니다.

- **GraalVM 팀의 초점 변경:** GraalVM 팀은 이제 Java 외의 언어인 GraalPy(Python) 및 GraalJS(JavaScript)에 집중할 예정입니다.
    
- **GraalVM for JDK 24의 종료:** GraalVM for JDK 24가 오라클 Java SE 제품의 일부로 라이선스 및 지원되는 마지막 GraalVM 버전입니다.
    
- **Graal JIT 중단:** Oracle JDK 24가 실험적이고 선택적인 Graal JIT를 포함하는 마지막 릴리스입니다. 오라클은 Graal JIT를 사용하는 고객에게 Oracle JDK JIT로 마이그레이션할 것을 권장합니다.
    
- **Native Image 기술 중단:** Java SE 제품 고객을 위한 Native Image를 포함한 GraalVM 얼리 어댑터 기술이 중단됩니다. 대신, 자바 프로그램의 시작 시간, 최고 성능 도달 시간, 메모리 사용량 개선 목표는 OpenJDK의 **Project Leyden**을 통해 표준 자바의 일부로 계속 진행될 것입니다.
    

결론적으로, 오라클은 더 이상 GraalVM의 Java 관련 기술을 자체 JDK 제품에 포함하지 않고, 대신 Project Leyden을 통해 기본 Java 플랫폼 내에서 AOT(Ahead-of-Time) 컴파일과 같은 기술을 통합하는 데 주력할 것입니다.