---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# **Lecture 03 Code Review**

**Course:** Application Development
**Topic:** Application Development

**The Big Three:**
* Safe from bugs (SFB)
* Easy to understand (ETU)
* Ready for change (RFC)

---

### What is Code Review?

**Definition:** The careful, systematic study of source code by people who are not the original author.

**Analogy:** It is like proofreading a paper.

**Industry Standard:** Widely practiced at companies like Google, where code cannot be merged without an engineer's sign-off.

---

### Two Main Purposes of Code Review

* **Improving the Code:** Finding bugs, checking clarity, and ensuring consistency with project standards.
* **Improving the Programmers:** A way for team members to teach each other about language features, design changes, and new techniques.

<!-- note: Research shows code review can find 70-90% of software defects. It is a high-value practice supported by engineers at Google and Microsoft. -->

---

### Style Standards

* **Consistency is Key:** Many large projects have detailed style guides (e.g., indentation, brace placement).
* **6.102 Stance:** No official style guide for brace placement, but self-consistency and following project conventions are mandatory.
* **Recommended Guides:** Idiomatic JavaScript or the Google TypeScript Style Guide.

---

### Smelly Example #1: dayOfYear

```typescript
function dayOfYear(month: number, dayOfMonth: number, year: number): number {
    if (month === 2) { dayOfMonth += 31; }
    else if (month === 3) { dayOfMonth += 59; }
    else if (month === 4) { dayOfMonth += 90; }
    // ... continues for all 12 months ...
    return dayOfMonth;
}
```

**The Problem:** This code "smells" (code hygiene issues).

---

### Don't Repeat Yourself (DRY)

**The Mantra:** Avoid duplicated code at all costs.

**The Risk:** If a bug exists in duplicated code, a maintainer might fix it in one place but forget the other.

**DRY Strategies:**
* Replace repeated expressions with variables.
* Replace repeated blocks with loops.
* Replace repeated blocks with helper functions.

---

### DRYing Out dayOfYear

In the example, the number of days in months is repeated multiple times.

**Solution:** Use a data structure like an array:
```typescript
const monthLength = [0, 31, 28, 31, 30, ...];
```

* Replace the long if-else chain with a single for loop that sums elements from the array.

---

### Comment Where Needed

Good comments make code ETU, SFB (by documenting assumptions), and RFC.

**Crucial Comment Types:**
* **Specifications:** Document function/class behavior (parameters, returns).
* **Provenance:** Documenting the source of copied/adapted code to avoid copyright issues and track known bugs.

---

### Documentation Comments in TypeScript

* Use the `/** ... */` syntax.
* Use `@param` and `@returns` tags.

**Example:**
```typescript
/**
 * Calculates the day of the year.
 * @param month month of the year (1-12)
 * @param dayOfMonth day of the month (1-31)
 * @param year year (e.g. 2023)
 * @returns day of the year (1-366)
 */
```

---

### Bad vs. Useful Comments

