# HD Wallet를 이용한 Aergo Store구현하기

**목차**

[TOC]

안녕하세요? 블로코 솔루션1팀에서 개발하고 있는 박성아입니다. 

이 글을 쓰게된 계기는 단순한 호기심에서 시작됐습니다. 블록체인을 공부하면서 가장 먼저 접한 책은 [비트코인, 블록체인과 금융의 혁신](https://book.naver.com/bookdb/book_detail.nhn?bid=9685493)이란 책이었는데, 이 책을 읽다보면 중간 쯤에 결정자적 지갑을 이용한 쇼핑몰 사례가 두어차례 등장합니다. 이때, 해당 페이지를 읽으면서 “나중이라도 꼭 한번 구현해보고 싶다.”는 생각이 들었습니다. 그래서 Aergo를 이용한 쇼핑몰을 구현하게 되었습니다.

# 결정자적 지갑 ?

결정자적 지갑 혹은 HD Wallet, Hierarchical Deterministic Wallert이라고 불리며, 시드(seed) 값만 가지고 있으면 여러 개의 계정을 손쉽게 생성할 수 있는 방법을 제공합니다. 

책마다 번역은 조금씩 다르지만 시드값 혹은 종자값만 있다면 항상 동일한 개인키와 공개키를 획득할 수 있습니다.

<img src="https://github.com/bitcoin/bips/blob/master/bip-0032/derivation.png?raw=true">

[^1]

이에 착안하여, 쇼핑몰 관리자는 시드값을 통해 블록체인 주소를 생성하고 이를 제품의 고유번호(기본키) 혹은 제품을 구매할 때 사용되는 계좌번호같은 역할로 사용하고자 합니다.

# 시드

결정자적 지갑은 시드로부터 동일한 개인키와 공개키를 계층적으로 생성해낼 수 있습니다. 이때, 시드란 아래와 같이 32자리의 16진수로 표현되는 값을 말합니다.

```
0C1E24E5917779D297E14D45F14E1A1A
```

그렇기 때문에 지갑 내의 모든 키를 기억할 필요없이 시드값을 기억하고 있으면 시드값으로부터 파생된 모든 개인키와 공개키를 복원할 수 있게 됩니다.

# 니모닉(Mnemonic)

니모닉 코드는 사람이 읽기 쉬운 형태의 텍스트로 표현됩니다. 이는 시드값은 사람이 기억하기 쉬운 형태가 아니기 때문에 비트코인 개발자들에 의해 제안[^2]되었습니다.

앞서 나온 개념들은 앞으로 나올 쇼핑몰 구현에 있어서 꼭 필요한 내용만을 정리하였습니다. 결정자적 지갑과 니모닉에 대해 자세한 내용이 궁금하다면 아래의 두가지 미디엄 글을 추천드립니다.



- [코드체인과 HD Wallet](https://medium.com/codechain-kr/%EC%BD%94%EB%93%9C%EC%B2%B4%EC%9D%B8%EA%B3%BC-hd-wallet-d7a9e120794f)
- [HD 지갑과 니모닉코드](https://medium.com/@devAsterisk/hd-지갑과-니모닉-mnemonic-코드-5a28cf0d4b07)

# “개념은 대충 알겠고 …,글을 쓰기위한 주제도 마련되었고..., 공부도 끝났는데...” 

산넘어 산이라 했던가? 시작은 좋았으나 현재 Aergo 개발을 지원하는 Java sdk

Heraj[^3]에서는 비트코인처럼 결정자적 지갑기능을 제공하지 않습니다. 

또한 Aergo는 UTXO기반의 블록체인이 아니기 때문에 구상부터 막막하였습니다.

## 그래도 일단은 해보자…

우선, 가장 중요한 Aergo에서 결정자적 지갑을 사용할 수 있을까? 라는 부분부터 해결해보기로 하였습니다. 

위에서 언급한대로 Heraj에서는 결정자적 지갑 기능을 제공하지않습니다. 하지만, 키 생성에 있어서 ECDSA(타원곡선함수)를 이용하는 생성방법을 제공하기 때문에 이를 이용하기로 하였습니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/ecdsa_key.png?raw=true">

결론부터 말씀드리면, 저는 이 문제를 Bitcoin core[^4]에 있는 DeterministicKey 관련로직을  활용하는 것으로써 풀어내었습니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/deterministic_key.png?raw=true">

해당 패키지에서 제공하는 DeterministicKey를 이용하여 ECDSAKey를 생성하였고,

생성한 ECDSAKey를 통해 다시 AergoKey를 생성하는데 성공하였습니다. 

결국 블록체인 주소라는 것은 타원곡선함수를 통해 생성된 대칭되는 키쌍이기 때문에 이러한 방식을 통해서 Aergo에서 사용하는 형태의 공개키와 기본키를 생성하였습니다. 



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/derive.png?raw=true">



# 구현

쇼핑몰을 구현하는 데는 스프링부트를 이용하였습니다. 단, 기존에 자주 애용하던 maven은 사용하지않고 평소에 초기세팅부터 구현해보고싶었던 gradle을 활용하기로 하였습니다. gradle에는 multi project라는 개념이 있는데 Django로 치면 프로젝트 하나의 여러개의 App을 생성해서 관리하는 것과 비슷합니다. 

우선 이클립스에서 새로운 gradle 프로젝트를 생성합니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/new_gradle_project.png?raw=true">

생성이 완료되면 다음과 같이 settings.gradle을 작성하고, 설정 내용으로는 

루트 프로젝트의 이름과 구성할 서브 프로젝트들의 이름을 입력합니다.

 



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/setting.gradle.png?raw=true">

다음으로 해야할 일은 build.gradle을 작성하는 일입니다.

```
group = 'io.blocko'

ext {
  springBootVersion = '2.0.0.RELEASE'
  herajVersion = '1.1.0'
}

buildscript { 
  repositories {
    maven {
      url "https://plugins.gradle.org/m2/"
    }
    google()
    mavenCentral()
  }
  dependencies {
    classpath "io.spring.gradle:dependency-management-plugin:1.0.8.RELEASE"
  }
}

allprojects {
  version '0.1-SNAPSHOT'
}

def javaProjects = [project('aergo-store-common'), project('aergo-store-web')]

configure(javaProjects) {
  apply plugin: 'java'
  apply plugin: 'maven'
  apply plugin: 'idea'
  apply plugin: 'checkstyle'
  apply plugin: 'eclipse'
  
  sourceCompatibility = JavaVersion.VERSION_1_8
  targetCompatibility = JavaVersion.VERSION_1_8
  
  eclipseJdt.doLast( {
    ant.propertyfile(file: ".settings/org.eclipse.core.resources.prefs") {
      ant.entry(key: "eclipse.preferences.version", value: "1")
      ant.entry(key: "encoding/<project>", value: "utf8")
    }
  } )
  
  idea {
    module {
      outputDir file('build/classes/java/main')
      testOutputDir file('build/classes/java/test')
    }
  }
  
  compileJava.options.encoding = 'UTF-8'
  compileTestJava.options.encoding = 'UTF-8'
  
  repositories {
    jcenter()
    mavenLocal()
    mavenCentral()
  }
  
  dependencies {
    compileOnly "org.projectlombok:lombok:1.18.8"
    testCompile "org.projectlombok:lombok:1.18.8"
  }	
  
  checkstyle {
    project.ext.checkstyleVersion = '8.24'
    project.ext.sevntuChecksVersion = '1.26.0'

    ignoreFailures = true
    configFile = file("${rootProject.projectDir}/styles.xml")
    reportsDir = file("${buildDir}/checkstyle-reports")
    configurations {
      checkstyle
    }
    checkstyleMain {
      source = sourceSets.main.allSource
      reportsDir = checkstyle.reportsDir
    }
    dependencies{
      assert project.hasProperty("checkstyleVersion")

      checkstyle "com.puppycrawl.tools:checkstyle:${checkstyleVersion}"
      checkstyle "com.github.sevntu-checkstyle:sevntu-checks:${sevntuChecksVersion}"
    }
  }
  
  task sourcesJar(type: Jar, dependsOn: classes) {
    classifier = 'sources'
    from sourceSets.main.allSource
  }
  
  jar {
    baseName = 'store'
  }

}
```

약간 다른 얘길하자면 개인적으로는 아직까지도 build.gradle을 작성하는 법에 대해서 best practice를 잘 모르겠습니다. 몇가지 살펴봐도 작성자마다 조금씩의 차이가 존재하는 것 같습니다.

이제 설정이 끝났으면, 프로젝트를 선택한 후  Refresh gradle project을 클릭합니다.

위에서 설정한 settings.gradle과 build.gradle에 의해 자동으로 프로젝트들이 구성되어 집니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/package.png?raw=true">



# @PrePersist

위에서 가장 큰 문제였던 결정자적 지갑문제를 해결하였습니다. 이제 이를 응용하여 인스턴스를 생성할 때 uuid로 Aergo Address를 사용해보도록 하겠습니다. 

javax.persistence 패키지를 살펴보면 PrePersist라는 어노테이션이 있습니다.

이는 persist()메서드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출됩니다. 

이를 이용해서 인스턴스를 DB에 저장하기 전, 인스턴스에 uuid를 부여하도록 합니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/prepersist.png?raw=true">

DB상에 저장되어지는 Entity들은 공통적으로 기본키를 가지게 됩니다. 이를 UuidEntity라는 Abstact class를 생성하여 모든 Entity들이 상속받게함으로써 해결합니다.

그림에 보이는 childKey()는 사전에 결정자적 지갑에서 추출한 AergoKey입니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/HDWallet.png?raw=true">

그림과 같이 처음 프로젝트 시작 시 , 위의 설정한 mnemonic code와 passphrase를 통해 

시드값을 통해 생성되는 자식키들을 인스턴스들은 uuid로써 사용하게 됩니다.

# @SessionAttribute

이번 쇼핑몰을 구현하면서 가장 많이 사용한건 세션에 관련된 내용이었습니다. 

로그인부터 제품에 대한 내용까지 대부분을 세션을 이용해서 처리하였습니다.

Spring boot security를 통해 더 디테일하게 구현하고 싶었지만 개발기간이 여의치않아서  단순히 구현하려다보니 HttpSession, sessionAttribute을 통해 처리한 부분이 상당히 많았던 것 같습니다.

# Frontend

이번에 프론트엔드를 개발하면서 고민이 상당히 많았는데 

예전에 쓰던 기존의 Thymeleaf 나 Freemarker와 같은 view resolver를 쓸 것이냐? 

최신 트랜드인 Vue나 React를 사용할 것이냐? 혹 서버 사이드 렌더링방식을 채택할 것이냐? 클라이언트 사이드 렌더링 방식을 채택할 것이냐 ? 등등 여러가지를 비교해봤지만

역시나 결국은 개발 시간을 단축하기위해 Thymeleaf를 사용하게 되었습니다.

대부분 html, javascript, jQuery, Ajax와 같은 기술을 사용하였습니다. 

# Template

개인적으로 MVP처럼 빠른 결과물이 필요할때에는 무료템플릿을 많이 애용합니다.



- [https://themewagon.com/](https://themewagon.com/)
- [https://html5up.net/](https://html5up.net/)

직접 구현한 것이 아니다보니 세세한 css나 레이아웃 등을 변경하기에는 시간이 오히려 더 걸릴 수도 있지만 가벼운 결과물이 필요한 경우, 무료 템플릿을 이용하는 방법도 추천합니다.

이번 쇼핑몰 구현을 위해서 사용된 템플릿은 [eShopper](https://themehunt.com/item/1524993-eshopper-free-ecommerce-html-template)이라는 무료 템플릿을 사용하였습니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/template.png?raw=true">



# Icon

프론트엔드 개발자분들이면 많이 아실수도 있지만 저는 개인적으로 많은 도움을 받은 fontawesome 입니다. (완소 덩어리들 …)





<img src="https://github.com/id6827/id6827.github.io/blob/master/images/icon.png?raw=true">



# Layout

이번 프로젝트에서는 View resolver로써 Thymleaf를 채택하였습니다.

그전까지는 SPA형식으로 개발하다보니 Layout을 사용할 기회가 없었는데, 이번 기회에는 Layout을 이용하여 구현을 시도하였습니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/layout.png?raw=true">

그림에서 보이는 바와 같이 해당 layout.html은 모든 html에 기본 base가 되고, 새로이 구성하는 html의 경우 layout:fragment:header영역과 layout:fragment:section만을 구현하면 사용 가능합니다. 본 프로젝트에서는 다음과 같은 디렉터리를 가지게 됩니다.





<img src="https://github.com/id6827/id6827.github.io/blob/master/images/layout2.png?raw=true">



# 서버 사이드 렌더링

다음의 그림은 메인 페이지입니다. 현재 미리 등록해둔 3가지의 product들이 존재하며, 이를 ModelAndView 객체에 담아서 뿌려주고 있습니다. 



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/main.png?raw=true">

소스상으로 살펴보면 aergo/store/controller/HomeController에서 @GetMapping(“index”) 에서 index페이지에 articles를 세션에 추가하여 반환하는 모습을 살펴볼  수 있습니다.

```
  @GetMapping("index")
  public ModelAndView home() {
    final ModelAndView mav = new ModelAndView("index");
    final Pageable pageable = PageRequest.of(0, 5);
    final List<Article> shirts = articleService.findByAll(pageable).getContent();
    mav.addObject("articles", shirts);

    return mav;
  }
```

로그인의 경우에도 서버사이드 렌더링을 통해 화면을 그려주고 있으며, 세션을 활용하여 로그인한 사용자의 정보를 저장합니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/login.png?raw=true">



```
  @PostMapping("login")
  public ModelAndView login(final User user, HttpServletResponse response) throws Throwable {
    log.info("Login user : {}", user);
    return userService.login(user).map(u -> new ModelAndView("index", "user", u))
        .orElse(new ModelAndView("index", "user", null));
  }
```

로그인이 성공하면 메인화면으로 이동하게 되고, Thymleaf에서 설정해뒀던 th:if 문에 의해

숨겨진 메뉴가 보여지게 됩니다.





<img src="https://github.com/id6827/id6827.github.io/blob/master/images/th_if.png?raw=true">



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/main2.png?raw=true">

해당 페이지에서 “Add to cart” 버튼을 누르게되면 해당 물건이 cart에 담기며, cart페이지로 이동 시 다음과 같은 화면을 보실 수 있습니다.



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/cart.png?raw=true">

해당 페이지는 jQuery + Ajax를 이용하여, 서버로 rest API를 호출하여 해당 유저가 선택한 물건들의 목록을 출력하게 합니다.

각각의 버튼에 이벤트를 걸어두어서 실시간으로 동작하게 되어있고, Buy버튼을 클릭하면



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/trading.png?raw=true">

Trading 페이지로 이동하게 됩니다. 우편번호찾기의 경우, Daum에서 제공하는 API를 사용하여 손쉽게 구현할 수 있었습니다. 



<img src="https://github.com/id6827/id6827.github.io/blob/master/images/search.png?raw=true">





<img src="https://github.com/id6827/id6827.github.io/blob/master/images/daum_api.png?raw=true">



# 마치며 ...

현재 개발 상태는 보시는 바와 같이 여기까지 구현되어 있습니다. 앞으로 추가할 사항은 

결제시스템을 완료해서 Aergo Test.net에 연결하는 것입니다. 

현재 테스트넷의 경우 [https://faucet.aergoscan.io/](https://faucet.aergoscan.io/)에 요청시 5 Aergo가 해당 Address로 들어오게 되어있는데,  이를 이용하여  크롬 확장프로그램인 [Aergo Connect](https://chrome.google.com/webstore/detail/aergo-connect/iopigoikekfcpcapjlkcdlokheickhpc?hl=ko)을 통해 실제로 구현한 서버에 제품을 구입하는 내용이 될 예정입니다.

추가로 최종 목표는 Thymelaf도 걷어내고 React로 변경하는 것이며, rest API에서 Graphql로 변경하는 것입니다.



전체소스는 https://github.com/id6827/aergo-store를 참고하시면 됩니다.

<!-- Footnotes themselves at the bottom. -->

## Notes

[^1]: 

```
 [https://github.com/bitcoin/bips/blob/master/bip-0032/derivation.png?raw=true](https://github.com/bitcoin/bips/blob/master/bip-0032/derivation.png?raw=true)
```

[^2]: 

```
 BIP-0039 - [https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
```

[^3]: 

```
 https://github.com/aergoio/heraj
```

[^4]: 

```
 https://mvnrepository.com/artifact/org.bitcoinj/bitcoinj-core
```

<!-- Docs to Markdown version 1.0β17 -->
