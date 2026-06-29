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

Plugin Repo: https://github.com/onlydevelop/FitnessFunctionPlugin

# Sample Code

Once again, I want to convey the hardwork to Emily Bache, who has created an excellent repository of refactoring kata. Code used here is from that only. 

Original kata is available here: https://github.com/emilybache/GildedRose-Refactoring-Kata

# Build Result

### agentKey=my-api-key ./gradlew clean build

```
> Task :app:checkFitness
Running fitness check with model: claude-sonnet-4-6
Prompt: Prioritize the code smells as high, medium, low. Fail the build if value of fitness function exceeds threshold. Give the 5 liner summerised output in ascii art format, not markdown.
=== Fitness Function Result ===

+=======================================================+
|          CODE SMELL ANALYSIS - GILDED ROSE            |
+=======================================================+
| SMELL                          | PRIORITY | COUNT     |
+--------------------------------+----------+-----------+
| Deep Nesting (6+ levels)       | HIGH     |     1     |
| Long Method (updateQuality)    | HIGH     |     1     |
| Magic Strings (hardcoded names)| HIGH     |     1     |
| Complex Conditionals           | MEDIUM   |     1     |
| Direct Field Access (no encap) | MEDIUM   |     1     |
| Duplicate Code (quality check) | MEDIUM   |     1     |
| Dead Code (x - x pattern)      | LOW      |     1     |
| Missing abstraction/polymorphsm| LOW      |     1     |
+=======================================================+

  FITNESS FUNCTION CALCULATION
  =============================
  HIGH   : 3 x 3.5 = 10.5
  MEDIUM : 3 x 2.7 =  8.1
  LOW    : 2 x 1.8 =  3.6
  ---------------------
  TOTAL SCORE        = 22.2
  THRESHOLD          = 20.0
  ---------------------
  STATUS : *** BUILD FAILED ***

+=======================================================+
|  [!] SCORE 22.2 EXCEEDS THRESHOLD 20.0               |
|  [!] 3 HIGH smells must be resolved immediately      |
|  [!] Replace magic strings with constants/polymorphsm |
|  [!] Refactor updateQuality into smaller methods      |
|  [!] Encapsulate Item fields; remove direct access    |
+=======================================================+

FITNESS_VALUE: 22.2
Fitness value: 22.2 (threshold: 20.0)

> Task :app:checkFitness FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:checkFitness'.
> Fitness function value 22.2 exceeds threshold 20.0. Build failed.

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights from a Build Scan (powered by Develocity).
> Get more help at https://help.gradle.org.

BUILD FAILED in 9s
9 actionable tasks: 9 executed
Configuration cache entry stored.
```

# Disclaimer

This is a POC to share the idea. None of the code, parameters, values are to be used as is in the production code. As it is using AI agent to derive, so expect non-deterministic results.

