# Build Tools: Maven & Gradle

Build automation tools for Java projects: dependency management, compilation, testing, packaging, and deployment.

---

## Maven

Maven is a declarative, XML-based build tool that follows a standard project structure and lifecycle.

### Project Structure

```
my-app/
├── pom.xml                 # Project configuration (Model Object)
├── src/
│   ├── main/
│   │   ├── java/          # Source code
│   │   └── resources/     # Config files, properties
│   └── test/
│       ├── java/          # Test code
│       └── resources/     # Test config
└── target/                # Build output (compiled classes, JARs)
```

### POM (Project Object Model)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project identification -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>My Application</name>
    
    <!-- Parent POM (inherit properties, plugins, dependencies) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
    </parent>
    
    <!-- Properties for reuse -->
    <properties>
        <java.version>21</java.version>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <!-- Compile scope (included in final JAR) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Test scope (only for testing) -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Runtime scope (not needed at compile time) -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Provided scope (container provides it) -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    
    <!-- Build plugins -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            
            <!-- Surefire: Run tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### Maven Lifecycle & Phases

**Build Lifecycle:** Sequence of phases that Maven executes

```
clean → validate → compile → test → package → verify → install → deploy
```

**Common Phases:**
- `clean`: Delete target directory
- `validate`: Validate project structure
- `compile`: Compile source code → target/classes/
- `test`: Run unit tests (JUnit, TestNG)
- `package`: Create JAR/WAR → target/my-app-1.0.0.jar
- `verify`: Run integration tests
- `install`: Install JAR to local repository (~/.m2/repository/)
- `deploy`: Upload to remote repository (Nexus, Artifactory)

**Commands:**
```bash
mvn clean                    # Delete target
mvn compile                  # Compile only
mvn test                     # Compile + run tests
mvn package                  # Compile + test + package JAR
mvn install                  # Compile + test + package + install locally
mvn deploy                   # Push to remote repo
mvn clean install            # Clean + compile + test + install
mvn clean install -DskipTests # Skip tests (faster for CI/CD)
```

### Dependency Management

```xml
<!-- Dependency with version override -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2022.0.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Exclude transitive dependencies -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## Gradle

Gradle is a declarative-imperative build tool using Groovy/Kotlin DSL with faster incremental builds.

### Project Structure

```
my-app/
├── build.gradle or build.gradle.kts  # Build configuration
├── settings.gradle                   # Multi-module projects
├── gradle.properties                 # Version definitions
├── src/
│   ├── main/
│   │   ├── java/                     # Source code
│   │   └── resources/                # Config
│   └── test/
│       ├── java/                     # Tests
│       └── resources/                # Test config
└── build/                            # Output
```

### Build Configuration (Groovy DSL)

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '21'

// Properties for reuse
ext {
    springCloudVersion = '2022.0.1'
}

// Dependency repositories
repositories {
    mavenCentral()          // Maven Central (public)
    maven {
        url 'https://repo.example.com/nexus/repository/releases/'
        credentials {
            username 'user'
            password 'pass'
        }
    }
}

// Dependencies
dependencies {
    // Compile scope (default)
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // Test scope
    testImplementation 'org.junit.jupiter:junit-jupiter'
    
    // Runtime only
    runtimeOnly 'com.mysql:mysql-connector-java'
    
    // Annotation processor (compile time)
    annotationProcessor 'org.projectlombok:lombok'
}

// Dependency management
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

// Tasks
tasks.named('test') {
    useJUnitPlatform()
}

// Custom task
task hello {
    doLast {
        println 'Hello from Gradle!'
    }
}
```

### Gradle Build Lifecycle

**Task Execution:** Directed acyclic graph (DAG) of task dependencies

```
build (default task)
  ├── compileJava
  ├── compileTestJava
  ├── processResources
  ├── test
  └── jar

$ gradle build          # Compile + test + package
$ gradle clean          # Delete build directory
$ gradle test           # Run tests only
$ gradle jar            # Create JAR
$ gradle compileJava    # Compile only
$ gradle --info build   # Verbose output
$ gradle build -x test  # Build excluding tests
$ gradle tasks          # List all tasks
```

### Gradle vs Maven Commands

| Task | Maven | Gradle |
|------|-------|--------|
| Compile | `mvn compile` | `gradle compileJava` |
| Test | `mvn test` | `gradle test` |
| Package | `mvn package` | `gradle jar` |
| Build | `mvn clean install` | `gradle build` |
| Skip tests | `mvn install -DskipTests` | `gradle build -x test` |
| List dependencies | `mvn dependency:tree` | `gradle dependencies` |
| Update dependencies | `mvn versions:display-dependency-updates` | `gradle dependencyUpdates` |

---

