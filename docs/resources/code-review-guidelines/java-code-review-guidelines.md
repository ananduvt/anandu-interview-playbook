# Java Code Review Guidelines

Comprehensive checklist for reviewing Java code — formatting, naming, exception handling, collections, null safety, performance, and best practices.

---

## 1. Variable & Constant Management

### String & Constant Declaration
- **Make string/global/instance variables `final`** unless they need to be reassigned
  ```java
  private final String constant = "VALUE";  // ✅ Good
  private String mutable = "VALUE";         // ⚠️ Only if needed
  ```

- **Declare common constants in a centralized Constants class**
  ```java
  // ❌ Bad - scattered across code
  String status = "Y";
  String status2 = "N";
  
  // ✅ Good - CommonImsApiConstants or similar
  public static final String STATUS_YES = "Y";
  public static final String STATUS_NO = "N";
  ```

- **Reuse constants for repeated values** (table names, column names, separators)
  ```java
  // ❌ Bad
  String separator = "---";
  String redisKey = key1 + "---" + key2 + "---" + key3;
  
  // ✅ Good
  private static final String SEPARATOR = "---";
  String redisKey = StringUtils.join(key1, SEPARATOR, key2, SEPARATOR, key3);
  ```

- **Use enums for status/type values**
  ```java
  // ❌ Bad
  String status = "PASSED";
  String messageType = "INFO";
  
  // ✅ Good
  enum MessageStatus { PASSED, FAILED, ERROR, WARNING, INFO }
  enum LogLevel { DEBUG, INFO, WARN, ERROR }
  ```

### Variable Naming Conventions
- **Use clear, meaningful names** — avoid single-letter variables (except loop counters)
  ```java
  // ❌ Bad
  String y = "Y";
  int n = 5;
  
  // ✅ Good
  String status = "Y";
  int maxRetries = 5;
  ```

- **Avoid overly long variable names**
  ```java
  // ❌ Bad
  String FAILED_TO_CONVERT_SPEED_PAY_MAINTENANCE_DATE = "...";
  
  // ✅ Good
  String SPEED_PAY_DATE_CONVERSION_ERROR = "...";
  ```

- **Boolean methods should start with `is` or `has`**
  ```java
  // ✅ Good
  public boolean isValid() { ... }
  public boolean hasPermission() { ... }
  public boolean isEmpty() { ... }
  ```

- **Getter methods must always return a value**
  ```java
  // ❌ Bad - null path
  public String getName() {
    if (name != null) return name;
    // else returns null implicitly
  }
  
  // ✅ Good - always returns
  public String getName() {
    return name != null ? name : "";
  }
  ```

---

## 2. Method Design

### Method Purpose & Signature
- **Each method should do ONE thing only** (Single Responsibility)
  ```java
  // ❌ Bad - does multiple things
  public void processAndValidateAndLog(Data data) { ... }
  
  // ✅ Good - single responsibility
  public void process(Data data) { ... }
  private void validate(Data data) { ... }
  private void log(Data data) { ... }
  ```

- **Limit method parameters to 6-7 maximum** — use object parameters for more
  ```java
  // ❌ Bad - too many params
  public void createRequest(String id, String name, String email, String phone, 
                           String address, String city, String state, String zip) { ... }
  
  // ✅ Good - encapsulate in object
  public void createRequest(UserRequest request) { ... }
  ```

- **Keep heavy/large objects as the last parameter**
  ```java
  // ✅ Good - context at the end
  public void process(String id, int priority, Map<String, Object> payload) { ... }
  ```

- **Limit statements per method to 50-70 lines** — break into smaller methods
  ```java
  // ❌ Bad - 200+ lines
  public void complexProcess() { ... }
  
  // ✅ Good - broken into steps
  public void complexProcess() {
    validateInput();
    fetchData();
    transform();
    save();
  }
  ```

### Return Statements & Logic
- **Avoid multiple return statements** — use single return or guard clauses
  ```java
  // ❌ Bad - multiple returns scattered
  public boolean validate(Data data) {
    if (data == null) return false;
    if (data.isEmpty()) return false;
    if (!data.isValid()) return false;
    return true;
  }
  
  // ✅ Good - single return
  public boolean validate(Data data) {
    if (data == null || data.isEmpty() || !data.isValid()) return false;
    return true;
  }
  
  // ✅ Also Good - guard clauses
  public boolean validate(Data data) {
    if (data == null) return false;
    if (data.isEmpty()) return false;
    return data.isValid();
  }
  ```

