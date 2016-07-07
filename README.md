# haxe-strings - [StringTools](http://api.haxe.org/StringTools.html) on steroids.

1. [What is it?](#what-is-it)
1. [The `Strings` utility class](#strings-class)
1. [The Spell Checker](#spell-checker)
1. [The string collection classes](#string-collections)
1. [The `Paths` utility class](#paths-class)
1. [Installation](#installation)
1. [Using the latest code](#latest)
1. [License](#license)


## <a name="what-is-it"></a>What is it?

A [haxelib](http://lib.haxe.org/documentation/using-haxelib/) for consistent cross-platform UTF-8 string manipulation. 
It has been extensively unit tested (over 1,200 individual test cases) on the targets C++, C#, Flash, Neko, Java, JavaScript, PHP, and Python.

The classes are under the package `hx.strings`.


## <a name="strings-class"></a>The `Strings` utility class

The [hx.strings.Strings](https://github.com/vegardit/haxe-strings/blob/master/src/hx/strings/Strings.hx) class provides handy utility methods for string manipulations.

It also contains improved implementations of functions provided by Haxe's [StringTools](http://api.haxe.org/StringTools.html) class.

Methods ending with the letter `8` (e.g. `length8()`, `indexOf8()`, `toLowerCase8()`) are improved versions of similar methods
provided by Haxe's [String](http://api.haxe.org/String.html) class, offering UTF-8 support and consistent behavior across all platforms.

The [hx.strings.Strings](https://github.com/vegardit/haxe-strings/blob/master/src/hx/strings/Strings.hx) class can also be used as a [static extension](http://haxe.org/manual/lf-static-extension.html).

    
### <a name="strings-examples"></a>Some examples

```haxe
using hx.strings.Strings; // augment all Strings with new functions

class Test {

    static function main() {
        // Strings are extended:
        "".isEmpty();      // returns true
        "   ".isBlank();   // returns true
        "123".isDigits();  // returns true
        "a".repeat(3);     // returns "aaa"
        "abc".reverse();   // returns "cba"

        // Int's are extended too:
        32.toChar().isSpace();       // returns true
        32.toChar().toString();      // returns " "
        32.toChar().isAscii();       // returns true
        6000.toChar().isAscii();     // returns false
        6000.toChar().isUTF8();      // returns true
        74.toHex();                  // returns "4A"
        
        // all functions are null-safe:
        var nullString:String = null;
        nullString.isWhiteSpace();    // returns true
        nullString.length8();         // returns 0
        nullString.contains("cat");   // returns false

        // all methods support UTF-8 on all platforms:
        "кот".toUpperCase8();         // returns "КОТ"
        "кот".toUpperCaseFirstChar(); // returns "Кот"
        "はいはい".length8();          // returns 4

        // ANSI escape sequence processing:
        "\x1B[1;33mHello World!\x1B[0m".ansiToHtml();  // returns '<span style="color:yellow;font-weight:bold;">Hello World!</span>'
        "\x1B[1mHello World!\x1B[0m".removeAnsi();     // returns "Hello World!"

        // case formatting:
        "look at me".toUpperCamel();       // returns "LookAtMe"
        "MyCSSClass".toLowerCamel();       // returns "myCSSClass"
        "MyCSSClass".toLowerHyphen();      // returns "my-css-class"
        "MyCSSClass".toLowerUnderscore();  // returns "my_css_class"
        "myCSSClass".toUpperUnderscore();  // returns "MY_CSS_CLASS"

        // ellipsizing strings:
        "The weather is very nice".ellipsizeLeft(20);    // returns "The weather is ve..."
        "The weather is very nice".ellipsizeMiddle(20);  // returns "The weath...ery nice"
        "The weather is very nice".ellipsizeRight(15);   // returns "...ther is very nice"

        // string differences:
        "It's green".diffAt("It's red"); // returns 5
        "It's green".diff("It's red");   // returns { left: 'green', right: 'red', pos: 5 }
        
        // hash codes:
        "Cool String".hashCode();       // generate a platform specific hash code
        "Cool String".hashCode(CRC32);  // generate a hash code using CRC32
        "Cool String".hashCode(JAVA);   // generate a hash code using the Java algorithm
        
        // cleanup:
        "/my/path/".removeLeading("/");       // returns "my/path/"
        "/my/path/".removeTrailing("/");      // returns "/my/path"
        "<i>So</i> <b>nice</b>".removeTags(); // returns "So nice"
    }
}
```

## <a name="spell-checker"></a>The Spell Checker

The package `hx.strings.spelling` contains an extensible spell checker implementation that is based on ideas outlined by Peter Norvig in his article [How to write a Spell Checker](http://www.norvig.com/spell-correct.html).

The `SpellChecker#correctWord()` method can for example be used to implement a Google-like "did you mean 'xyz'?" feature for a custom search engine.

Now let's do some spell checking...

```haxe
import hx.strings.spelling.checker.*;
import hx.strings.spelling.dictionary.*;
import hx.strings.spelling.trainer.*;

class Test {

    static function main() {
        /* 
         * first we use the English spell checker with a pre-trained dictionary 
         * that is bundled with the library:
         */
        EnglishSpellChecker.INSTANCE.correctWord("speling");  // returns "spelling"
        EnglishSpellChecker.INSTANCE.correctWord("SPELING");  // returns "spelling"
        EnglishSpellChecker.INSTANCE.correctWord("SPELLING"); // returns "spelling"
        EnglishSpellChecker.INSTANCE.correctWord("spell1ng"); // returns "spelling"
        EnglishSpellChecker.INSTANCE.correctText("sometinG zEems realy vrong!") // returns "something seems really wrong!"
        EnglishSpellChecker.INSTANCE.suggestWords("absance"); // returns [ "absence", "advance", "balance" ]
        
        /* 
         * let's check the pre-trained German spell checker
         */
        GermanSpellChecker.INSTANCE.correctWord("schreibweise");  // returns "Schreibweise"
        GermanSpellChecker.INSTANCE.correctWord("Schreibwiese");  // returns "Schreibweise"
        GermanSpellChecker.INSTANCE.correctWord("SCHREIBWEISE");  // returns "Schreibweise"
        GermanSpellChecker.INSTANCE.correctWord("SCHRIBWEISE");   // returns "Schreibweise"
        GermanSpellChecker.INSTANCE.correctWord("Schre1bweise");  // returns "Schreibweise"
        GermanSpellChecker.INSTANCE.correctText("etwaz kohmische Aepfel ligen vör der Thür"); // returns "etwas komische Äpfel liegen vor der Tür"
        GermanSpellChecker.INSTANCE.suggestWords("Sistem");       // returns[ "System", "Sitte", "Sitten" ]

        /*
         * now we train our own dictionary from scratch
         */
        var myDict = new InMemoryDictionary("English");
        // download some training text with good vocabular
        var trainingText = haxe.Http.requestUrl("http://www.norvig.com/big.txt");
        // populating the dictionary might take a while:
        EnglishDictionaryTrainer.INSTANCE.trainWithString(myDict, trainingText);
        // let's use the trained dictionary with a spell checker
        var mySpellChecker = new EnglishSpellChecker(myDict);
        mySpellChecker.INSTANCE.correctWord("speling");  // returns "spelling"
        
        // since training a dictionary can be quite time consuming, we save
        // the analyzed words and their popularity/frequency to a file
        myDict.exportWordsToFile("myDict.txt");
        
        // the word list can later be loaded using
        myDict.loadWordsFromFile("myDict.txt");
    }
}
```


## <a name="string-collections"></a>The String collection classes

The package `hx.strings.collection` contains some useful collection classes for Strings.

1. `StringSet` is a collection of unique strings. Each string is guaranteed to only exists once within the collection.

   ```haxe
   var set = new hx.strings.collection.StringSet();
   set.add("a");
   set.add("a");
   set.add("b");
   set.add("b");
   // at this point the set only contains two elements: one 'a' and one 'b'
   ```
   
2. `SortedStringSet` is a sorted collection of unique strings. A custom comparator can be provided for using different sorting algorithm.

3. `StringTreeMap` is a map that is sorted by there keys (which are of type String).


## <a name="paths-class"></a>The `Paths` utility class

The [hx.strings.Strings](https://github.com/vegardit/haxe-strings/blob/master/src/hx/strings/Paths.hx) class provides
utility methods for string manipulations related to local filesystem paths. 
It can be seen as an improved and extended version of the built-in `haxe.io.Path` class. 
In contrast to functions provided by `haxe.io.Path`, there are no situations where the result of a function is unspecified.

```haxe
import hx.strings.Paths;

class Test {

    static function main() {
        Paths.addDirectorySeparator("/dir");      // returns "/dir/"
        Paths.addDirectorySeparator("C:\\dir");   // returns "C:\dir\"
        Paths.addDirectorySeparator("C:");        // returns "C:\"
        Paths.addDirectorySeparator("dir");       // returns "dir/"
        Paths.addDirectorySeparator("dir", WIN);  // returns "dir\"
        
        Paths.basename("/dir/file.txt");      // returns "file.txt"
        Paths.basename("C:\\dir\\file.txt");  // returns "file.txt"
        Paths.basename("/dir//");             // returns "dir"
        
        Paths.basenameWithoutExtension("/dir/file.txt");      // returns "file.txt"
        Paths.basenameWithoutExtension("C:\\dir\\file.txt");  // returns "file.txt"
        Paths.basenameWithoutExtension("/dir//");             // returns "dir"
        Paths.basenameWithoutExtension("/dir/..");            // returns ".."
        
        Paths.dirname("C:\\Users\\Default\\Desktop\\");       // returns "C:\Users\Default"
        
        Paths.ellipsize("C:\\Users\\Default\\Desktop\\", 15); // returns "C:\...\Desktop"

        Paths.extension("dir.cfg/file.txt");  // returns "txt"
        
        // use glob pattern matching:
        Paths.globToEreg("src/**/*.hx").match("src/haxe/strings/Char.hx");            // returns true
        Paths.globToEreg("assets/**/*.{js,css}").match("assets/theme/dark/dark.css"); // returns true
        Paths.globToEreg("SystemOut[0-9].log").match("SystemOut1.log");               // returns true
        Paths.globToEreg("SystemOut[!0-9].log").match("SystemOut1.log");              // returns false

        Paths.isAbsolute("/");                       // returns true
        Paths.isAbsolute("C:");                      // returns true
        Paths.isAbsolute("\\\\winserver\\dir");      // returns true
        Paths.isAbsolute("../dir");                  // returns false

        // very convenient path joining:
        Paths.join("dir", "test.txt");               // returns "dir/test.txt"
        Paths.join("dir1\\..\\dir2", "dir3");        // returns "dir2\\dir3"
        Paths.join("dir1\\..\\dir2", "dir3", NIX);   // returns "dir2/dir3"
        Paths.join(["dir1\\dir2", "dir3", "dir4"]);  // returns "dir1\\dir2\\dir3\\dir4"
        Paths.join(["dir1/dir2", "dir3", "dir4"]);   // returns "dir1/dir2/dir3/dir4"
        
        Paths.normalize("C:\\dir1\\..\\dir2\\");               // returns "C:\dir2"
        Paths.normalize("\\\\\\server.local\\a\\b\\..\\c\\");  // returns "\\server.local\a\c"
    }
}
```

## <a name="installation"></a>Installation

1. install the library via haxelib using the command:
   ```
   haxelib install haxe-strings
   ```

2. use in your Haxe project

   * for [OpenFL](http://www.openfl.org/)/[Lime](https://github.com/openfl/lime) projects add `<haxelib name="haxe-strings" />` to your [project.xml](http://www.openfl.org/documentation/projects/project-files/xml-format/)
   * for free-style projects add `-lib haxe-strings`  to `your *.hxml` file or as command line option when running the [Haxe compiler](http://haxe.org/manual/compiler-usage.html)


## <a name="latest"></a>Using the latest code

### Using `haxelib git`

```
haxelib git haxe-strings https://github.com/vegardit/haxe-strings master
```

###  Using Git

1. check-out the master branch
    ```
    git clone https://github.com/vegardit/haxe-strings --branch master --single-branch D:\haxe-projects\haxe-strings
    ```

2. register the development release with haxe
    ```
    haxelib dev haxe-strings D:\haxe-projects\haxe-strings
    ```
    
###  Using Subversion

1. check-out the trunk
    ```
    svn checkout https://github.com/vegardit/haxe-strings/trunk D:\haxe-projects\haxe-strings
    ```

2. register the development release with haxe
    ```
    haxelib dev haxe-strings D:\haxe-projects\haxe-strings
    ```

    
## <a name="license"></a>License

All files are released under the [Apache License 2.0](https://github.com/vegardit/haxe-strings/blob/master/LICENSE.txt).
