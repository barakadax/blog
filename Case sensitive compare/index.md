# Case sensitive compare
- [The problem](#the-problem)
- [The solution](#the-solution)
  - [C#](#c)
  - [Python](#python)
- [Sources](#sources)

## The problem

Many programmers encounter in the programming experience the need to compare two strings,
Most cases a lot of programmers will just lower or upper case the strings and then compare them,
This is not the best solution because it will not work for non-English characters,

For example `;` is not equal to `;`, even through they look the same the first one unicode is `U+003B` and the second is `U+037E`,
Another famous example in operating system is different types of English keyboard, US English, UK english etc,
In Turkish you have 4 types of I, both upper and lower with dotted and dotless, their unicode is different but when reading and mostly depends on the font they will look alike.

For both examples converting to upper or lower case won't work,

## The solution

Many programming languages provide ways to compare strings with solution to this problem,

### C#
.net contains culture information, if you know what culture the input comes from the best way to compare is:
```shell
using System;
using System.Globalization;
					
public class Program
{
	public static void Main()
	{
		string turkishUpperI = "İ";
		string englishLowerI = "i";
		
		var trComparer = StringComparer.Create(new CultureInfo("tr-TR"), true);
		bool isTurkishEqual = trComparer.Equals(turkishUpperI, englishLowerI);
		
		Console.WriteLine($"Specific Turkish Comparer: {isTurkishEqual}"); // True
	}
}
```

If you would want to compare the semicolon and greek question mark you would use:
```shell
using System;
using System.Globalization;
					
public class Program
{
	public static void Main()
	{
		string semicolon = ";";
		string greekQuestionMark = ";";
		
		bool isEqualCurrentCultureIgnoreCase = string.Equals(semicolon, greekQuestionMark, StringComparison.CurrentCultureIgnoreCase);
		
		Console.WriteLine($"Current Culture Comparer: {isEqualCurrentCultureIgnoreCase}"); // True
	}
}
```

### Python
Python doesn't contain language specific string comparison, I recommend using `unidecode` library for this purpose.
```shell
from unidecode import unidecode

unidecode("İ").casefold() == unidecode("i").casefold() # True
```

Semicolon compare with Greek question mark:
```shell
import unicodedata
char1 = ";"
char2 = ";"
char1 == char2 # False

normalized1 = unicodedata.normalize('NFKC', char1)
normalized2 = unicodedata.normalize('NFKC', char2)
normalized1 == normalized2 # True
```

## Sources

- [The Turkish I problem](https://haacked.com/archive/2012/07/05/turkish-i-problem-and-why-you-should-care.aspx/)
- [Dotted and dotless I in computing](https://en.wikipedia.org/wiki/Dotted_and_dotless_I_in_computing)
- [C# StringComparer](https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparer?view=net-10.0)
- [C# StringComparison](https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparison?view=net-10.0)
- [Python Pip Unidecode package](https://pypi.org/project/Unidecode/)
- [Python unicodedata](https://docs.python.org/3/library/unicodedata.html)
