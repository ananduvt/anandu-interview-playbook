# Date and Time (java.time)

The `java.time` API (Java 8) replaces the error-prone, mutable `java.util.Date` and `Calendar`. All classes are **immutable and thread-safe** and follow the ISO-8601 calendar.

## Core Classes

| Class | Represents | Example |
|-------|-----------|---------|
| `Instant` | Point on the UTC timeline (nanosecond precision); machine timestamps | `2026-10-26T10:40:00Z` |
| `LocalDate` | Date without time or zone | `2026-10-26` |
| `LocalTime` | Time without date or zone | `16:10:00` |
| `LocalDateTime` | Date and time, no zone | `2026-10-26T16:10:00` |
| `ZonedDateTime` | Date-time with a resolved zone | `2026-10-26T16:10:00+05:30[Asia/Kolkata]` |
| `Duration` | Time-based amount (seconds, nanos) | `Duration.ofMinutes(90)` |
| `Period` | Date-based amount (years, months, days) | `Period.ofDays(7)` |

## Common Operations

```java
import java.time.*;
import java.time.format.DateTimeFormatter;

LocalDate today = LocalDate.now();                         // factory: now()
LocalDate date  = LocalDate.of(2026, Month.OCTOBER, 26);   // factory: of()

LocalDate nextWeek = today.plusDays(7);   // immutable: returns a NEW object
today.plusDays(7);                        // result discarded, 'today' unchanged

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String text = LocalDateTime.now().format(fmt);                    // object -> String
LocalDateTime parsed = LocalDateTime.parse("2026-10-26 16:10:00", fmt); // String -> object

long minutes = Duration.between(parsed, LocalDateTime.now()).toMinutes();
Period age = Period.between(LocalDate.of(1990, 5, 1), today);
ZonedDateTime ist = Instant.now().atZone(ZoneId.of("Asia/Kolkata"));
```

## Interview Tips

- Store and transmit **`Instant`** (UTC); convert to `ZonedDateTime` only for display.
- Use `LocalDateTime` only when the zone truly does not matter; it cannot identify a moment on its own.
- `DateTimeFormatter` is thread-safe, unlike `SimpleDateFormat`. Pattern letters are case sensitive (`MM` month, `mm` minute, `yyyy` year, `HH` 24-hour).
- Pass a `Clock` to `now(clock)` to make time-dependent code testable.
- `Duration` for seconds/hours, `Period` for days/months/years (`Period` respects calendar rules, `Duration` does not).

**Practice questions:** [Java Core](../interview-questions/verbal/verbal-java-core-questions.md)
