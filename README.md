# TIL

## 7. 스프링 MVC - 웹페이지 만들기

### 7-1. 프로젝트 생성

#### 스프링 부트 스타터 사이트에서 스프링 프로젝트 생성

[6장](https://github.com/Jungmin-Seo0527/springmvc#6-1-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%83%9D%EC%84%B1) 과 같은 설정으로
생성

##### build.gradle

```
plugins {
	id 'org.springframework.boot' version '2.5.1'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

```

#### index.html - Welcome 페이지

* `src/main/resources/static/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li>상품 관리
        <ul>
            <li><a href="/basic/items">상품 관리 - 기본</a></li>
        </ul>
    </li>
</ul>
</body>
</html>
```

### 7-2. 요구사항 분석

* 상품 도메인 모델
    * 상품 ID
    * 상품명
    * 가격
    * 수량

* 상품 관리 기능
    * 상품 목록
    * 상품 상세
    * 상품 등록
    * 상품 수정


* 서비스 화면
  ![](https://i.ibb.co/jfnGVyN/bandicam-2021-06-12-10-43-48-460.jpg)
  ![](https://i.ibb.co/j8r2Gqf/bandicam-2021-06-12-10-45-04-565.jpg)
  ![](https://i.ibb.co/RpX2vNM/bandicam-2021-06-12-10-45-37-718.jpg)
  ![](https://i.ibb.co/G2LpQ8J/bandicam-2021-06-12-10-46-10-078.jpg)

#### 서비스 제공 흐름

![](https://i.ibb.co/98DNvkw/bandicam-2021-06-12-10-46-45-111.jpg)

요구사항이 정리되고 디자이너, 웹 퍼블리셔, 백엔드 개발자가 업무를 나누어 진행한다.

* 디자이너: 요구사항에 맞도록 디자인하고, 디자인 결과물을 웹 퍼블리셔에게 넘겨준다.
* 웹 퍼블리셔: 디자이너에게 받은 디자인을 기반으로 HTML, CSS를 만들어 개발자에게 제공한다.
* 백엔드 개발자: 디자이너, 웹 퍼블리셔를 통해서 HTML 화면이 나오기 전까지 시스템을 설계하고, 핵심 비즈니스 모델을 개발한다. 이후 HTML이 나오면 이 HTML을 뷰 템플릿으로 변환해서 동적으로 화면을
  그리고, 또 웹 화면의 흐름을 제어한다.

> 참고    
> React, Vue.js 같은 웹 클라이언트 기술을 사용하고, 웹 프론트엔드 개발자가 별도로 있으면, 웹 프론트엔드 개발자가 웹 퍼블리셔 역할까지 포함해서 하는 경오도 있다.   
> 웹 클라이언트 기술을 사용하면, 웹 프론트엔드 개발자가 HTML을 동적으로 만드는 역할과 웹 화면의 흐름을 담당한다. 이 경우 백엔드 개발자는 HTML 뷰 템플릿을 직접 만지는 대신에, HTTP API를 통해 웹 클라이언트가 필요로 하는 데이터와 기능을 제공하면 된다.

## Note