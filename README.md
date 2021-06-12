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

### 7-3. 상품 도메인 개발

#### Item - 상품 객체

* `src/main/java/hello/itemservice/domain/item/Item.java`

```java
package hello.itemservice.domain.item;

import lombok.Data;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}

```

#### ItemRepository - 상품 저장소

* `src/main/java/hello/itemservice/domain/item/ItemRepository.java`

```java
package hello.itemservice.domain.item;

import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Repository
public class ItemRepository {

    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }

    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    public void clearStore() {
        store.clear();
    }
}

```

#### ItemRepositoryTest - 상품 저장소 테스트

* `src/test/java/hello/itemservice/domain/item/ItemRepositoryTest.java`

```java
package hello.itemservice.domain.item;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

class ItemRepositoryTest {

    ItemRepository itemRepository = new ItemRepository();

    @AfterEach
    void afterEach() {
        itemRepository.clearStore();
    }

    @Test
    void save() {
        // given
        Item item = new Item("itemA", 10000, 10);

        // when
        Item savedItem = itemRepository.save(item);

        // then
        Item findItem = itemRepository.findById(item.getId());
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void findAll() {
        // given
        Item item1 = new Item("item1", 10000, 10);
        Item item2 = new Item("item2", 10000, 20);

        itemRepository.save(item1);
        itemRepository.save(item2);

        // when
        List<Item> result = itemRepository.findAll();

        // then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(item1, item2);
    }

    @Test
    void updateItem() {
        // given
        Item item = new Item("itemA", 10000, 10);

        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();

        // when
        Item updateParam = new Item("item2", 20000, 30);
        itemRepository.update(itemId, updateParam);

        // then
        Item findItem = itemRepository.findById(itemId);
        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());
    }
}
```

### 7-4. 상품 서비스 HTML

#### 부트스트랩

HTML을 편리하게 개발하기 위해 부트스트랩을 사용했다.

* 부트스트랩 공식 사이트: https://getbootstrap.com/
* 부트스트랩을 다운로드 받고 압축을 푼다.
    * 이동: https://getbootstrap.com/docs/5.0/getting-started/download/
    * Compiled CSS and JS 항목을 다운로드 한다.
    * 압축을 풀고 `bootstrap.min.css`를 복사해서 다음 폴더에 추가한다.
    * `resources/static/css/bootstrap.min.css`

> 참고    
> 부트스트랩(Bootstrap)은 웹사이트를 쉽게 만들 수 있게 도와주는 HTML, CSS, JS 프레임워크이다.    
> 하나의 CSS로 휴대폰, 태블릿, 데스크탑까지 다양한 기기에서 작동한다. 다양한 기능을 제공하여 사용자가 쉽게 웹사이트를 제작, 유지, 보수할 수 있도록 도와준다.

#### HTML, css 파일

* `/resources/static/css/bootstrap.min.css` -> 부트스트랩 다운로드
* `/resources/static/html/items.html` -> 아래 참조
* `/resources/static/html/item.html`
* `/resources/static/html/addForm.html`
* `/resources/static/html/editForm.html`

참고로 `/resources/static`에 넣어두었기 때문에 스프링 부트가 정적 리소스를 제공한다.

* http://localhost:8080/html/items.html   
  그런데 정적 리소스여서 해당 파일을 탐색기를 통해 직접 열어도 동작하는 것을 확인할 수 있다.

> 참고    
> 이렇게 정적 리소스가 공개되는 `/resources/static`폴더에 HTML을 넣어두면, 실제 서비스에서도 공개된다. 서비스를 운영한다면 지금처럼 공개할 필요없는 HTML을 두는 것은 주의하자.

#### items.html - 상품 목록 HTML

* `src/main/resources/static/html/items.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>상품 목록</h2>
    </div>
    <div class="row">
        <div class="col">
            <button class="btn btn-primary float-end"
                    onclick="location.href='addForm.html'" type="button">상품
                등록
            </button>
        </div>
    </div>
    <hr class="my-4">
    <div>
        <table class="table">
            <thead>
            <tr>
                <th>ID</th>
                <th>상품명</th>
                <th>가격</th>
                <th>수량</th>
            </tr>
            </thead>
            <tbody>
            <tr>
                <td><a href="item.html">1</a></td>
                <td><a href="item.html">테스트 상품1</a></td>
                <td>10000</td>
                <td>10</td>
            </tr>
            <tr>
                <td><a href="item.html">2</a></td>
                <td><a href="item.html">테스트 상품2</a></td>
                <td>20000</td>
                <td>20</td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```

