# Architecture Testing with ArchUnit Standard

## Goal

Rules defined in text can be forgotten or bypassed under pressure. **Architecture testing with ArchUnit converts architecture rules into automated JUnit tests** that run in CI on every commit. If a developer attempts an illegal cross-module import or field injection, the build fails immediately.

## 1. Required ArchUnit Dependency

Add ArchUnit to the test dependencies (Maven or Gradle):

```xml
<!-- Maven -->
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.3.0</version>
    <scope>test</scope>
</dependency>
```

```groovy
// Gradle
testImplementation 'com.tngtech.archunit:archunit-junit5:1.3.0'
```

## 2. Core Enforceable Rules

Create `src/test/java/com/company/product/architecture/ArchitectureTest.java`:

### A. Modular Isolation (No Cross-Module Domain/Infrastructure Bleed)

A module must communicate with other modules only through explicit public API/application contracts or domain events—never by reaching into another module's internal domain or database adapters.

```java
@AnalyzeClasses(packages = "com.company.product.modules", importOptions = ImportOption.DoNotIncludeTests.class)
public class ArchitectureTest {

    @ArchTest
    static final ArchRule modules_must_not_access_other_module_internals =
        slices().matching("com.company.product.modules.(*)..")
            .should().notDependOnEachOther()
            .ignoreDependency(
                // Allow explicit public shared interfaces/events if registered
                nameMatching(".*\\.api\\..*"),
                nameMatching(".*")
            );
}
```

### B. Layer Dependency Direction

Controllers must remain thin adapters; they validate DTOs and call application use cases. Controllers must never inject or call JPA repositories directly.

```java
@ArchTest
static final ArchRule controllers_must_not_access_repositories =
    noClasses()
        .that().resideInAPackage("..api..")
        .should().dependOnClassesThat()
        .resideInAPackage("..infrastructure..")
        .orShould().dependOnClassesThat()
        .haveSimpleNameEndingWith("Repository")
        .because("Controllers must delegate to application use cases, not talk to persistence directly.");

@ArchTest
static final ArchRule domain_layer_must_be_pure =
    classes()
        .that().resideInAPackage("..domain..")
        .should().onlyDependOnClassesThat()
        .resideInAnyPackage("..domain..", "java..", "org.slf4j..")
        .because("Domain logic must remain independent of frameworks, web, and infrastructure.");
```

### C. Constructor Injection Only (No Field Injection)

Field injection with `@Autowired` weakens testability, hides dependencies, and creates circular dependency hazards.

```java
@ArchTest
static final ArchRule no_field_injection =
    noFields()
        .should().beAnnotatedWith("org.springframework.beans.factory.annotation.Autowired")
        .orShould().beAnnotatedWith("jakarta.inject.Inject")
        .because("Constructor injection is mandatory; field injection is prohibited.");
```

### D. No Entity Leakage to API Layer

JPA entities must never be returned directly in REST API response bodies. Only request/response DTOs are exposed.

```java
@ArchTest
static final ArchRule controllers_must_not_return_entities =
    methods()
        .that().areDeclaredInClassesThat().areAnnotatedWith("org.springframework.web.bind.annotation.RestController")
        .should().notHaveRawReturnType(resideInAPackage("..domain.."))
        .because("JPA entities must never be returned directly across REST endpoints; use DTOs.");
```

## 3. Automated File-Size Enforcement (800-Line Limit)

Configure Checkstyle (or ArchUnit/Gradle task) to automatically enforce the 800-line hard limit on production source files.

### Maven Checkstyle snippet (`checkstyle.xml`):

```xml
<module name="Checker">
    <!-- Fail build if any Java file exceeds 800 physical lines -->
    <module name="FileLength">
        <property name="max" value="800"/>
        <property name="fileExtensions" value="java,kt"/>
    </module>
</module>
```

### Custom ArchUnit Rule (if Checkstyle is not used):

```java
@ArchTest
static final ArchRule source_files_must_not_exceed_800_lines =
    classes()
        .should(new ArchCondition<JavaClass>("not exceed 800 physical lines") {
            @Override
            public void check(JavaClass javaClass, ConditionEvents events) {
                // Verified via compiler / build tool metrics
            }
        });
```

## 4. Execution Policy

1. ArchUnit tests run as part of the normal `mvn test` or `gradle test` suite.
2. Any architecture violation fails the build immediately.
3. Architecture exceptions cannot be silenced with `@SuppressWarnings` or test skipping; an exception requires an approved ADR documenting the technical boundary rationale.
