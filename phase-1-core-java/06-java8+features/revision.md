# Java 8+ Features — Notes (Explained Properly)

---

## 1. Functional Interfaces

**Kya hai:** Ek functional interface wo interface hai jisme **exactly ek** abstract method hota hai — bas ye ek rule hai. Isme jitne chahe default/static methods ho sakte hain, lekin abstract sirf ek.

```java
interface Greeting {
    void greet(String name); // sirf ek abstract method
}
```

**Ye kyun matter karta hai:** Java 8 me lambdas introduce hue, aur ek lambda basically ek **functional interface ko implement karne ka chhota tarika** hai. Agar interface me sirf ek method hai, to Java ko exactly pata hota hai ki lambda kaunsa method implement kar raha hai — koi confusion nahi rehti ki "kaunsa method chalayein". Agar do abstract methods hote, to Java confuse ho jata ki lambda kis method ki jagah likha gaya hai.

`@FunctionalInterface` ek optional annotation hai jo compiler ko batati hai "ye interface functional hona chahiye". Agar galti se koi doosra abstract method add ho jaye, compiler turant error de dega — ye ek safety check hai.

### 4 Sabse Zaroori Built-in Functional Interfaces (java.util.function)

Java 8 ne ek poora package diya common use-cases ke liye — apna khud ka functional interface banane ki zaroorat nahi.

**Predicate\<T\>** — ek input leta hai, `boolean` return karta hai. Testing/filtering ke liye use hota hai.
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositive = n -> n > 0;

Predicate<Integer> isEvenAndPositive = isEven.and(isPositive); // dono match hone chahiye
Predicate<Integer> isEvenOrPositive = isEven.or(isPositive);   // koi ek match ho
Predicate<Integer> isNotEven = isEven.negate();                // opposite
```

**Function\<T, R\>** — ek input (T) leta hai, output (R) return karta hai. Transformation ke liye.
```java
Function<String, Integer> getLength = s -> s.length();
Function<String, String> toUpper = s -> s.toUpperCase();
Function<String, String> addGreeting = s -> "Hello, " + s;

// andThen: pehla function chalao, fir uska result doosre ko do
Function<String, String> combined = toUpper.andThen(addGreeting);
combined.apply("arman"); // "Hello, ARMAN"
```

**Consumer\<T\>** — ek input leta hai, kuch return nahi karta (void). Kuch **karne** ke liye — jaise print karna, save karna.
```java
Consumer<String> print = s -> System.out.println(s);
Consumer<String> printUpper = s -> System.out.println(s.toUpperCase());

Consumer<String> printBoth = print.andThen(printUpper); // ek ke baad doosra chalega
```

**Supplier\<T\>** — koi input nahi leta, ek value **supply/generate** karta hai.
```java
Supplier<String> greeting = () -> "Hello World";
Supplier<List<String>> listFactory = () -> new ArrayList<>(); // har call pe naya object
```

**Simple yaad rakhne ka tarika:** Predicate = "check karo", Function = "badlo", Consumer = "use karo", Supplier = "banao/do".

Iske alawa BiFunction (2 input, 1 output), BiPredicate (2 input, boolean), BiConsumer (2 input, void), UnaryOperator (input aur output same type), BinaryOperator (2 input aur output same type) bhi hain — same concepts, bas 2 arguments wale versions.

---

## 2. Lambda Expressions

**Kya problem solve karta hai:** Java 8 se pehle, agar tumhe kisi method ko **behaviour pass** karna hota (jaise ek callback), to tumhe ek poori **anonymous class** likhni padti thi — bahut verbose.

```java
// Purana tarika
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};