- **Segregate logic from return statement**
  ```java
  // ❌ Bad - complex logic in return
  return values.stream().filter(...).map(...).sorted(...).collect(...);
  
  // ✅ Good - separate logic and return
  List<Result> filtered = filterAndProcess(values);
  return filtered;
  ```

---

## 3. Exception Handling

### Exception Declaration & Propagation
- **Use `throws` in method signature** — don't wrap with unnecessary try-catch
  ```java
  // ❌ Bad - wrapping in try-catch
  public void processRequest() {
    try {
      executeMultiDbRequest();
    } catch (Exception e) {
      LOGGER.error(e);
    }
  }
  
  // ✅ Good - let caller handle
  public void processRequest() throws BaseException {
    executeMultiDbRequest();
  }
  ```

- **Throw specific, relevant exceptions** — not generic `Exception`
  ```java
  // ❌ Bad
  throw new Exception("Error processing");
  
  // ✅ Good
  throw new ValidationException("Invalid customer ID");
  throw new DatabaseException("Connection failed");
  ```

- **For `throws` method, use naming convention `throw[ExceptionType][MethodName]`**
  ```java
  // ❌ Bad
  private void validateData() throws BaseException { ... }
  
  // ✅ Good
  private void throwValidationException() throws BaseException { ... }
  ```

### Logging Exceptions
- **Log meaningful exception information** — use `getMessage()` or detail, not raw exception
  ```java
  // ❌ Bad - too verbose or vague
  LOGGER.error(exception);  // Prints whole stack
  LOGGER.error("Error");    // No details
  
  // ✅ Good - clear message + exception for context
  LOGGER.error("Failed to process customer {}: {}", customerId, exception.getMessage(), exception);
  
  // ✅ Also Good - message-only when appropriate
  LOGGER.error("Database connection failed", exception);
  ```

- **Don't suppress unchecked exceptions at class level**
  ```java
  // ❌ Bad
  @SuppressWarnings("unchecked")
  public class DataProcessor { ... }
  
  // ✅ Good - suppress only when necessary
  @SuppressWarnings("unchecked")
  List<String> items = (List<String>) unknownList;
  ```

- **Use appropriate log levels in switch/default cases**
  ```java
  // ✅ Good - error for unhandled case
  switch(type) {
    case TYPE_A: ...
    case TYPE_B: ...
    default: LOGGER.error("Unexpected type: {}", type);
  }
  ```

---

## 4. Null & Empty Checks

### Null Safety
- **Use `Object.nonNull()` for null checks**
  ```java
  // ❌ Bad
  if (obj != null) { ... }
  
  // ✅ Good - clearer intent
  if (Objects.nonNull(obj)) { ... }
  ```

- **Avoid double negatives in null checks**
  ```java
  // ❌ Bad - confusing
  if (!StringUtils.isNotBlank(value)) { ... }
  if (!CollectionUtils.isNotEmpty(list)) { ... }
  
  // ✅ Good - direct check
  if (StringUtils.isBlank(value)) { ... }
  if (CollectionUtils.isEmpty(list)) { ... }
  ```

- **Multiple null checks on same variable — use single check**
  ```java
  // ❌ Bad - redundant
  if (value != null && NumberUtils.isParsable(value)) { ... }
  
  // ✅ Good - isParsable includes null check
  if (NumberUtils.isParsable(value)) { ... }
  ```

- **Avoid unnecessary null checks when guaranteed by framework**
  ```java
  // Example: If cardSeq.getCurrValue() is guaranteed not null by ORM
  // ❌ Bad - redundant check
  if (sh10.getCardSeq() != null && sh10.getCardSeq().getCurrValue() != null) { ... }
  
  // ✅ Good - trust the contract if CardSeq is PK
  if (sh10.getCardSeq().getCurrValue().equals(expectedValue)) { ... }
  ```

### Collection Checks
- **Use Spring's `CollectionUtils` instead of manual checks**
  ```java
  // ❌ Bad
  if (list != null && !list.isEmpty()) { ... }
  
  // ✅ Good
  if (!CollectionUtils.isEmpty(list)) { ... }
  
  // ✅ Also Good
  if (CollectionUtils.isNotEmpty(list)) { ... }  // Positive condition
  ```

