# Case sensitive compare
- [The problem](#the-problem)
  - [Linguistic Casing Differences (The Turkish I Problem)](#linguistic-casing-differences-the-turkish-i-problem)
  - [Unicode Homoglyphs and Collation Equivalence](#unicode-homoglyphs-and-collation-equivalence)
- [The solution](#the-solution)
  - [C#](#c)
    - [Ordinal Comparison (OrdinalIgnoreCase)](#ordinal-comparison-ordinalignorecase)
    - [Culture-Specific Linguistic Comparison](#culture-specific-linguistic-comparison)
    - [Homoglyphs and Collation Traps](#homoglyphs-and-collation-traps)
  - [Python](#python)
    - [Case-Insensitive Comparison with `casefold()`](#case-insensitive-comparison-with-casefold)
    - [Unicode Normalization with `unicodedata`](#unicode-normalization-with-unicodedata)
    - [Linguistic Comparison in Python](#linguistic-comparison-in-python)
- [Sources](#sources)

## The problem

Many programmers encounter the need to compare two strings case-insensitively.
In most cases, a naive approach is to convert both strings using `.ToLower()` or `.ToUpper()` and then compare them with the standard equality operator (`==`).

This approach introduces subtle bugs and security vulnerabilities when dealing with internationalization, Unicode homoglyphs, or culture-specific linguistic rules.

### Linguistic Casing Differences (The Turkish I Problem)

In Latin-based alphabets like English, casing is straightforward: the lowercase of `I` (U+0049) is `i` (U+0069), and the uppercase of `i` is `I`.

In Turkish and Azerbaijani, however, the alphabet has two distinct versions of the letter I:
- **Dotted I**: Lowercase `i` (U+0069) $\leftrightarrow$ Uppercase `İ` (U+0130, Latin Capital Letter I with Dot Above).
- **Dotless I**: Lowercase `ı` (U+0131, Latin Small Letter Dotless I) $\leftrightarrow$ Uppercase `I` (U+0049).

They represent different vowel sounds and change the meaning of words.
If code runs on a machine set to the Turkish culture (`tr-TR`):
- `"file".ToUpper()` becomes `"FİLE"` (with a dot), not `"FILE"`.
- `"FILE".ToLower()` becomes `"fıle"` (dotless), not `"file"`.

Consequently, programmatic checks like `if (filename.ToUpper() == "FILE.TXT")` or `if (header.ToLower() == "content-type")` will fail on Turkish machines.
Conversely, comparing Turkish text using standard English/invariant casing rules will fail to match `İ` with `i`.

### Unicode Homoglyphs and Collation Equivalence

A related issue occurs when characters look identical (homoglyphs) but represent different Unicode code points.

For example, the standard ASCII semicolon `;` (`U+003B`) and the Greek question mark `;` (`U+037E`) look identical in most fonts.
While they have distinct code points and neither is a cased letter, Unicode collation tables used by culture-aware comparison algorithms classify them as equivalent.
If a system compares them using culture-sensitive rules instead of binary byte/ordinal rules, it may inadvertently treat them as identical.

---

## The solution

Modern programming platforms provide explicit mechanisms to separate **programmatic/ordinal** string comparisons from **linguistic/cultural** string comparisons.

### C#

In .NET, Microsoft recommends specifying a `StringComparison` enum value explicitly whenever comparing strings or using `StringComparer` with collections.

#### Ordinal Comparison (OrdinalIgnoreCase)

For programmatic identifiers, configuration keys, file paths, URLs, XML/JSON tags, and protocol headers, you should almost always use `StringComparison.OrdinalIgnoreCase` (or `StringComparer.OrdinalIgnoreCase`).

Ordinal comparison performs a byte-by-byte comparison using simple uppercase invariant mapping:
- Fast and deterministic.
- Immune to the current machine's locale or culture settings.
- Safe from unexpected linguistic collation equivalences.

```csharp
using System;

public class Program
{
    public static void Main()
    {
        // Programmatic key comparison: safe across all system locales
        string configKey1 = "API_KEY";
        string configKey2 = "api_key";
        bool isKeyEqual = string.Equals(configKey1, configKey2, StringComparison.OrdinalIgnoreCase);
        Console.WriteLine($"OrdinalIgnoreCase (API_KEY == api_key): {isKeyEqual}"); // True

        // Case-insensitive hash collections using StringComparer.OrdinalIgnoreCase
        var headers = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
        {
            "Authorization"
        };
        Console.WriteLine($"Contains 'authorization': {headers.Contains("authorization")}"); // True
    }
}
```

#### Culture-Specific Linguistic Comparison

When comparing user-facing strings where the human linguistic rules of a particular language must apply, use `StringComparison.CurrentCultureIgnoreCase` or create a culture-specific `StringComparer`:

```csharp
using System;
using System.Globalization;

public class Program
{
    public static void Main()
    {
        string turkishUpperI = "İ"; // U+0130
        string latinLowerI = "i";   // U+0069

        // 1. Using specific Turkish culture rules:
        var trComparer = StringComparer.Create(new CultureInfo("tr-TR"), ignoreCase: true);
        bool isTurkishEqual = trComparer.Equals(turkishUpperI, latinLowerI);
        Console.WriteLine($"Turkish Comparer (İ == i): {isTurkishEqual}"); // True

        // 2. Using OrdinalIgnoreCase:
        // Returns False because their binary code points (U+0130 and U+0069) differ
        bool isOrdinalEqual = string.Equals(turkishUpperI, latinLowerI, StringComparison.OrdinalIgnoreCase);
        Console.WriteLine($"OrdinalIgnoreCase (İ == i): {isOrdinalEqual}"); // False
    }
}
```

#### Homoglyphs and Collation Traps

Comparing the ASCII semicolon `;` (`U+003B`) and the Greek question mark `;` (`U+037E`):

```csharp
using System;

public class Program
{
    public static void Main()
    {
        string semicolon = ";";          // U+003B
        string greekQuestionMark = ";";  // U+037E

        // Culture-aware comparison performs linguistic collation and treats them as equivalent:
        bool isEqualCurrentCulture = string.Equals(semicolon, greekQuestionMark, StringComparison.CurrentCultureIgnoreCase);
        Console.WriteLine($"CurrentCultureIgnoreCase: {isEqualCurrentCulture}"); // True

        // Ordinal comparison compares binary code points and treats them as distinct characters:
        bool isEqualOrdinal = string.Equals(semicolon, greekQuestionMark, StringComparison.OrdinalIgnoreCase);
        Console.WriteLine($"OrdinalIgnoreCase: {isEqualOrdinal}"); // False
    }
}
```

> [!TIP]
> For internal logic, parsers, security tokens, and data lookups, always prefer `StringComparison.OrdinalIgnoreCase` to prevent unintended collation matches.
> Reserve `CurrentCultureIgnoreCase` strictly for displaying or sorting localized UI text.

---

### Python

#### Case-Insensitive Comparison with `casefold()`

For case-insensitive string matching, Python provides `str.casefold()` (PEP 3138), which is more thorough than `str.lower()` because it implements full Unicode casefolding.
For example, it folds the German lowercase sharp s (`ß`) into `"ss"`:

```python
# Standard case-insensitive comparison
filename1 = "FILE.TXT"
filename2 = "file.txt"
print(filename1.casefold() == filename2.casefold())  # True

# German sharp s (eszett)
german_word1 = "straße"
german_word2 = "STRASSE"
print(german_word1.casefold() == german_word2.casefold())  # True
```

#### Unicode Normalization with `unicodedata`

To handle homoglyphs, compatibility characters, or decomposed Unicode characters (like comparing `;` with `;`), Python's standard library includes the `unicodedata` module.
Normalizing strings to `NFKC` (Compatibility Decomposition, followed by Canonical Composition) maps compatibility variants to their standard base forms:

```python
import unicodedata

char1 = ";"  # ASCII Semicolon (U+003B)
char2 = ";"  # Greek Question Mark (U+037E)

print(char1 == char2)  # False

# Normalize to compatibility form NFKC
normalized1 = unicodedata.normalize("NFKC", char1)
normalized2 = unicodedata.normalize("NFKC", char2)
print(normalized1 == normalized2)  # True
```

#### Linguistic Comparison in Python

Python's built-in `casefold()` follows Unicode default casing algorithms, so `'İ'.casefold()` evaluates to `'i\u0307'` and does not match `'i'` directly.

- For locale-aware string comparison adhering to specific language rules (like Turkish), use Python's standard library `locale.strcoll()` or the industry-standard `PyICU` library (`icu.Collator`).
- Be cautious with libraries like `unidecode`: while useful for creating ASCII URL slugs, `unidecode` performs lossy transliteration that collapses distinct letters (such as Turkish dotless `ı` and dotted `i` both into `"i"`), which can conflate entirely different words.

---

## Sources

- [Microsoft: Best Practices for Comparing Strings in .NET](https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings)
- [C# StringComparison Enum](https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparison?view=net-10.0)
- [C# StringComparer Class](https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparer?view=net-10.0)
- [The Turkish I problem and why you should care](https://haacked.com/archive/2012/07/05/turkish-i-problem-and-why-you-should-care.aspx/)
- [Dotted and dotless I in computing (Wikipedia)](https://en.wikipedia.org/wiki/Dotted_and_dotless_I_in_computing)
- [Python str.casefold documentation](https://docs.python.org/3/library/stdtypes.html#str.casefold)
- [Python unicodedata documentation](https://docs.python.org/3/library/unicodedata.html)