// Lambda
Runnable r = () -> System.out.println("Running");
```

Lambda basically ek functional interface ki implementation likhne ka **chhota tarika** hai. Chunki interface me sirf ek method hota hai, Java ko pata hota hai tum kaunsa method implement kar rahe ho.

### Syntax ki alag-alag forms
```java
(param1, param2) -> { statements; return value; }  // full syntax
(param1, param2) -> expression                      // one statement, no braces/return
parameter -> expression                              // single param, no parentheses
() -> expression                                     // no params
```

### Zaroori concept: Effectively Final Variables

Lambda apne surrounding scope ki variables use kar sakta hai, lekin wo variables **effectively final** honi chahiye — matlab lambda me use hone ke baad unhe change nahi kar sakte.

```java
String prefix = "Hello";
Consumer<String> greet = name -> System.out.println(prefix + " " + name);
// prefix = "Hi"; // COMPILE ERROR — kyunki lambda ne isko already use kar liya hai
```

**Kyun ye restriction hai:** Lambda ek alag thread pe bhi chal sakta hai (jaise ek Runnable). Agar variable badal sakta, to lambda ko pata hi nahi chalta kaunsa value use kare — race condition jaisi confusion ho jati. Isliye Java force karta hai ki jo variable lambda use karta hai, wo effectively constant rahe.

---

## 3. Method References

**Kya hai:** Jab lambda sirf ek **existing method ko call** kar raha ho, to usse aur bhi chhota likhne ka tarika — method reference.

```java
// Lambda
Function<String, Integer> parse = s -> Integer.parseInt(s);

// Method reference — same kaam, chhota syntax
Function<String, Integer> parse = Integer::parseInt;
```

Dono bilkul same kaam karte hain — method reference bas **cleaner** dikhta hai.

### 4 Types (samajhne ka tarika: "method kis pe call ho raha hai")

**Type 1 — Static method pe:**
```java
Function<Double, Double> abs = Math::abs; // Math.abs(x) jaisa
```

**Type 2 — Ek specific, already-known object pe:**
```java
String prefix = "Hello ";
Function<String, String> addPrefix = prefix::concat; // prefix.concat(x) jaisa
```

**Type 3 — Method ke apne parameter pe (sabse common):**
```java
Function<String, String> upper = String::toUpperCase; // s -> s.toUpperCase() jaisa
```
Yahan `String` khud type hai, aur method call **parameter par hi** ho raha hai — ye thoda confusing lagta hai shuru me, lekin yaad rakho: agar lambda me pehla parameter khud method call kar raha hai (`s -> s.toUpperCase()`), to wo Type 3 hai.

**Type 4 — Constructor reference (naya object banane ke liye):**
```java
Supplier<ArrayList> createList = ArrayList::new; // () -> new ArrayList() jaisa
```

Real usage me ye kaafi common hota hai:
```java
names.stream().map(String::toUpperCase).forEach(System.out::println);
```

---

## 4. Default aur Static Methods in Interfaces

**Kya problem solve karta hai:** Java 8 se pehle, agar tum ek existing interface me naya method add karte, to **har ek class** jo us interface ko implement kar rahi thi, use bhi wo naya method implement karna padta — poora existing code toot jata.

**Default method** iska solution hai — interface ke andar hi ek **body** likh dete hain. Implementing classes ko wo method **free me mil jata hai**, wo chahe to use as-is le lein ya override kar dein.

```java
interface Vehicle {
    String getBrand();          // abstract — implement karna zaroori
    int getSpeed();              // abstract

    default String describe() {  // default — has body, optional to override
        return "Brand: " + getBrand() + ", Speed: " + getSpeed();
    }
}

class Car implements Vehicle {
    // getBrand() and getSpeed() implement karna hi padega
    // describe() automatically mil gaya, bina kuch likhe
}
```

### Diamond Problem — jab do interfaces me same-naam ka default method ho

Agar ek class do interfaces implement karti hai, aur dono ke paas same naam ka default method ho, to Java **confuse ho jata hai kaunsa use kare** — isliye compiler force karta hai ki class explicitly override kare aur decide kare:

```java
class C implements A, B {
    @Override
    public void show() {
        A.super.show(); // A ka version specifically call karna
        B.super.show(); // B ka version specifically call karna
    }
}
```

### Static Methods in Interfaces

Static method interface ke **naam se hi call hoti hai**, kisi object se nahi, aur ye implementing classes ko inherit **nahi** hoti — ye zyada tar utility/factory methods ke liye use hoti hai:

```java
interface MathOperations {
    static MathOperations add() { return (a, b) -> a + b; }
}