- **Use `StringUtils.isAllEmpty()` for multiple string checks**
  ```java
  // ❌ Bad - verbose
  if (StringUtils.isEmpty(state) && StringUtils.isEmpty(abbrev) && StringUtils.isEmpty(code)) { ... }
  
  // ✅ Good - concise
  if (StringUtils.isAllEmpty(state, abbrev, code)) { ... }
  ```

- **Check before calling trim() or strip()**
  ```java
  // ❌ Bad - NPE if value is null
  String trimmed = value.trim();
  
  // ✅ Good - check first
  String trimmed = StringUtils.isNotBlank(value) ? value.trim() : "";
  ```

---

## 5. Collections & Streams

### Stream API Best Practices
- **Use Java 8+ Streams instead of traditional loops** when possible
  ```java
  // ❌ Bad - verbose loop
  List<String> upper = new ArrayList<>();
  for (String s : words) {
    upper.add(s.toUpperCase());
  }
  
  // ✅ Good - stream
  List<String> upper = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
  ```

- **Use predicates for complex filter conditions**
  ```java
  // ❌ Bad - complex condition in filter
  List<Data> result = dataList.stream()
    .filter(x -> x.getType() != null && x.getType().equals("A")
               && x.getStatus() != null && x.getStatus().equals("ACTIVE")
               && x.getDate() != null && x.getDate().equals(targetDate))
    .collect(Collectors.toList());
  
  // ✅ Good - separate predicate
  Predicate<Data> isValidData = d -> d.getType().equals("A")
                                  && d.getStatus().equals("ACTIVE")
                                  && d.getDate().equals(targetDate);
  List<Data> result = dataList.stream()
    .filter(isValidData)
    .collect(Collectors.toList());
  ```

- **Use `collect()` over `reduce()` for accumulation**
  ```java
  // ❌ Bad - reduce for string concat
  String result = words.stream()
    .reduce("", (a, b) -> a + b);  // Inefficient
  
  // ✅ Good - use Collectors
  String result = words.stream()
    .collect(Collectors.joining(""));  // Uses StringBuilder
  ```

- **Use `IntStream.range()` for numeric ranges**
  ```java
  // ❌ Bad
  List<Integer> months = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12);
  
  // ✅ Good
  List<Integer> months = IntStream.range(1, 13)
    .boxed()
    .collect(Collectors.toList());
  ```

### Collection Initialization
- **Use `List.of()` instead of `new ArrayList<>()` for immutable lists**
  ```java
  // ❌ Bad
  List<String> items = new ArrayList<>(Arrays.asList("a", "b", "c"));
  
  // ✅ Good - immutable
  List<String> items = List.of("a", "b", "c");
  
  // Note: Use ArrayList only if you need mutability
  List<String> mutable = new ArrayList<>(List.of("a", "b", "c"));
  ```

- **Use pattern matching for instanceof + casting (Java 16+)**
  ```java
  // ❌ Bad - verbose casting
  if (responseObject instanceof IFF) {
    IFF iff = (IFF) responseObject;
    process(iff);
  }
  
  // ✅ Good - pattern matching
  if (responseObject instanceof IFF iff) {
    process(iff);
  }
  ```

---

## 6. Code Formatting & Style

### Braces & Structure
- **Opening brace `{` at end of line, closing brace `}` on new line**
  ```java
  // ✅ Good
  if (condition) {
    doSomething();
  }
  
  public void method() {
    ...
  }
  ```

- **Always use braces, even for single-statement blocks**
  ```java
  // ❌ Bad
  if (condition) doSomething();
  
  // ✅ Good
  if (condition) {
    doSomething();
  }
  ```

- **Avoid unnecessary nested blocks** — keep nesting to 3 levels max
  ```java
  // ❌ Bad - 5+ levels
  if (a) {
    if (b) {
      if (c) {
        if (d) {
          if (e) {
            action();
          }
        }
      }
    }
  }
  
  // ✅ Good - guard clauses
  if (!isValid(a, b, c, d, e)) return;
  action();
  ```

- **Use early returns to avoid else blocks**
  ```java
  // ❌ Bad - else block
  if (condition) {
    doA();
  } else {
    doB();
  }
  
  // ✅ Good - early return
  if (!condition) {
    doB();
    return;
  }
  doA();
  ```

- **Leave a blank line after logical groups of statements**
  ```java
  // ✅ Good - clear separation
  String name = request.getName();
  int age = request.getAge();
  
  validate(name, age);
  save(request);
  ```