#### item.html - 상품 상세 HTML

* `src/main/resources/static/html/item.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
    <style> .container {
 max-width: 560px;
 }     
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2>
    </div>
    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control"
               value="1" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control"
               value="상품A" readonly>
    </div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control"
               value="10000" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control"
               value="10" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg" onclick="location.href='editForm.html'" type="button">상품 수정
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg"
                    onclick="location.href='items.html'" type="button">목록으로
            </button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```

#### addForm.html - 상품 등록 폼 HTML

* `src/main/resources/static/html/addForm.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
 .container {
 max-width: 560px;
 }    
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 등록 폼</h2>
    </div>
    <h4 class="mb-3">상품 입력</h4>
    <form action="item.html" method="post">
        <div><label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="formcontrol"
                   placeholder="이름을 입력하세요">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="formcontrol"
                   placeholder="수량을 입력하세요">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">상품
                    등록
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'" type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

#### editForm.html - 상품 수정 폼 HTML

* `src/main/resources/static/html/editForm.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <link href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
 .container {
 max-width: 560px;
 }  
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 수정 폼</h2>
    </div>
    <form action="item.html" method="post">
        <div>
            <label for="id">상품 ID</label>
            <input type="text" id="id" name="id" class="form-control" value="1"
                   readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="formcontrol"
                   value="상품A">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   value="10000">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="formcontrol"
                   value="10">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">저장
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='item.html'" type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

### 7-5. 상품 목록 - 타임리프

#### BasicItemController

* `src/main/java/hello/itemservice/web/basic/BasicItemController.java`

```java
package hello.itemservice.web.basic;

import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.annotation.PostConstruct;
import java.util.List;

@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }
}

```

컨트롤러 로직은 itemRepository에서 모든 상품을 조회한 다음에 모델에 담는다. 그리고 뷰 템플릿을 호출한다.

* `@RequiredArgsContructor`
    * `final`이 붙은 맴버변수만 사용해서 생성자를 자동으로 만들어 준다.
      ```
      public BasicItemController(ItemRepository itemRepository) {
          this.itemRepository = itemRepository;
      }
      ```
    * 이렇게 생성자가 딱 1개만 있으면 스프링이 해당 생성자에 `@Autowired`로 의존관계를 주입해준다.

    * 따라서 **final 키워드를 빼면 안된다!** 그러면 `ItemRepository`의존관계 주입이 안된다.
    * 스프링 핵심원리 - 기본편 강의 참고

* `public void init()`: 테스트용 데이터 추가
    * 테스트용 데이터가 없으면 회원 목록 기능이 정상 동작하는지 확인하기 어렵다.
    * `@PostContruct`: 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다.
    * 여기서는 간단히 테스트용 데이터를 넣기 위해서 사용했다.

#### item.html - 수정

* item.html 정적 HTML 을 뷰 템플릿(template)영역으로 복사하고 다음과 같이 수정한다.
* `src/main/resources/templates/basic/items.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container" style="max-width: 600px">
    <div class="py-5 text-center">
        <h2>상품 목록</h2>
    </div>
    <div class="row">
        <div class="col">
            <button class="btn btn-primary float-end"
                    onclick="location.href='addForm.html'"
                    th:onclick="|location.href='@{/basic/items/add}'|"
                    type="button">상품
                등록
            </button>
        </div>
    </div>
    <hr class="my-4">
    <div>
        <table class="table">
            <thead>
            <tr>
                <th>ID</th>
                <th>상품명</th>
                <th>가격</th>
                <th>수량</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${items}">
                <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}"
                       th:text="${item.id}">회원id</a></td>
                <td><a href="item.html" th:href="@{|/basic/items/${item.id}|}"
                       th:text="${item.itemName}">상품명</a></td>
                <td th:text="${item.price}">10000</td>
                <td th:text="${item.quantity}">10</td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```