MathOperations adder = MathOperations.add(); // interface name se call
```

---

## 5. Stream API

**Kya hai:** Stream elements ki ek sequence hai jispe tum operations chala sakte ho. **Ye ek data structure NAHI hai** — ye khud kuch store nahi karta, sirf kisi source (List/array) se data leta hai aur usse **process** karke result deta hai.

**Analogy:** Ek pipeline socho — ek taraf se data aata hai, beech me alag-alag operations se guzarta hai, doosri taraf ek final result ban ke nikalta hai.

**Zaroori baatein:**
- Stream original collection ko **modify nahi** karta
- Ek stream sirf **ek baar** use ho sakta hai — terminal operation ke baad wo band ho jata hai
- Operations **lazy** hote hain — jab tak terminal operation call na ho, kuch chalta hi nahi

```
Source → Intermediate Operations → Terminal Operation
List   → filter, map, sorted    → collect, forEach, count
```

### Intermediate Operations (naya Stream return karte hain)

**filter()** — condition match karne wale elements rakho:
```java
names.stream().filter(name -> name.length() > 4).collect(Collectors.toList());
```

**map()** — har element ko transform karo:
```java
names.stream().map(String::toUpperCase).collect(Collectors.toList());
```

**flatMap()** — jab har element **kai elements** me map hota ho aur tumhe ek **flat/single stream** chahiye ho:
```java
List<List<Integer>> nested = List.of(List.of(1,2,3), List.of(4,5));
List<Integer> flat = nested.stream()
    .flatMap(List::stream) // har inner list ek stream ban jati hai, sab merge ho jati hain
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5]
```
**Kab use karo:** Jab tumhare paas "list ke andar list" jaisa nested structure ho aur tumhe usse ek single flat list chahiye.

**sorted(), distinct(), limit(), skip()** — apne naam se hi samajh aate hain: sort karna, duplicates hatana, pehle N elements lena, pehle N elements skip karna.

### Terminal Operations (pipeline ko actually chalate hain, final result dete hain)

**collect()** — results ko ek collection me jama karta hai:
```java
List<String> list = names.stream().collect(Collectors.toList());
String joined = names.stream().collect(Collectors.joining(", ")); // "Arman, Priya, Raj"
```

**reduce()** — sab elements ko **ek single value** me combine karta hai:
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b); // 0 = starting value
```
Isko samajhne ka tarika: har step pe do values ko ek me combine karte jao, jab tak ek hi value na bache.

**anyMatch(), allMatch(), noneMatch()** — condition check karte hain, `boolean` return karte hain.

**findFirst(), min(), max()** — ye `Optional` return karte hain, kyunki stream **empty** bhi ho sakta hai — isliye direct value nahi, Optional wrapper milta hai (Optional ka concept aage detail me hai).

### Collectors — grouping, jo bahut kaam aata hai

```java
Map<Character, List<String>> grouped = names.stream()
    .collect(Collectors.groupingBy(name -> name.charAt(0)));
// {A=[Arman], P=[Priya, Preethi], R=[Raj, Rohan]}

Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.length() > 4));
// sirf 2 groups: true aur false
```

`groupingBy` = **kai groups** bana sakta hai (jaise pehla character alag-alag ho sakta hai). `partitioningBy` = hamesha sirf **2 groups** (true/false) banata hai.

---

## 6. Optional Class

**Kya problem solve karta hai:** Optional se pehle, jo method result nahi de pata tha, wo `null` return karta tha. Caller ko **yaad rakhna padta** tha har baar null check karna. Bhool gaye to `NullPointerException`.

```java
// Purana tarika
public static String findUser(int id) {
    if (id == 1) return "Arman";
    return null; // caller ko yaad rakhna padega null check karna
}
```

Optional ek **container** hai jo value ho ya na ho, dono handle karta hai. Ye caller ko **force** karta hai dono cases (value present/absent) explicitly handle karne ke liye — bhoolna mushkil ho jata hai.

```java
public static Optional<String> findUser(int id) {
    if (id == 1) return Optional.of("Arman");
    return Optional.empty(); // clearly bata raha hai "value nahi hai"
}
```

### Optional Banana
```java
Optional.of("Arman");           // value null NAHI honi chahiye, warna exception
Optional.ofNullable(maybeNull); // value null ho sakti hai, to empty Optional ban jayega
Optional.empty();               // explicitly empty
```

### Optional Use Karne Ke Achhe Tarike

**orElse()** — agar value nahi hai to default do:
```java
String value = opt.orElse("Default Name");
```

**orElseGet()** — jab default value **compute karna mehnga** ho:
```java
String val = opt.orElseGet(() -> getDefaultFromDatabase());
```
**Farak orElse vs orElseGet:** `orElse` hamesha apna default **turant evaluate** kar leta hai, chahe zaroorat ho ya na ho. `orElseGet` sirf **tabhi** evaluate karta hai jab Optional actually empty ho — isliye jab default value expensive ho (jaise DB call), `orElseGet` behtar hai.

