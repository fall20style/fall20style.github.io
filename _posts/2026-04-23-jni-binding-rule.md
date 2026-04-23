JNI C 코드 작성 시 적용되는 명명 규칙과 주요 특징

## 1. 함수 이름 규칙
JVM이 네이티브 메서드를 찾을 수 있도록 아래 형식을 반드시 지켜야 함.

* 기본 구조: Java_패키지명_클래스명_메서드명으로 작성함.
* 구분자 변경: Java 패키지의 점(.)을 언더바(_)로 모두 교체함.
* 언더바 처리: Java 이름 자체에 _가 있으면 _1로 변환하여 표기함.
* 오버로딩: 매개변수 구분이 필요할 때 이름 뒤에 __를 붙이고 인자 타입을 추가함.



------------------------------
## 2. 필수 구성 요소
C 함수 선언 시 표준적으로 포함되어야 하는 키워드임.

* JNIEXPORT: 함수를 외부에서 호출 가능하도록 내보내는 역할을 함.
* JNICALL: 플랫폼별 호출 규약을 정의하는 매크로임.
* JNIEnv:* Java 환경에 접근하여 객체를 제어할 수 있는 포인터임.
* jobject/jclass: 호출한 Java 객체나 클래스 정보를 담고 있음.
* 

------------------------------
## 3. 코드 적용 예시
com.app.Helper 클래스의 sendData 메서드를 구현하는 경우임.

``` C
// Java: package com.app; class Helper { native void sendData(int n); }
// C 이름: Java_com_app_Helper_sendData
```

``` java
JNIEXPORT void JNICALL Java_com_app_Helper_sendData
  (JNIEnv *env, jobject obj, jint n) {
    // 구현 코드 작성함
}
```

------------------------------
## 4. 데이터 타입 매핑
Java와 C 사이의 데이터 호환을 위해 전용 타입을 사용함.

| Java 타입 | JNI(C) 타입 | 비고 |
|---|---|---|
| int | jint | 32비트 정수형임 |
| long | jlong | 64비트 정수형임 |
| String | jstring | 객체 타입임 |
| void | void | 반환값 없음 |

------------------------------
직접 작성하기 어렵다면 javac -h 명령어로 헤더 파일을 자동 생성하는 것이 가장 정확함.


