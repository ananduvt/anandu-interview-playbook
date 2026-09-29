# Java Testing

## Junit Basics

[https://junit.org/junit5/docs/current/user-guide/](https://junit.org/junit5/docs/current/user-guide/)
[https://www.vogella.com/tutorials/JUnit/article.html](https://www.vogella.com/tutorials/JUnit/article.html)
[https://www.simplilearn.com/tutorials/java-tutorial/what-is-junit](https://www.simplilearn.com/tutorials/java-tutorial/what-is-junit)

## How to Test Private Methods in Java

[https://medium.com/@AlexanderObregon/how-to-test-private-methods-in-java-ec1872e81911](https://medium.com/@AlexanderObregon/how-to-test-private-methods-in-java-ec1872e81911)
[https://www.baeldung.com/java-unit-test-private-methods](https://www.baeldung.com/java-unit-test-private-methods)

As a rule, the unit tests we write should only check our public methods contracts. Private methods are implementation details that the callers of our public methods aren’t aware of. Furthermore, changing our implementation details shouldn’t lead us to change our tests.

Generally speaking, urging to test a private method highlights one of the following problems:

* We have dead code in our private method.
* Our private method is too complex and should belong to another class.
* Our method wasn’t meant to be private in the first place.

Hence, when we feel like we need to test a private method, what we should really do is fix the underlying design problem instead.

In case we want to test the Junt, there are two ways to do this.

1. **Same package**
   The standard way is to define your test class in the same package of the class to be tested. This should be easily done as modern IDE generates test cases in the same package of the class being tested by default.
2. **Use Reflection**
   The non-standard but very useful way is to use reflection. This allows you to define private methods as real "private" rather than "package private". For example, if you have class.
   class MyClass { private Boolean methodToBeTested(String argument) { ........ } }

   You can have your test method like this:
   class MyTestClass {
      @Test
      public void testMethod() {
          Method method = MyClass.class.getDeclaredMethod("methodToBeTested", String.class);
          method.setAccessible(true);
          Boolean result = (Boolean)method.invoke(new MyClass(), "test parameter");
          Assert.assertTrue(result);
      }
   }