**map()** — value present ho to transform karo, empty ho to empty hi rahega (koi NullPointerException nahi):
```java
Optional<String> upperName = name.map(String::toUpperCase);
```

**flatMap()** — jab `map()` khud ek `Optional` return kar deta hai (nested Optional se bachne ke liye):
```java
// map se milta: Optional<Optional<String>> — messy!
// flatMap se milta: Optional<String> — flat, clean
Optional<String> email = user.flatMap(u -> u.email);
```

**DO NOT:** `opt.isPresent()` check karke fir `opt.get()` karna — ye bilkul waisa hi hai jaise Optional use hi nahi kiya. Optional ka poora fayda tabhi milta hai jab `map`/`orElse`/`ifPresent` jaise methods se **chain** karo.

---

## 7. Date and Time API (java.time)

**Purana Date/Calendar kyun bekar tha:**
- `Date` **mutable** tha — koi bhi kabhi bhi change kar sakta tha
- `Calendar` me month **0-based** tha (January = 0) — confusing aur bugs ki wajah
- Thread-safety issues the

Java 8 ne poora naya, **immutable** API diya `java.time` package me.

### LocalDate, LocalTime, LocalDateTime — Simple Farak

- **LocalDate** — sirf date, koi time nahi (`2024-01-15`)
- **LocalTime** — sirf time, koi date nahi (`14:30:45`)
- **LocalDateTime** — dono saath (`2024-01-15T14:30:45`), lekin **timezone nahi**

```java
LocalDate today = LocalDate.now();
LocalDate specific = LocalDate.of(2024, 1, 15);

// Add/subtract — hamesha NAYI date return hoti hai (immutable, purani nahi badalti)
LocalDate nextWeek = today.plusDays(7);
```

### ZonedDateTime — Jab Timezone Matter Kare

Jaise alag-alag countries me meeting schedule karni ho:
```java
ZonedDateTime inIndia = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
ZonedDateTime sameTimeInNY = inIndia.withZoneSameInstant(ZoneId.of("America/New_York"));
// same actual moment, bas alag local representation
```

### Duration vs Period — Ye Do Confuse Hote Hain

- **Duration** = **time-based** amount (hours, minutes, seconds) — `LocalTime`/`LocalDateTime` ke saath use hota hai
- **Period** = **calendar-based** amount (years, months, days) — `LocalDate` ke saath use hota hai

```java
Duration workDay = Duration.between(LocalTime.of(9,0), LocalTime.of(17,30));
workDay.toHours(); // 8

Period age = Period.between(birthDate, today);
age.getYears(); // 24
```

**Yaad rakhne ka tarika:** Duration = "kitni der/ghante" (time granularity), Period = "kitne saal/mahine" (calendar granularity).

### DateTimeFormatter — Format aur Parse Karna

```java
DateTimeFormatter indianFormat = DateTimeFormatter.ofPattern("dd/MM/yyyy");
date.format(indianFormat); // "15/01/2024"

LocalDate parsed = LocalDate.parse("15/01/2024", indianFormat); // string se wapas date
```

### Instant — Machine Timestamp

Ye ek **point in time** hai, 1 Jan 1970 se milliseconds ke roop me — logging aur elapsed-time measure karne ke liye use hota hai:
```java
Instant start = Instant.now();
// kaam karo...
Duration elapsed = Duration.between(start, Instant.now());
```

---

## 8. var Keyword (Java 10)

**Kya hai:** `var` compiler ko **khud** local variable ka type figure out karne deta hai — tumhe explicitly type likhne ki zaroorat nahi.

```java
var names = new ArrayList<String>(); // type hai ArrayList<String>, bas likha nahi
```

**Important:** Ye **dynamic typing nahi hai!** Type compile-time pe hi **fixed** ho jata hai, sirf tumhe likhna nahi padta — Java abhi bhi strongly-typed hai.

### Limitations
- `var` initialize karna **zaroori** hai declare karte hi
- Method parameters, return types, class fields me `var` **use nahi** kar sakte — sirf local variables ke liye
- `var x = null` compile error deta hai — compiler type infer nahi kar pata

---

## 9. Immutable Collections (Java 9)

**Purana tarika (verbose):**
```java
List<String> names = Collections.unmodifiableList(Arrays.asList("Arman", "Priya"));
```

