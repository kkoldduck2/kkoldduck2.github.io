---
layout: single
title:  "MyBatis와 MyBatis-Spring "
categories: spring
tag: [web, mybatis, spring]
toc: true
author_profile: true
---
## 서론

이번에 오픈소스 컨트리 뷰션에서 제플린 프로젝트를 spring boot로 바꾸는 작업에 참여하게 되었다. 우선 spring boot dependency를 추가해야하는데 빌드 중에 엄청난 dependency 오류가 발생했다. 

![image](https://user-images.githubusercontent.com/47748246/191922672-91288280-e2a3-4a29-8508-69ff220d0610.png)

여기서 충돌난 모든 dependencies들을 처리해줘야하는데 그에 앞서 메이븐 document를 보고 메이븐의 의존성 관리에 대해 정리해봤다. 

## Transitive Dependencies

- 전사적 의존성..?
- 메이븐을 사용하면 내 프로젝트가 필요로 하는 라이브러리들을 직접 찾거나 정의할 필요가 없다.  메이븐이 관련된 모든 dependency들을 자동으로 포함시키기 때문
- 이러한 특성 때문에 dependencies 그래프는 점점 커지게 된다. 
→ 이로인해 어떤 dependency가 추가되어야하는지에 대해 **제한**하는 몇몇 특성들이 존재한다.
    1. **`Dependency Mediation`**
        - 같은 artifact에 대한 서로 다른 버전의 dependency가 dependencies 그래프에 존재할 때, 어느 버전을 선택할지에 대한 규칙이다.
        - 메이븐은 이 경우 더 가까운 버전을 선택한다. 아래 그림을 예로 들면,
        
       ![image](https://user-images.githubusercontent.com/47748246/191922716-cf8004c6-c4b9-4c19-8fd6-bcc1bace9f3b.png)
        
        - 이런 형태의 의존성 그래프의 경우, 메이븐은 D artifact 버전으로 1.0을 선택할 것이다. 왜냐하면 A → B → C → D 2.0 보다 A→E→D1.0이 더 가깝기 때문이다.
        - 만약 D1.0이 아니라 D 2.0을 선택하고 싶으면 아래와 같이 D2.0 의존성을 A에 추가해주면 된다.
        
       ![image](https://user-images.githubusercontent.com/47748246/191922748-71113565-89c6-4ee0-a4a9-653614486fed.png)
        

1. **`Dependency Management`**
    - 앞선 예에서 보다시피 A 프로젝트는 D를 직접적으로 사용하지 않음에도 자신의 Dependency Management에 D2.0을 명시함으로써, 다른 B, C, E 프로젝트가 사용하는 D의 버전을 2.0으로 고정시킨다.
    - 이처럼 프로젝트(A) 작성자가 aritifact의 버전을 직접적으로 명시한 것이 다른 의존성 프로젝의 해당 artifact에 대한 버전을 대체하는 것을 말한다.

1. **`Dependency scope`**
    - 현재 빌드 단계에 적합한 의존성들만을 포함시킬 수 있도록 하는 것. 하단에서 자세히 설명
    
2. **`Excluded dependencies`**
    - 만약 프로젝트 X가 Y를 포함하고, Y가  Z를 포함한다고 할 때, X는 `exclusion` element를 이용해서 Z를 포함하지 않도록 할 수 있다.
    
3. **`Optional dependencies`**
    - 만약 프로젝트 Y가 Z를 포함하고 있을 때, 프로젝트 Y의 소유자가 `optional` element를 이용하여 Z를 optional dependency로 표시할 수 있다. 
    → 프로젝트 X가 Y를 의존성으로 포함할때, X는 오직 Y에만 의존하고 Y의 optional dependency인 Z는 포함하지 않는다. 
    → 프로젝트 X의 소유자는 명시적으로 Z를 의존성에 포함할 수 있다. (optional)
    - 쉽게 말해서 optional dependency는 “`excluded by default`"라고 이해하면 된다.

## Dependency Scope

- Dependency Scope는 앞서 설명했듯이 transitivitiy of a dependency를 제한하기 위해, 그리고 클래스 패스에 해당 dependency가 언제 포함되어야 하는지를 정의하기 위해 사용된다.
- 총 6가지 scope가 존재한다.
- **`compile`**
    - default scope : scope를 생략하면 적용됨
    - compile 의존 관계에 있는 것은 프로젝트의 모든 클래에서 사용 가능하다.
    - 이 scope의 dependency는 dependent project에 전파된다.
- **`provided`**
    - compile이랑 비슷하지만 해당 의존성을 runtime에 제공하기 위해서는 JDK나 컨테이너가 필요하다는 것을 뜻한다.
    - 이 depencency는 runtime이 아닌 컴파일과 테스트를 위해 classpath에 추가된다. 또한 trasitive하지 않다.
    - 패키징 시 포함되지 않는다.
- **`runtime`**
    - 이 scope는 해당 dependency가 컴파일 클래스패스에 필요하지 않고, 오직 실행 시와 테스트 클래스 패스에 필요함을 뜻한다.
- **`test`**
    - 이 scope는 해당 dependency가 애플리케이션 사용에 필요하지 않고, 테스트 컴파일과 실행에만 필요함을 의미한다.
- **`system`**
    - provided과 유사하지만 반드시 해당 JAR를 포함해야 한다는 것을 뜻한다.
    
    ```xml
    <dependency>
      <groupId>com.baelung</groupId>
      <artifactId>custom-dependency</artifactId>
      <version>1.3.2</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/libs/custom-dependency-1.3.2.jar</systemPath>
    </dependency>
    ```
    

- **`import`**
    - 이 scope는 `<dependencyManagement>` section에 있는 `pom` 타입의 dependency에서만 사용이 가능하다.
    - 다른 POM들을 우리 프로젝트의 dependencyManagement section에 import 시킨다는 뜻
    - <dependencyManagement> section에 있는 dependency list가 교체될 수 있음을 명시한다.
    
    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>other.pom.group.id</groupId>
                <artifactId>other-pom-artifact-id</artifactId>
                <version>SNAPSHOT</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>   
        </dependencies>
    </dependencyManagement>
    ```
    
    - 위 코드가 뜻하는 것은 `other-pom-artifact-id`의 dependencyManagement 영역에 정의된 dependencies가 우리 POM의 dependencyManagement에 포함됨을 의미한다.
    - 따라서 우리는 이러한 dependencies를 우리 POM의 dependency 영역에서 버전 명시 없이 reference가 가능하다.
    - 하지만 만약 우리 프로젝트이 POM에서 `other-pom-artifact-id` 에 대한 dependency를 정의하면 이것이 우리 프로젝트에 transitive하게 포함되기 때문에 → 이는 결국 dependencyManagement 영역의 해당 artifact-id를 대체한다.
        
        [https://stackoverflow.com/questions/11778276/what-is-the-difference-between-pom-type-dependency-with-scope-import-and-wit](https://stackoverflow.com/questions/11778276/what-is-the-difference-between-pom-type-dependency-with-scope-import-and-wit)
        

## Dependency Management

- dependency 정보를 중앙화(centralizing)하기 위함이다.
- 만약 내가 갖고 있는 프로젝트들이 공통된 부모를 상속받고 있다면, 공통된 의존성들을 모두 부모의 POM에 포함시키고 자식 POM들은 해당 의존성의 artifact에 대한 간단한 reference만 갖고 있으면 된다.
- 예
    - 부모 POM의 종속성 버전 정의
        
        ```xml
        <project>
        	<modelVersion>4.0.0</modelVersion>
        	<groupId>kr.co.vicki</groupId>
        	<artifactId>a-parent</artifactId>
        	<version>0.0.1-SNAPSHOT</version>
        	<packaging>pom</packaging>
        
        	<properties>
        		<spring.data.version>[1.1.0, ]</spring.data.version>
        	</properties>
        
        	<dependencyManagement>
        		<dependencies>
        			<dependency>
        				<groupId>org.springframework.data</groupId>
        				<artifactId>spring-data-jpa</artifactId>
        				<version>${spring.data.version}</version>
        			</dependency>
        		</dependencies>
        	...
        	</dependencyManagement>
        	...
        </project>
        ```
        
    
    - 자식 프로젝트 POM
        
        ```xml
        <project>
        	<modelVersion>4.0.0</modelVersion>
        	<parent>
        		<groupId>kr.co.vicki</groupId>
        		<artifactId>project-a</artifactId>
        		<version>0.0.1-SNAPSHOT</version>
        	</parent>
        	<version>${parent.version}</version>
        	<packaging>jar</packaging>
        	...
        	<dependencies>
        		<dependency>
        			<groupId>org.springframework.data</groupId>
        			<artifactId>spring-data-jpa</artifactId>
        		</dependency>
        	</dependencies>
        	...
        </project>
        ```
        
    - 즉,  자식 프로젝트에 버전을 명시하지 않아도 부모 POM <dependencyManagement> 엘리먼트에서 정의되었기 때문에, 자식 프로젝트의 spring-data-jpa 디펜던시의 버전 정보가 전달된다.
    - 결론적으로 <dependencyManagement> 엘리먼트는 버전 정보를 지정하지 않아도 어느 하위 프로젝트라도 디펜던시를 정의할 수 있도록 해주는 환경변수와 같다.
    - 여러 프로젝트에서 공통으로 사용하는 디펜던시라면 부모 POM에 정의해서 버전관리를 하는 것이 좋다.
- dependencyManagement는 transitive dependencies에서 aritifacts의 버전 컨트롤에도 유용하게 사용된다.
    - 예를 들어 프로젝트 A는 dependencyManagement 영역에 다음과 같은 artifact들을 포함한다.
        - a ver1.2 / b ver1.0, scope=compile / c ver 1.0, scope=compile / d ver 1.2
    - 그리고 프로젝트 B는 A를 부모로 갖고, 다음과 같은 POM 형태를 갖고 있다.
        - dependencyManagement : d ver1.0
        - dependencies : d ver 1.0, scope=runtime / a ver 1.0, scope=runtime / c, scope=runtime
    - 만약 maven이 프로젝트 B에서 동작할 때,

- 한가지 더, Dependency Management의 또 다른 중요한 포인트 중 하나는 transitive dependencies에서 사용된 artifacts들의 버전을 컨트롤하는 것이다.

```xml
<!-- project A-->
<project>
 <modelVersion>4.0.0</modelVersion>
 <groupId>maven</groupId>
 <artifactId>A</artifactId>
 <packaging>pom</packaging>
 <name>A</name>
 <version>1.0</version>
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>test</groupId>
       <artifactId>a</artifactId>
       <version>1.2</version>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>b</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>c</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>d</artifactId>
       <version>1.2</version>
     </dependency>
   </dependencies>
 </dependencyManagement>
</project>
```

```xml
<!-- project B-->
<project>
  <parent>
    <artifactId>A</artifactId>
    <groupId>maven</groupId>
    <version>1.0</version>
  </parent>

  <modelVersion>4.0.0</modelVersion>
  <groupId>maven</groupId>
  <artifactId>B</artifactId>
  <packaging>pom</packaging>
  <name>B</name>
  <version>1.0</version>
 
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>test</groupId>
        <artifactId>d</artifactId>
        <version>1.0</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
 
  <dependencies>
    <dependency>
      <groupId>test</groupId>
      <artifactId>a</artifactId>
      <version>1.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>test</groupId>
      <artifactId>c</artifactId>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

### Importing Dependencies

- 앞서 상속을 이용한 의존성 관리 방법을 알아보았다. 하지만 큰 프로젝트에서는 이 방식으로 의존성을 관리하는게 쉽지 않다.
- 그 이유는 하나의 프로젝트는 오직 하나의 부모만을 가질 수 있기 때문이다.
- 따라서 프로젝트는 다른 프로젝트의 managed dependency를 **import**할 수 있다.
- 이는 POM artifact를 `import` scope를 가진 의존성으로 정의함으로서 가능하다.

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>maven</groupId>
  <artifactId>B</artifactId>
  <packaging>pom</packaging>
  <name>B</name>
  <version>1.0</version>
 
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>maven</groupId>
        <artifactId>A</artifactId>
        <version>1.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>test</groupId>
        <artifactId>d</artifactId>
        <version>1.0</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
 
  <dependencies>
    <dependency>
      <groupId>test</groupId>
      <artifactId>a</artifactId>
      <version>1.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>test</groupId>
      <artifactId>c</artifactId>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

- A의 managed dependencies들 (다시 말해 <dependencyManagement> 안에 정의된 의존성들)이 모두 B에 병합될 것이다.
- 단, d는 현재 POM에 정의되어 있으므로 제외됨

  
[https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
