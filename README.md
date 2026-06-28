# Refactoring Fitness Function

Of late, in LinkedIn I wrote about the idea of creating a fitness function for code smells and fail the build if it goes beyond a threshold. 

This was purely from governance for clean code, where this will will run in nightly build (if you have tokens you can run it once before pushing your changes to remote).

So, I just came up with a gradle plugin where I used anthropic agent to do the analysis, calculate the fitness function and publish the result, and of course fail the build if smell is beyond a configurable threshold.

## Plugin configuration

Plugin configuration is simple. We need to give the prompt, fitness function and threshold. The prompt is very vague, just for testing not to be used in production, but just to give an idea. Similary, fitness function is also a madeup one, nowhere it is a standard function.

```
plugins {
    ...
    id 'com.anvil.fitness-function' version '1.0.0'
}

tasks.named('build') {
    dependsOn 'checkFitness'
}

fitnessFunction {
    agent = 'claude'
    model = 'claude-sonnet-4-6'
    prompt = 'Prioritize the code smells as high, medium, low. Print the value of the fitness function with individual scores. Fail the build if value exceeds threshold.'
    fitnessFunction = 'high x 3.5 + medium x 2.7 + low x 1.8'
    threshold = 20
}
```

# Sample Code

Once again, I want to convey the hardwork to Emily Bache, who has created an excellent repository of refactoring kata. Code used here is from that only. 

Original kata is available here: https://github.com/emilybache/GildedRose-Refactoring-Kata

# Build Result

agentKey=my-api-key ./gradlew clean build
```
> Task :app:checkFitness
Running fitness check with model: claude-sonnet-4-6
Prompt: Prioritize the code smells as high, medium, low. Print the value of the fitness function with individual scores. Fail the build if value exceeds threshold.
```
=== Fitness Function Result ===
# Code Smell Analysis & Fitness Function Evaluation

## Detected Code Smells

### 🔴 HIGH Priority (weight: 3.5)

| # | Smell | Location | Description |
|---|-------|----------|-------------|
| H1 | **Deep Nesting** | `GildedRose.java` | `updateQuality()` has nesting depth of 6+ levels, making it nearly unreadable |
| H2 | **Long Method** | `GildedRose.java:updateQuality()` | Method is 50+ lines doing too many things at once |
| H3 | **Magic Strings** | `GildedRose.java` | Item names like `"Aged Brie"`, `"Sulfuras, Hand of Ragnaros"`, `"Backstage passes to a TAFKAL80ETC concert"` hardcoded repeatedly |

**High Count: 3**

---

### 🟡 MEDIUM Priority (weight: 2.7)

| # | Smell | Location | Description |
|---|-------|----------|-------------|
| M1 | **Duplicate Conditional** | `GildedRose.java` | `items[i].name.equals("Sulfuras...")` repeated 3 times |
| M2 | **Feature Envy** | `GildedRose.java` | `updateQuality()` directly manipulates `Item` fields instead of delegating to `Item` |
| M3 | **Magic Numbers** | `GildedRose.java` | Numbers `50`, `11`, `6` used without named constants |
| M4 | **Refused Bequest / Anemic Domain Model** | `Item.java` | `Item` is a pure data class with no behavior; logic belongs inside it |

**Medium Count: 4**

---

### 🟢 LOW Priority (weight: 1.8)

| # | Smell | Location | Description |
|---|-------|----------|-------------|
| L1 | **Obscure Expression** | `GildedRose.java:line 46` | `items[i].quality = items[i].quality - items[i].quality` instead of simply `= 0` |
| L2 | **Array Index Loop** | `GildedRose.java` | Using index-based `for` loop instead of enhanced `for-each` |
| L3 | **Dead / Trivial Code** | `App.java` | `App.java` has no meaningful logic; `getGreeting()` adds no value |
| L4 | **Poor Naming** | `GildedRose.java` | Loop variable `i` acceptable but `items[i]` repeated ~20 times; missing extraction to local variable |

**Low Count: 4**

---

## Fitness Function Calculation

```
Fitness = (High × 3.5) + (Medium × 2.7) + (Low × 1.8)
        = (3 × 3.5)  + (4 × 2.7)      + (4 × 1.8)
        = 10.5       + 10.8            + 7.2
        = 28.5
```

### Score Breakdown

```
┌─────────────────────────────────────────────┐
│          FITNESS FUNCTION REPORT             │
├──────────────┬───────┬────────┬─────────────┤
│ Priority     │ Count │ Weight │ Score       │
├──────────────┼───────┼────────┼─────────────┤
│ HIGH         │   3   │  3.5   │  10.50      │
│ MEDIUM       │   4   │  2.7   │  10.80      │
│ LOW          │   4   │  1.8   │   7.20      │
├──────────────┴───────┴────────┼─────────────┤
│ TOTAL FITNESS VALUE           │  28.50      │
│ THRESHOLD                     │  20.00      │
│ EXCEEDED BY                   │   8.50      │
└───────────────────────────────┴─────────────┘
```

---

## 🚨 BUILD FAILED

```
╔══════════════════════════════════════════════════╗
║              ❌  BUILD FAILED                    ║
║                                                  ║
║  Fitness Value : 28.50                           ║
║  Threshold     : 20.00                           ║
║  Exceeded by   : 8.50                            ║
║                                                  ║
║  Resolve HIGH and MEDIUM code smells before      ║
║  merging. Refactor GildedRose.updateQuality()    ║
║  as a priority.                                  ║
╚══════════════════════════════════════════════════╝
```

FITNESS_VALUE: 28.5
Fitness value: 28.5 (threshold: 20.0)

> Task :app:checkFitness FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:checkFitness'.
> Fitness function value 28.5 exceeds threshold 20.0. Build failed.

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights from a Build Scan (powered by Develocity).
> Get more help at https://help.gradle.org.

BUILD FAILED in 18s
9 actionable tasks: 9 executed
Configuration cache entry stored.

# Disclaimer

This is a POC to share the idea. None of the code, parameters, values are to be used as is in the production code. As it is using AI agent to derive, so expect non-deterministic results.