## Maven vs Gradle Comparison

| Aspect | Maven | Gradle |
|--------|-------|--------|
| **Configuration** | XML (declarative) | Groovy/Kotlin DSL (flexible) |
| **Learning Curve** | Moderate (structured) | Steep (flexible = complex) |
| **Build Speed** | Slower (no caching) | Faster (incremental builds, caching) |
| **Customization** | Limited (fixed lifecycle) | Highly customizable (custom tasks) |
| **Dependency Management** | Transitive (works well) | Transitive (same) |
| **Multi-module** | Complex XML nesting | Cleaner with settings.gradle |
| **IDE Support** | Excellent (all IDEs) | Good (IntelliJ, Gradle native) |
| **Industry Adoption** | ⭐⭐⭐ (ubiquitous) | ⭐⭐ (growing, Android default) |
| **When to Use** | Standard enterprise projects | Complex builds, Android, performance-critical |
| **Spring Boot Projects** | Common | Increasingly popular |

---

## Plugins & Extensions

### Maven Plugins

```xml
<build>
    <plugins>
        <!-- Compiler: Set Java version -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>21</source>
                <target>21</target>
            </configuration>
        </plugin>
        
        <!-- Surefire: Run tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
        </plugin>
        
        <!-- Jar: Create executable JAR -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.App</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
        
        <!-- Assembly: Package with dependencies -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### Gradle Plugins

```gradle
plugins {
    id 'java'                               // Java compilation
    id 'org.springframework.boot' version '3.1.0'  // Spring Boot
    id 'io.spring.dependency-management'   // Dependency management
    id 'com.google.protobuf' version '0.9.0'      // Protocol Buffers
}

// Apply plugin
apply plugin: 'jacoco'  // Code coverage

// Configure plugin
jacoco {
    toolVersion = "0.8.8"
}
```

---

## Dependency Scopes

### Maven Scopes

| Scope | Included in JAR | Used for |
|-------|-----------------|----------|
| `compile` | ✅ Yes | Source code compilation |
| `test` | ❌ No | Test code only |
| `runtime` | ✅ Yes | Runtime execution (not compile) |
| `provided` | ❌ No | Container provides (servlet-api) |
| `import` | ❌ No | Import another POM's dependencies |

### Gradle Configurations

| Config | Included in JAR | Used for |
|--------|-----------------|----------|
| `implementation` | ✅ Yes | Source code (hides from consumers) |
| `testImplementation` | ❌ No | Test code only |
| `runtimeOnly` | ✅ Yes | Runtime (not compile) |
| `compileOnly` | ❌ No | Compile only (container provides) |
| `annotationProcessor` | ❌ No | Annotation processing (@Data, @Builder) |

---

## Practical Examples

### Maven: Spring Boot Application

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app \
    -DarchetypeArtifactId=maven-archetype-quickstart
cd my-app
mvn spring-boot:run        # Run Spring Boot
mvn clean install          # Build and install
mvn deploy                 # Deploy to Nexus/Artifactory
```

### Gradle: Spring Boot Application

```bash
gradle init --type java-application --test-framework junit-jupiter
cd my-app
gradle bootRun             # Run Spring Boot
gradle build               # Build
gradle publish             # Publish to repository
```

---

## Common Issues & Solutions

| Issue | Maven | Gradle |
|-------|-------|--------|
| **Dependency conflict** | Use `<exclusion>` or `<version>` in POM | Use `force` or constraint in build.gradle |
| **Slow builds** | Use `-o` offline flag | Enable build cache: `org.gradle.caching=true` |
| **Test failures** | `mvn test -DskipFailingTests` | `gradle test --continue` |
| **Clean build** | `mvn clean install` | `gradle clean build` |
| **Check updates** | `mvn versions:display-updates` | `gradle dependencyUpdates` |

---

## Interview Talking Points

- **Maven vs Gradle?** Maven is declarative (XML), stable, standard; Gradle is flexible (DSL), faster builds, custom tasks
- **When to use Maven?** Enterprise projects, team requires standardization, simple builds
- **When to use Gradle?** Performance-critical builds, complex customization, Android development
- **Dependency management?** Transitive dependencies; Maven has more conservative approach; Gradle allows override
- **Build lifecycle?** Maven: fixed phases (clean → validate → compile → test → package); Gradle: task DAG (directed acyclic graph)
- **Plugin usage?** Maven: compiler, surefire, assembly; Gradle: spring-boot, protobuf, custom task definitions
- **Incremental builds?** Maven: full rebuilds; Gradle: only affected tasks (faster)
- **Multi-module projects?** Maven: complex XML; Gradle: cleaner with settings.gradle

---

**See also:** [CI/CD](ci-cd.md), [DevOps Overview](index.md)
