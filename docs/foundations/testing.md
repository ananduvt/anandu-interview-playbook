# Testing / QA

## Test pyramid
- **Unit** (most) — fast, isolated, mock dependencies.
- **Integration** — components together (DB, external services / virtualization).
- **End-to-end** (fewest) — full system through the UI/API; slow, brittle.

## Black box vs white box

1. **Black Box Testing**
   The technique of testing in which the tester doesn’t have access to the source code of the software and is conducted at the software interface without any concern with the internal logical structure of the software is known as black-box testing.

2. **White-Box Testin**g
   The technique of testing in which the tester is aware of the internal workings of the product, has access to its source code, and is conducted by making sure that all internal operations are performed according to the specifications is known as white box testing.

**Unit Testing**
A level of the software testing process where individual units/components of a software/system are tested. The purpose is to validate that each unit of the software performs as designed.

**Integration Testing**
A level of the software testing process where individual units are combined and tested as a group. The purpose of this level of testing is to expose faults in the interaction between integrated units

**System Testing**
A level of the software testing process where a complete, integrated system/software is tested. The purpose of this test is to evaluate the system’s compliance with the specified requirements.

**Acceptance Testing**
A level of the software testing process where a system is tested for acceptability. The purpose of this test is to evaluate the system’s compliance with the business requirements and assess whether it is acceptable for delivery.

**Regression testing**
Checking whether new features break or degrade functionality

**Smoke tests**
check the basic functionality of an application, check ready for other tests, tests simple and basic functionalities, such as if the user is able to log in or log out.

**Performance tests**
Test the run-time performance of software - load testing

**Stress tests**
check how the system works under unfavorable conditions

## TDD & BDD
See [SDLC → BDD & TDD](sdlc.md#bdd--tdd) for the driven-development approaches.

## Principles & tooling
- **AAA** — Arrange, Act, Assert. Test behavior, not implementation. Deterministic, independent, fast.
- Java: **JUnit 5** + **Mockito**; `@SpringBootTest` / `@WebMvcTest` (MockMvc) for Spring; Testcontainers for real deps.
- Test doubles: dummy · stub · spy · mock · fake. Coverage (JaCoCo) is a guide, not a goal.