### Logical Operators & Readability
- **Use parentheses for complex conditions** — especially with multiple `||`
  ```java
  // ❌ Bad - unclear precedence
  if (StringUtils.isNotBlank(time) && EventType.TYPE_975.equals(type) || EventType.TYPE_976.equals(type)) { ... }
  
  // ✅ Good - explicit grouping
  if (StringUtils.isNotBlank(time) && (EventType.TYPE_975.equals(type) || EventType.TYPE_976.equals(type))) { ... }
  ```

- **When using `||`, put the object check before method calls**
  ```java
  // ❌ Bad - obj could be null in second condition
  if (obj.isEmpty() || obj == null) { ... }
  
  // ✅ Good - null check first
  if (obj == null || obj.isEmpty()) { ... }
  ```

- **Use `.matches()` instead of multiple `||` operators**
  ```java
  // ❌ Bad - multiple checks
  if (status.equals("Y") || status.equals("YES") || status.equals("1")) { ... }
  
  // ✅ Good - regex
  if (status.matches("Y|YES|1")) { ... }
  ```

---

## 7. Performance & Optimization

### Database & Caching
- **Minimize CICS calls** — use multi-ref features when available
  ```java
  // ❌ Bad - multiple CICS calls
  Data d1 = fetchData("TABLE1", key);
  Data d2 = fetchData("TABLE2", key);
  Data d3 = fetchData("TABLE3", key);
  
  // ✅ Good - batch call
  MultiRefResult result = fetchMultiRefData(Arrays.asList("TABLE1", "TABLE2", "TABLE3"), key);
  ```

- **Use SSA qualifications for filtering** — don't filter in Java code
  ```java
  // ❌ Bad - Java-side filter
  List<Data> result = fetchAll();
  filtered = result.stream().filter(d -> d.getStatus().equals("ACTIVE")).collect(Collectors.toList());
  
  // ✅ Good - filter in query
  List<Data> result = fetchWithQualification("STATUS = 'ACTIVE'");
  ```

- **Use StringBuilder instead of String concatenation in loops**
  ```java
  // ❌ Bad - creates many String objects
  String result = "";
  for (String s : items) {
    result += s + separator;
  }
  
  // ✅ Good - single StringBuilder
  StringBuilder result = new StringBuilder();
  for (String s : items) {
    result.append(s).append(separator);
  }
  ```

- **Reuse common utility methods** — don't duplicate logic
  ```java
  // ❌ Bad - custom implementation
  private String getRestartToken(String... params) {
    StringBuilder sb = new StringBuilder();
    for (String p : params) {
      sb.append(p).append("|");
    }
    return sb.toString();
  }
  
  // ✅ Good - use StringUtils.join
  String key = StringUtils.join(param1, "|", param2, "|", param3);
  ```

### Assignment & Type Handling
- **Avoid unnecessary assignments and casting**
  ```java
  // ❌ Bad - extra assignment and cast
  var amt = new AmtAtCalculation();
  amt = (AmtAtCalculation) CurrencyUtil.formatCurrency(amt, original, currency);
  
  // ✅ Good - direct call
  CurrencyUtil.formatCurrency(amt, original, currency);
  ```

- **Use var for type inference** (Java 10+) when type is obvious
  ```java
  // ✅ Good - type is clear
  var customers = repository.findAll();
  var count = items.size();
  
  // ⚠️ Avoid when unclear
  var x = complexMethodCall();  // What type is x?
  ```

---

## 8. Spring & Framework-Specific

### Dependency Injection & Components
- **Use `@Component` on helper classes** — enables DI and Spring management
  ```java
  // ❌ Bad - manual instantiation
  private EbcidicComparator comparator = new EbcidicComparator();
  
  // ✅ Good - DI via Spring
  @Component
  public class EbcidicComparator { ... }
  
  @Autowired
  private EbcidicComparator comparator;
  ```

- **Annotate DataHelper with `@Component`** — for Spring management and lifecycle
  ```java
  @Component
  public class DataHelper {
    // Spring manages lifecycle, no manual instantiation
  }
  ```

- **Use `@PostConstruct` for initialization**
  ```java
  // ✅ Good - called after DI
  @PostConstruct
  private void initSegmentRetrievalInfo() {
    this.segments = initializeSegments();
  }
  ```

### Method Parameter Handling
- **For PacketProcessor, don't set PacketID again** — base method already does it
  ```java
  // ❌ Bad - redundant
  BasePacketRequest request = createBasePacketRequest();
  request.setPacketID(packetId);  // Already done in createBasePacketRequest()
  
  // ✅ Good
  BasePacketRequest request = createBasePacketRequest();
  // PacketID already set
  ```