#### 타임리프 간단히 알아보기

* 타임리프 사용 선언    
  `<html xmlns:th="http://www.thymeleaf.org">`

* 속성 변경 - th:href   
  `th:gref="@{/css/bootstrap.min.css}"`
    * `href="value1"`을 `th:href="value2"`의 값으로 변경한다.
    * 타임리프 뷰 템플릿을 거치게 되면 원래 값을 `th:xxx`값으로 변경한다. 만약 값이 없다면 새로 생성한다.
    * HTML을 그대로 볼 때는 `href`속성이 사용되고, 뷰 템플릿을 거치면 `th:href`의 값이 `href`로 대체되면서 동적으로 변경할 수 있다.
    * 대부분의 HTML 속성을 `th:xxx`로 변경할 수 있다.

* 타임리프 핵심
    * 핵심은 `th:xxx`가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. `th:xxx`이 없으면 기존 html의 `xxx`속성이 그대로 사용된다.
    * HTML을 파일로 직접 열었을 때, `th:xxx`가 있어도 웹 브라우저는 `th:`속성을 알지 못하므로 무시한다.
    * 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.

* URL 링크 표현식 - @{...}   
  `th:href="@{/css/bootstrap.min.css}"`
    * `@{...}`: 타임리프는 URL 링크를 사용하는 경우 `@{...}`를 사용한다. 이것을 URL 링크 표현식이라 한다.
    * URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.

상품 등록 폼으로 이동

* 속성 변경 - th:onclick
    * `onclick="locatin.href='addForm.html'"`
    * `th:onclick="|location.href='@{/basic/items/add}'|"`
      여기에는 다음에 설명하는 리터럴 대체 문법이 사용되었다.

* 리터럴 대체 - |...|    
  `|...|`: 이렇게 사용한다.
    * 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
        * `<span th:text="'Welcome to our application, ' + ${user.name} + '!'"`
    * 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
        * `<span th:text="|Welcome to our application, ${user.name}!|">`
          <br><br>
    * 결과를 다음과 같이 만들어야 하는데
        * `location.href='/basic/items/add`
    * 그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
        * `th:onclick="'locatin.href=' + '\'' + @{/basic/items/add} + '\''"`
    * 리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.
        * `th:onclick="|location.href='@{/basic/items/add}'|"`

* 반복 출력 - th:each
    * `<tr th:each="item : ${items}">`
    * 반복은 `th:each`를 사용한다. 이렇게 하면 모델에 포함된 `items`컬렉션 데이터가 `item`변수에 하나씩 포함되고, 반복문 안에서 `item` 변수를 사용할 수 있다.
    * 컬렉션의 수 만큼 `<tr> .. </tr>`이 하위 테그를 포함해서 생성된다.

* 변수 표현식 - ${...}
    * `<td th:text="${item.price}">10000</td>`
    * 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
    * 프로퍼티 접근법을 사용한다. (`item.getPrice()`)

* 내용 변경 - th:text
    * `<td th:text="${item.price}">10000</td>`
    * 내용의 값을 `th:text`의 값으로 변경한다.
    * 여기서는 10000을 `${item.price}`의 값으로 변경한다.

* `URL 링크 표현식2 - @{...}`
    * `th:href="@{/basic/items/{itemsId}(itemId=${item.id})}"`
    * 상품 ID를 선택하는 링크를 확인해보자.
    * URL 링크 표현식을 사용하면 경로를 템플릿 처럼 편리하게 사용할 수 있다.
    * 경로 변고(`{itemId}`)뿐만 아니라 쿼리 파리미터도 생성한다.
    * 예) `th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='text)}"`
        * 생성 링크: `http://localhost:8080/basic/items/1?query=test`

* URL 링크 간단히
    * `th:href="@{|/basic/items/${item.id}|}"`
    * 상품 이름을 선택하는 링크를 확인해보자.
    * 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

> 참고  
> 타임리프는 순서 HTML을 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코드와 HSML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야 한다.    
> 이렇게 순수 **HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임르프의 특징을 네츄럴 템플릿**(natural templates)이라 한다.