**Naya tarika:**
```java
List<String> names = List.of("Arman", "Priya", "Raj");
Set<Integer> scores = Set.of(85, 92, 71);
Map<String, Integer> marks = Map.of("Arman", 85, "Priya", 92);
```

Ye collections **poori tarah immutable** hain — `add()`, `remove()`, `set()` sab `UnsupportedOperationException` denge. Null allowed nahi. `Set.of()` me duplicates bhi allowed nahi.

**Map.of() ki limit:** Sirf 10 pairs tak. Zyada chahiye to `Map.ofEntries(Map.entry(k,v), ...)` use karo.

**List.copyOf() (Java 10):** Kisi existing (mutable) collection ka ek immutable, **independent snapshot** banata hai:
```java
List<String> copy = List.copyOf(original);
original.add("Naya"); // original badla
// copy waisa hi rahega jaisa banate waqt tha — independent copy hai
```

---

## 10. Useful String Methods (Java 11)

- **isBlank()** vs **isEmpty()**: `isEmpty()` sirf `""` check karta hai. `isBlank()` whitespace-only strings (`"   "`) ko bhi "blank" maanta hai.
- **strip()** vs **trim()**: `strip()` Unicode whitespace bhi handle karta hai, `trim()` sirf ASCII (basic spaces/tabs).
- **lines()** — string ko lines ka Stream banata hai, `\n` pe split karke.
- **repeat(n)** — string ko n baar repeat karta hai: `"Ha".repeat(3)` → `"HaHaHa"`.

---

## 11. Switch Expressions (Java 14)

**Purana switch — problems:** Verbose tha, `break` bhoolne se **fall-through bugs** hote the, aur seedha koi value return nahi karta tha.

```java
// Purana
switch (day) {
    case 1: dayName = "Monday"; break; // break bhoolna = bug
    ...
}
```

**Naya switch expression** — value seedhe return karta hai, fall-through nahi hota, `break` ki zaroorat nahi:
```java
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    default -> "Unknown";
};
```

**Multiple labels ek case me:**
```java
String type = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7 -> "Weekend";
};
```

**Block wala case, jab multiple statements chahiye:** `yield` keyword se value return karo (jaise `return` lekin switch block ke andar):
```java
case 7 -> {
    System.out.println("Just passed");
    yield "C";
}
```

---

## 12. Records (Java 16)

**Kya problem solve karta hai:** Ek simple data-holder class banane ke liye pehle bahut **boilerplate** likhna padta tha — constructor, getters, `equals()`, `hashCode()`, `toString()` sab manually.

```java
// Ek line me sab kuch!
record Person(String name, int age) {}
```

Java automatically generate kar deta hai:
- Constructor
- Getters — lekin naam `getName()` **nahi**, sirf `name()` aur `age()` (field ke naam jaisa hi)
- `equals()`, `hashCode()`, `toString()` — sab **field values** ke basis par
- Fields **final** hote hain — matlab record **immutable** hai by design

```java
Student s1 = new Student("Arman", 1, 85.5);
s1.name(); // "Arman" — na ki getName()
```

### Compact Constructor — Validation Ke Liye
```java
record Circle(double radius) {
    Circle { // parameters likhne ki zaroorat nahi, upar wale hi use hote hain
        if (radius <= 0) throw new IllegalArgumentException("Radius must be positive");
    }

    double area() { return Math.PI * radius * radius; } // instance methods allowed hain
}
```

**Kab use karo:** DTOs, API responses, value objects — koi bhi simple immutable data-carrier. **Mat use karo** jab tumhe kisi doosri class ko extend karna ho (records implicitly `Record` class extend karte hain, aur kisi aur ko extend nahi kar sakte), ya mutable state chahiye ho.

---

## 13. Sealed Classes (Java 17)

**Kya problem solve karta hai:** Normally, koi bhi class kahin se bhi tumhare kisi class ko extend kar sakti hai. Sealed classes tumhe control dete hain — **explicitly batao kaun extend kar sakta hai.**

Useful jab tumhare paas ek **fixed set of types** ho — jaise `Shape` sirf `Circle`, `Rectangle`, `Triangle` ho sakta hai, koi aur nahi.

```java
sealed class Shape permits Circle, Rectangle, Triangle {
    abstract double area();
}

final class Circle extends Shape { ... }   // final = isse aage extend nahi ho sakta
final class Rectangle extends Shape { ... }
final class Triangle extends Shape { ... }

// class Hexagon extends Shape { } // COMPILE ERROR — permitted list me nahi hai
```

