# Software Design Principles

## SOLID Principles

**SOLID** is an acronym for the first five **object-oriented design** (OOD) principles by Robert C. Martin (also known as Uncle Bob). These five software development principles are guidelines to follow when building software so that it is easier to scale and maintain.

* **Single Responsibility** Principle (SRP) – A class should have only one reason to change.
* **Open/Closed** Principle (OCP) – Software entities should be open for extension but closed for modification.
* **Liskov Substitution** Principle (LSP) – Subtypes should be substitutable for their base types without altering correctness.
* **Interface Segregation** Principle (ISP) – Clients should not be forced to depend on interfaces they do not use.
* **Dependency Inversion** Principle (DIP) – High-level modules should not depend on low-level modules; both should depend on abstractions.

![](../assets/solid-principles-overview.png)

**S — Single Responsibility**
![](../assets/srp-single-responsibility.png)
***A class should have a single responsibility***

If a Class has many responsibilities, it increases the possibility of bugs because making changes to one of its responsibilities, could affect the other ones without you knowing.

*Goal: This principle aims to separate behaviours so that if bugs arise as a result of your change, it won’t affect other unrelated behaviours.*

**O — Open-Closed**
![](../assets/ocp-open-closed.png)
**Classes should be open for extension, but closed for modification**

Changing the current behaviour of a Class will affect all the systems using that Class.
If you want the Class to perform more functions, the ideal approach is to add to the functions that already exist NOT change them.

*Goal: This principle aims to extend a Class’s behaviour without changing the existing behaviour of that Class. This is to avoid causing bugs wherever the Class is being used.*

**L — Liskov Substitution**
**![](../assets/lsp-liskov-substitution.png)**

**If S is a subtype of T, then objects of type T in a program may be replaced with objects of type S without altering any of the desirable properties of that program.**

When a child Class cannot perform the same actions as its parent Class, this can cause bugs.
If you have a Class and create another Class from it, it becomes a parent and the new Class becomes a child. The child Class should be able to do everything the parent Class can do. This process is called Inheritance.
The child Class should be able to process the same requests and deliver the same result as the parent Class or it could deliver a result that is of the same type.
The picture shows that the parent Class delivers Coffee(it could be any type of coffee). It is acceptable for the child Class to deliver Cappucino because it is a specific type of Coffee, but it is NOT acceptable to deliver Water.
If the child Class doesn’t meet these requirements, it means the child Class is changed completely and violates this principle.

*Goal: This principle aims to enforce consistency so that the parent Class or its child Class can be used in the same way without any errors.*

**I — Interface Segregation**
**![](../assets/isp-interface-segregation.png)**
**Clients should not be forced to depend on methods that they do not use.**

When a Class is required to perform actions that are not useful, it is wasteful and may produce unexpected bugs if the Class does not have the ability to perform those actions.
A Class should perform only actions that are needed to fulfil its role. Any other action should be removed completely or moved somewhere else if it might be used by another Class in the future.

*Goal: This principle aims at splitting a set of actions into smaller sets so that a Class executes ONLY the set of actions it requires.*

**D — Dependency Inversion**
![](../assets/dip-dependency-inversion.png)
**High-level modules should not depend on low-level modules. Both should depend on the abstraction.**
**Abstractions should not depend on details. Details should depend on abstractions.**

Firstly, let’s define the terms used here more simply

High-level Module(or Class): Class that executes an action with a tool.
Low-level Module (or Class): The tool that is needed to execute the action
Abstraction: Represents an interface that connects the two Classes.
Details: How the tool works

This principle says a Class should not be fused with the tool it uses to execute an action. Rather, it should be fused to the interface that will allow the tool to connect to the Class.
It also says that both the Class and the interface should not know how the tool works. However, the tool needs to meet the specification of the interface.

*Goal: This principle aims at reducing the dependency of a high-level Class on the low-level Class by introducing an interface.*