### 7-6. 상품 상세

#### BasicItemController - 메소드 추가

* `src/main/java/hello/itemservice/web/basic/BasicItemController.java`

```java
public class BasicItemController {
    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "basic/item";
    }
}
```

`PathVariable`로 넘어온 상품 ID로 상품을 조회하고, 모델에 담아둔다. 그리고 뷰 템플릿을 호출한다.

#### item.html - 수정

* 정적 HTML을 뷰 템플릿(template)영역으로 복사하고 다음과 같이 수정한다.
* `src/main/resources/templates/basic/item.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style> .container {
 max-width: 560px;
 }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 상세</h2>
    </div>
    <div>
        <label for="itemId">상품 ID</label>
        <input type="text" id="itemId" name="itemId" class="form-control"
               value="1" th:value="${item.id}" readonly>
    </div>
    <div>
        <label for="itemName">상품명</label>
        <input type="text" id="itemName" name="itemName" class="form-control"
               value="상품A" th:value="${item.itemName}" readonly>
    </div>
    <div>
        <label for="price">가격</label>
        <input type="text" id="price" name="price" class="form-control"
               value="10000" th:value="${item.price}" readonly>
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" id="quantity" name="quantity" class="form-control"
               value="10" th:value="${item.quantity}" readonly>
    </div>
    <hr class="my-4">
    <div class="row">
        <div class="col">
            <button class="w-100 btn btn-primary btn-lg"
                    th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|"
                    onclick="location.href='editForm.html'" type="button">상품 수정
            </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-secondary btn-lg"
                    th:onclick="|location.href='@{/basic/items}'|"
                    onclick="location.href='items.html'" type="button">목록으로
            </button>
        </div>
    </div>
</div> <!-- /container -->
</body>
</html>
```

* 속성 변경 - **th:value**  
  `th:value="${item.id}"`
    * 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력한다.(`item.getId()`)
    * `value`속성을 `th:value`속성으로 변경한다.

* 상품수정 링크
    * `th:onclick="|location.href='@{/basic/items/{itemId}/edit(itemId=${item.id})}'|`

* 목록으로 링크
    * `th:onclick="|location.href='@{/basic/items}'|"`

### 7-7. 상품 등록 폼

#### BasicItemController - 추가

* 상품 등록 폼

```java
public class BasicItemController {
    @GetMapping("/add")
    public String addForm() {
        return "basic/addForm";
    }
}
```

#### addForm.html - 상품 등록 폼 뷰

* 정적 HTML을 뷰 템플릿(templates)영역으로 복사하고 다음과 같이 수정한다.
* `src/main/resources/templates/basic/addForm.html`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
 .container {
 max-width: 560px;
 }
    </style>
</head>
<body>
<div class="container">
    <div class="py-5 text-center">
        <h2>상품 등록 폼</h2>
    </div>
    <h4 class="mb-3">상품 입력</h4>
    <form th:action
          action="item.html" method="post">
        <div><label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form-control"
                   placeholder="이름을 입력하세요">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control"
                   placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form-control"
                   placeholder="수량을 입력하세요">
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">상품
                    등록
                </button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        th:onclick="|location.href='@{/basic/items}'|"
                        onclick="location.href='items.html'" type="button">취소
                </button>
            </div>
        </div>
    </form>
</div> <!-- /container -->
</body>
</html>
```

* `<action>`
    * `<form>` 태크의 action 속성은 폼 데이터(form data)를 서버로 보낼 때 해당 데이터가 도착할 URL을 명시

* 속성 변경 - th:action
    * `th:action`
    * HTML Form에서 `action`에 값이 없으면 현재 URL에 데이터를 전송한다.
    * 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
        * 상품 등록 폼: GET`/basic/items/add`
        * 상품 등록 처리: POST`/basic/items/add`
    * 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다.

* 취소
    * 취소시 상품 목록으로 이동한다.
    * `th:onclick="|location.href='@{/basic/items}'|`

### 7-8. 상품 등록 처리 - @ModelAttribute

상품 등록 폼은 다음 방식으로 서버에 데이터를 전달한다.