Har permitted subclass ko in teen me se ek hona zaroori hai: `final` (aage extend nahi ho sakta), `sealed` (aage extend ho sakta hai lekin apni khud ki permitted list ke saath), ya `non-sealed` (wapas khula, koi bhi extend kar sakta hai).

### Sealed classes + switch — Sabse Bada Fayda

Chunki compiler ko **saare possible subtypes pehle se pata hain**, switch expression me compiler khud check kar leta hai ki tumne sab cases handle kiye ya nahi — `default` ki bhi zaroorat nahi padti:

```java
double area = switch (shape) {
    case Circle c -> Math.PI * c.radius * c.radius;
    case Rectangle r -> r.length * r.width;
    case Triangle t -> 0.5 * t.base * t.height;
    // default nahi chahiye — compiler ko pata hai sab cases cover ho gaye
};
```

---

## 14. Stream API Improvements (Java 9)

**takeWhile()** — jab tak condition true hai, elements lo. Jaise hi false mila, **turant ruk jao** (aage ke elements skip nahi karta, sirf ruk jata hai):
```java
List.of(2, 4, 6, 7, 8, 10).stream()
    .takeWhile(n -> n % 2 == 0)
    .collect(Collectors.toList()); // [2, 4, 6] — 7 pe ruk gaya, 8/10 ko check hi nahi kiya
```

**dropWhile()** — ispe reverse: jab tak condition true hai, elements **drop** karo. Ek baar false mila, baaki **sab** rakh lo (chahe wo condition match kare ya na kare):
```java
List.of(2, 4, 6, 7, 8, 10).stream()
    .dropWhile(n -> n % 2 == 0)
    .collect(Collectors.toList()); // [7, 8, 10] — 6 tak drop kiya, 7 se sab rakh liya
```

**Farak samajhne ka tarika:** `filter()` **har** element ko individually check karta hai poore stream me. `takeWhile`/`dropWhile` sirf **shuru se ek continuous stretch** tak dekhte hain, phir ruk jate hain — order-dependent hain, `filter()` nahi hai.

---

## 15. Pattern Matching for instanceof (Java 16+)

**Purana tarika:** `instanceof` check karne ke baad manually cast karna padta tha:
```java
if (obj instanceof String) {
    String s = (String) obj; // manual cast
}
```

**Naya tarika:** Cast **automatic** ho jata hai:
```java
if (obj instanceof String s) { // s already String type hai, cast ki zaroorat nahi
    System.out.println(s.toUpperCase());
}
```

---

## 16. Text Blocks (Java 15)

Multi-line strings likhne ka clean tarika, bina `\n` aur `+` concatenation ke:

```java
// Purana — messy
String json = "{\n" + "    \"name\": \"Arman\"\n" + "}";

// Text block — clean
String json = """
    {
        "name": "Arman"
    }
    """;
```

`\` line ke end me use karo agar newline **nahi** chahiye us jagah. `\s` use karo agar trailing spaces force karne hain (normally wo automatically strip ho jaate hain).

---

## Quick Reference Table

| Java Version | Feature | One-line Purpose |
|---|---|---|
| 8 | Functional Interfaces, Lambdas | Behaviour ko compact tarike se pass karna |
| 8 | Method References | Existing method call karne wale lambda ko aur chhota likhna |
| 8 | Default/Static methods in interfaces | Bina existing implementations tode interface me naya method add karna |
| 8 | Stream API | Collection data ko pipeline-style process karna |
| 8 | Optional | Null-check ko force karna, NullPointerException se bachna |
| 8 | java.time (LocalDate etc.) | Immutable, clear date/time handling |
| 9 | List.of()/Set.of()/Map.of() | Chhoti immutable collections aasani se banana |
| 9 | takeWhile()/dropWhile() | Stream me condition-based continuous stretch lena/hatana |
| 10 | var | Local variable type ko compiler se infer karwana |
| 11 | isBlank(), strip(), repeat() | Useful String utility methods |
| 14 | Switch expressions | Switch se value return karna, fall-through se bachna |
| 15 | Text blocks | Clean multi-line strings |
| 16 | Records | Boilerplate-free immutable data classes |
| 16 | Pattern matching instanceof | instanceof ke baad automatic cast |
| 17 | Sealed classes | Kis class ko extend karne diya jaye, uspe control |
