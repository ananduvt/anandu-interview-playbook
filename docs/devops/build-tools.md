# Build Tools


## Gradle

**REF** : [Part 1: Initializing the Project](https://docs.gradle.org/current/userguide/part1_gradle_init.html)

## Maven

## Gradle vs Maven

| Basis | Gradle | Maven |
| :---- | :---- | :---- |
| Based on | Gradle is based on developing domain-specific language projects. | Maven is based on developing pure Java language-based software. |
| Configuration | It uses a Groovy-based Domain-specific language (DSL) for creating project structure. | It uses Extensible Markup Language (XML) for creating project structure. |
| Focuses on | Developing applications by adding new features to them. | Developing applications in a given time limit. |
| Performance | It performs better than maven as it is optimized for tracking only current running tasks. | It does not create local temporary files during software creation and is hence – slower. |
| Java Compilation | It avoids compilation. | It is necessary to compile. |
| Usability | It is a new tool, which requires users to spend a lot of time to get used to it. | This tool is a known tool for many users and is easily available. |
| Customization | This tool is highly customizable as it supports a variety of IDE’s. | This tool serves a limited number of developers and is not that customizable. |
| Languages supported | It supports software development in Java, C, C++, and Groovy. | It supports software development in Java, Scala, C#, and Ruby and it does not natively support C and C++ but can support through plugins like “maven-native-plugin” or we can integrate other build systems like CMake or Makefile. |
| Project Configuration | For declaring the project configuration, it does not use the XML files. | For declaring the project configuration, it uses the XML files. |
| Based on | Graph of task dependencies that do the work. | On the phases of the fixed and linear model. |
| Goal | To add functionality in the project is the main goal of the Gradle. | To finish the project in the given timeline is the main goal of the Maven. |
