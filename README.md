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