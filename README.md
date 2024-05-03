# JetBrains Internship Assignment - Solution Summary

There is already an existing method called `isSyntheticRecordMethod` in the `ClassWriter` class. It returns `true` if the method to be decompiled is one of `equals`, `hashCode` or `toString` that Java generates for records automatically. Without any changes from my side, these methods are not shown in the FernFlower output. I decided to implement my solution in this method as well. 

First, I access all fields an put them in a list. Preserving the order is important to check the constructor later on. 

If the method is a constructor (name `<init>`), we need to build a string containing zero or more assignments in the form `this.x = x;` for each field `x`. I then simply make sure that the built string indeed matches the contents of the method.

If the method is named the same as the variable, e.g. `x()`, then I check that it's contents is `return this.x;`, i.e. it's a simple accessor method.

For the bonus part, I added a new constant called `COMPACT_RECORDS_OUTPUT` and a CLI option `-cro`. By default it's on, so the output is without the canonical constructor and accessor methods. If it's off, it works exactly as before my changes.

Finally, I added a single test called `TestRecordPartial` which has another constructor explicitly written in the record. My code successfully preserves that constructor when `cro=1`. I found other existing tests sufficient for testing my code. I only had remove respective methods from the expected output files.

### About Fernflower

Fernflower is the first actually working analytical decompiler for Java and 
probably for a high-level programming language in general. Naturally it is still 
under development, please send your bug reports and improvement suggestions to the
[issue tracker](https://youtrack.jetbrains.com/newIssue?project=IDEA&clearDraft=true&c=Subsystem+Decompiler).

### Licence

Fernflower is licenced under the [Apache Licence Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

### Running from command line

`java -jar fernflower.jar [-<option>=<value>]* [<source>]+ <destination>`

\* means 0 or more times\
\+ means 1 or more times

\<source>: file or directory with files to be decompiled. Directories are recursively scanned. Allowed file extensions are class, zip and jar.
          Sources prefixed with -e= mean "library" files that won't be decompiled, but taken into account when analysing relationships between 
          classes or methods. Especially renaming of identifiers (s. option 'ren') can benefit from information about external classes.          

\<destination>: destination directory 

\<option>, \<value>: a command-line option with the corresponding value (see "Command-line options" below).

##### Examples:

`java -jar fernflower.jar -hes=0 -hdc=0 c:\Temp\binary\ -e=c:\Java\rt.jar c:\Temp\source\`

`java -jar fernflower.jar -dgs=1 c:\Temp\binary\library.jar c:\Temp\binary\Boot.class c:\Temp\source\`

### Command-line options

With the exception of mpm and urc the value of 1 means the option is activated, 0 - deactivated. Default 
value, if any, is given between parentheses.

Typically, the following options will be changed by user, if any: hes, hdc, dgs, mpm, ren, urc 
The rest of options can be left as they are: they are aimed at professional reverse engineers.

- rbr (1): hide bridge methods
- rsy (0): hide synthetic class members
- din (1): decompile inner classes
- dc4 (1): collapse 1.4 class references
- das (1): decompile assertions
- hes (1): hide empty super invocation
- hdc (1): hide empty default constructor
- dgs (0): decompile generic signatures
- ner (1): assume return not throwing exceptions
- den (1): decompile enumerations
- rgn (1): remove getClass() invocation, when it is part of a qualified new statement
- lit (0): output numeric literals "as-is"
- asc (0): encode non-ASCII characters in string and character literals as Unicode escapes
- bto (1): interpret int 1 as boolean true (workaround to a compiler bug)
- nns (0): allow for not set synthetic attribute (workaround to a compiler bug)
- uto (1): consider nameless types as java.lang.Object (workaround to a compiler architecture flaw)
- udv (1): reconstruct variable names from debug information, if present
- ump (1): reconstruct parameter names from corresponding attributes, if present
- rer (1): remove empty exception ranges
- fdi (1): de-inline finally structures
- mpm (0): maximum allowed processing time per decompiled method, in seconds. 0 means no upper limit
- ren (0): rename ambiguous (resp. obfuscated) classes and class elements
- urc (-): full name of a user-supplied class implementing IIdentifierRenamer interface. It is used to determine which class identifiers
  
           should be renamed and provides new identifier names (see "Renaming identifiers")
- inn (1): check for IntelliJ IDEA-specific @NotNull annotation and remove inserted code if found
- lac (0): decompile lambda expressions to anonymous classes
- nls (0): define new line character to be used for output. 0 - '\r\n' (Windows), 1 - '\n' (Unix), default is OS-dependent
- ind: indentation string (default is 3 spaces)
- crp (0): use record patterns where it is possible
- cps (0): use switch with patterns where it is possible 
- log (INFO): a logging level, possible values are TRACE, INFO, WARN, ERROR

### Renaming identifiers

Some obfuscators give classes and their member elements short, meaningless and above all ambiguous names. Recompiling of such
code leads to a great number of conflicts. Therefore it is advisable to let the decompiler rename elements in its turn, 
ensuring uniqueness of each identifier.

Option 'ren' (i.e. -ren=1) activates renaming functionality. Default renaming strategy goes as follows:

- rename an element if its name is a reserved word or is shorter than 3 characters
- new names are built according to a simple pattern: (class|method|field)_\<consecutive unique number>  
  You can overwrite this rules by providing your own implementation of the 4 key methods invoked by the decompiler while renaming. Simply 
  pass a class that implements org.jetbrains.java.decompiler.main.extern.IIdentifierRenamer in the option 'urc'
  (e.g. -urc=com.example.MyRenamer) to Fernflower. The class must be available on the application classpath.

The meaning of each method should be clear from naming: toBeRenamed determine whether the element will be renamed, while the other three
provide new names for classes, methods and fields respectively.  