* POST - HTML Form
    * `content-type: application/x-www-form-urlencoded`
    * 메시지 바디에 쿼리 파라미터 형식으로 전달 `itemName=itemA&price=1000&quantity=10`
    * 예) 회원 가입, 상품 주문, HTML Form 사용

요청 파라미터 형식을 처리해야 하므로 `@RequestParam`을 사용하자.

#### BasicItemController - addItemV1 (@RequestParam)

```java
public class BasicItemController {
    @PostMapping("/add")
    public String addItemV1(@RequestParam String itemName,
                            @RequestParam int price,
                            @RequestParam Integer quantity,
                            Model model) {

        Item item = new Item();
        item.setItemName(itemName);
        item.setPrice(price);
        item.setQuantity(quantity);

        itemRepository.save(item);

        model.addAttribute("item", item);

        return "basic/item";
    }

}
```

* 먼저 `@RequestParam String itemName`: itemName 요청 파라미터 데이터를 해당 변수에 받는다.
* `Item`객체를 생성하고 `itemRepository`를 통해서 저장한다.
* 저장된 `item`을 모델에 담아서 뷰에 전달한다.

**중요: 여기서는 상품 상세에서 사용한 `item.html` 뷰 템플릿을 그대로 재활용한다.**

#### BasicItemController - addItemV2 (@ModelAttribute)

`@RequestParam`으로 변수를 하나하나 받아서 `Item`을 생성하는 과정은 불편하다.   
이번에는 `@ModelAttribute`를 사용해서 한번에 처리해보자.

```java
public class BasicItemController {
    /**
     * @ModelAttribute("item") Item item
     * model.addAttribute("item", item); 자동 추가
     */
    @PostMapping("/add")
    public String addItemV2(@ModelAttribute("item") Item item) {
        itemRepository.save(item);

        return "basic/item";
    }
}
```

* `@ModelAttribute` - 요청 파라미터 처리
    * `@ModelAttribute`는 `Item`객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.

* `@ModelAttribute` - Model 추가
    * `@ModelAttribute`는 중요한 한가지 기능이 더 있는데, 바로 모델(Model)에 `@ModelAttribute`로 지정한 객체를 자동으로 넣어준다.
    * 지금 코드를 보면 `model.addAttribute("item", item)`을 생략했는데도 잘 동작하는 것을 확인할 수 있다.

모델에 데이터를 담을 때느 이름이 필요하다. 이름은 `@ModelAttribute`에 지정한 `name(value)`속성을 사용한다. 만약 다음과 같이 `@ModelAttribute`의 이름을 다르게 지정하면 다른
이름으로 모델에 포함된다.

`@ModelAttribute("hello) Item item` -> 이름을 `hello`로 지정    
`model.addAttribute("hello", item);` -> 모델에 `hello`이름으로 저장

> **주의**    
> 실행전에 이전 버전인 `addItemV1`에 `@PostMapping("/add")`를 꼭 주석처리 해주어야 한다. 그렇지 않으면 중복 매핑으로 오류가 발생한다.

#### ModelAttribute - addItemV3 (ModelAttribute 이름 생략)

```java
public class BasicItemController {
    @PostMapping("/add")
    public String addItemV3(@ModelAttribute Item item) {
        itemRepository.save(item);

        return "basic/item";
    }

}
```

* `@ModelAttribute`의 이름을 생략할 수 있다.

> **주의**  
> `@ModelAttribute`의 이름을 생략하면 모델에 저장될 때 클래스명을 사용한다. 이때 **클래스의 첫글자만 소문자로 변경**해서 등록한다.    
> 예) `@ModelAttribute`클래스명 -> 모델에 자동 추가되는 이름  
>        * `Item` -> `item`    
>        * `HelloWorld` -> `helloWorld`

#### ModelAttribute - addItemV4 (ModelAttribute 전체 생략)

```java
public class BasicItemController {
    @PostMapping("/add")
    public String addItemV4(Item item) {
        itemRepository.save(item);

        return "basic/item";
    }
}
```

`@ModelAttribute` 자체도 생략 가능하다. 대상 객체는 모델에 자동 등록된다. 나머지 사항은 기존과 동일하다.

## Note