---

## 9. Miscellaneous

### Avoiding Anti-Patterns
- **Don't use `@Deprecated` in production code** — remove deprecated code
  ```java
  // ❌ Bad
  @Deprecated
  public void oldMethod() { ... }
  
  // ✅ Good - remove or provide migration path
  public void newMethod() { ... }
  ```

- **Don't use `HttpServletRequest.getRequestSessionId()`** — use Spring Security
  ```java
  // ✅ Good - use Spring Security
  Authentication auth = SecurityContextHolder.getContext().getAuthentication();
  String userId = auth.getPrincipal().toString();
  ```

- **Don't commit files from `bin/` folder** — add to `.gitignore`
- **Don't commit files with no changes** — review diffs before staging

### Comments & Documentation
- **Avoid redundant comments** — code should be self-documenting
  ```java
  // ❌ Bad - obvious what it does
  // Increment counter
  counter++;
  
  // ✅ Good - explains WHY
  // Increment counter to track failed retry attempts
  counter++;
  ```

- **No comments for removed code** — use git history
  ```java
  // ❌ Bad - leaves clutter
  // DoSomething();  // Removed in v2.0
  
  // ✅ Good - let git track it
  // Code removed in commit abc123
  ```

---

## 10. Enum & Type Safety

### Enum Design
- **Make enum fields `private` and set via constructor**
  ```java
  // ❌ Bad - public fields, security risk
  public enum Status {
    ACTIVE("A"), INACTIVE("I");
    public String code;
  }
  
  // ✅ Good - private fields, immutable
  public enum Status {
    ACTIVE("A"), INACTIVE("I");
    private final String code;
    
    Status(String code) {
      this.code = code;
    }
    
    public String getCode() { return code; }
  }
  ```

### Replace Switch with Enum
- **Use Enum methods instead of switch statements** when possible
  ```java
  // ❌ Bad - switch statement
  switch(status) {
    case "ACTIVE": return true;
    case "INACTIVE": return false;
    default: return false;
  }
  
  // ✅ Good - enum method
  public enum Status {
    ACTIVE(true), INACTIVE(false);
    private final boolean isActive;
    
    public boolean isActive() { return isActive; }
  }
  ```

---

## Checklist for Code Reviews

### Before Review
- [ ] Understand the PR intent and requirements
- [ ] Check if related tests are included
- [ ] Look for TODO/FIXME comments (should be in issues, not code)

### Critical Issues
- [ ] All methods throw exceptions properly (no silent failures)
- [ ] Exception handling uses specific exceptions, not generic `Exception`
- [ ] Null checks are in place for external input
- [ ] No SQL injection or security vulnerabilities
- [ ] No hardcoded credentials or sensitive data

### Code Quality
- [ ] Methods are focused and do one thing
- [ ] No method exceeds 50-70 statements
- [ ] Variable names are clear and meaningful
- [ ] No commented-out code
- [ ] No unnecessary imports or variables
- [ ] Logging is appropriate and non-verbose

### Performance
- [ ] No N+1 database queries
- [ ] Collections properly sized/initialized
- [ ] Streams used correctly (lazy evaluation)
- [ ] No unnecessary string concatenation in loops
- [ ] CICS calls are minimized

### Style & Formatting
- [ ] Braces formatted correctly (opening at end of line)
- [ ] No deeply nested blocks (max 3 levels)
- [ ] Blank lines separate logical sections
- [ ] Complex conditions have parentheses for clarity
- [ ] Consistent with team style guide

### Testing
- [ ] All new code has tests
- [ ] Edge cases covered (null, empty, boundary values)
- [ ] No test files modified unless necessary
- [ ] Test data is realistic and documented

---

## Source & Attribution

### External References & Best Practices
- **Java Code Style**: [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- **Clean Code**: Robert C. Martin's "Clean Code" — method design, naming conventions
- **Effective Java**: Joshua Bloch — null safety, collections, generics
- **Design Patterns**: Gang of Four design patterns — enum patterns, immutability
- **Spring Framework**: [Spring Best Practices](https://spring.io/guides) — DI, component lifecycle
- **Java 8+ Features**: Streams, lambda expressions, Optional patterns
- **Performance Tuning**: String concatenation, collection initialization best practices

### Standard Documentation
- Oracle Java Language Specification
- Maven & Gradle conventions
- Spring Framework documentation
- JUnit & testing best practices