* **Avoid Transliteration:** Don't explain what the code literally does (e.g., `++i; // increment i`).
* **Explain "Why" or "Obscure" Logic:** Use comments for complex formulas (like Gauss's formula) or approximations.
* **Topic Sentences:** Use comments as "paragraphs" to group lines with a focused purpose.

---

### Fail Fast

**Principle:** Code should reveal bugs as early as possible.

**Hierarchy of Failure Speed:**
1. Static checking (Fastest/Best).
2. Dynamic checking.
3. Wrong answer (Slowest/Worst - may corrupt data later).

---

### Improving dayOfYear to Fail Fast

The current function might return a wrong answer if arguments are swapped (e.g., passing 2019 as a month).

**Improvement Options:**
* Change `month` to a string or enum (Static checking).
* Add a check: `if (month < 1 || month > 12) throw new Error(...)` (Dynamic checking).

---

### Avoid Magic Numbers

**Magic Number:** A constant that appears without explanation.

**Reasons to avoid them:**
* **Readability:** `FEBRUARY` is clearer than `2`.
* **Ready for Change:** Constants might change (e.g., precision of $\pi$).
* **Hand-computation:** Don't use "59" if it's the sum of 31 + 28; use a named constant or calculation.

---

### Constants as Data

If you have many magic numbers, store them in a data structure.

* Instead of hardcoding month lengths in an if statement, use a list or map.
* **Benefit:** Makes the code more DRY and easier to understand.

---

### Magic Strings

Strings can also be "magic" and risky.

**Example:** `pluralize('dog', 'English')`.

* **Problem:** If a client typos `'Engish'`, the code won't catch it at compile-time (Not SFB).
* **Better:** Use Enums or specific types to catch errors statically.

---

### One Purpose for Each Variable

Don't reuse variables or parameters.
Reusing `dayOfMonth` to store the final result of the function is confusing to the reader.

**Advice:** Introduce variables freely with good names; use `const` whenever possible to prevent reassignment.

---

### Smelly Example #2: leap

```typescript
function leap(y: number): boolean {
    let tmp = y.toString();
    if (tmp[1] === '1' || tmp[1] === '3' ...) {
        if (tmp[2] === '2' || tmp[2] === '6') return true;
        else return false;
    }
    // ...
}
```

**Issues:** String manipulation for math, logic bugs (doesn't handle 3-digit years), and heavy use of magic numbers.

---

### Use Good Names

Avoid `tmp`, `temp`, or `data`.

**Naming Conventions:**
* **Classes:** Capitalized (e.g., `Payment`).
* **Variables/Functions:** camelCase (e.g., `isLeapYear`, `secondsPerDay`).
* **Global Constants:** ALL_CAPS (e.g., `MAX_SIZE`).
* **Functions:** Use verb phrases; **Variables:** Use noun phrases.

---

### Naming Tips

* **Avoid abbreviations:** `message` is better than `msg`.
* **Avoid single-letter names:** Except for coordinates (`x`, `y`) or loop indices (`i`, `j`).
* **Self-Documentation:** A good name can often replace a comment.

---

### Whitespace and Punctuation

* **Indentation:** Use consistent spaces, never tabs (tabs look different in different editors).
* **Line Punctuation:** Always use semicolons at the end of statements in TypeScript.
* **Braces:** Always use curly braces for `if`, `while`, and `for` blocks, even for single lines.

---

### Readability with Spaces

* Put spaces around binary operators (`===`, `||`).
* Align code to make similarities pop out, which can reveal opportunities to DRY out the code.

---

### Smelly Example #3: countLongWords

```typescript
let LONG_WORD_LENGTH = 5;
let longestWord;

function countLongWords(text: string): void {
    let words = text.split(' ');
    if (words.length === 0) { console.log("0"); return; }
    // ... logic ...
    console.log(n);
}
```

**The Smell:** Uses global variables and prints results instead of returning them.

---

### Don't Use Global Variables

**Global Variable:** A name whose value can change and is accessible from anywhere.

* **The Danger:** They create hidden dependencies and make it hard to localize bugs.
* **Exceptions:** Global constants (declared with `const` and an immutable type) are fine.

---

### Kinds of Variables

In snapshot diagrams, we distinguish:
* **Local Variables:** Inside a function; exist only during the call.
* **Instance Variables:** Inside an object; exist as long as the object is accessible.
* **Global Variables:** Outside any function; exist for the life of the program.

---

### Functions Should Return Results

Avoid `console.log` in logic: It makes the code not ready for change (RFC).

* **The Problem:** If another part of the program needs the value, it can't "read" the console.
* **Rule:** Lower-level parts of a program should take parameters and return results.

---

### Avoid Special-Case Code

Resist the temptation to handle parameters like `0` or empty strings with unique `if` statements.

* **Why?** General-case code is often shorter, safer, and covers special cases naturally.
* **Performance Trap:** Don't optimize for special cases unless you have evidence it matters.

---

### Code at the Right Length

* **Avoid Monoliths:** Don't write thousands of lines in one sequence.
* **Function Length:** Ideally at most one "page" so the reader sees the whole logic at once.
* **Line Length:** 70-100 characters to avoid horizontal scrolling.
* **Paragraphs:** Break logic into small groups of lines separated by whitespace.

---

### Refactoring

**Definition:** Changing code to improve its SFB/ETU/RFC properties without changing what it does.

**Process for Safe Refactoring:**
1. Make small steps.
2. Use static checking to find all call sites of a function you modified.
3. Run tests after every single change.

---

### Refactoring Tips

* Comment out old code temporarily while writing the new version to compare logic.
* Commit frequently to version control after each small working step.
* Delete old code once the new version passes tests; don't leave dead code.

---

### Summary and the Big Three

* **Safe from Bugs:** Review finds defects; DRY prevents bug propagation; Fail Fast catches errors early.
* **Easy to Understand:** Good names, whitespace, and useful comments make code readable for others.
* **Ready for Change:** Returning results (not printing) and DRYing code makes it adaptable to new requirements.

<!-- note: After a long day of code review, remember that another proven software practice is sleep. -->
