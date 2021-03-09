# Kotcell
## Excel overview
**Excel** is known as a powerful tool for computations and analyzing data using its table representation. It has lots of functionality out of the box and has a very low entry threshold. But let's look at it from a programmer's point of view because it is widely used in many fields of activity where programming is also used. Its advantages are already listed, so let's take a look at the list of disadvantages.

### Autodetecting of type by string representation.
This may seem helpful when you don't need to specify the type of value inside the cell, but that leads to unexpected conversations:
* `1.01` can be recognised either as a decimal number or a date (1<sup>st</sup> of January)
* [Scientists rename human genes to stop Microsoft Excel from misreading them as dates](https://www.theverge.com/2020/8/6/21355674/human-genes-rename-microsoft-excel-misreading-dates)

The last example causes the following consequences:
* Is there any guarantee that all vital things that are based on Excel computations are safe and don't contain similar bugs? No, they don't.
* The tool stops being just a tool. It makes specialists correct the activity area to make the tool possible to use. And this thing is not connected with something fundamental for this area (representation models). It is connected with chosen names and their wrong recognitions with this tool. This is not what is expected from the convenient programming tool.

### Everything is a `String` or other of other built-in types.
If you consider a cell equation to be a function that computes something, then the return type in Excel is always `String` or other built-in (may be autodetected as said above). Most application areas have a lot higher level of abstraction than built-in primitives. Using proper abstractions instead of primitives is highly recommended in programming. However, there is no such feature in **Excel**. Such a feature would:
* Help to think in terms of the area.
* Encapsulate code inside.
* Significantly simplify usage of the cell from other cells by escaping deserializing from string or collecting data from different cells.
* Let specialize representation: string representation of the value is no longer a value inside, it is just a representation of the value in some readable format. Furthermore, we can provide some overridable by cell method `toCellRepresentation()` that sets content to the result of `toString()` by default. This would let user easily create custom cell fillers (like inline tex-like equations, inline diagrams, dynamic picture viewers and other complex custom widgets)

### Partial translation
A very weird strategy was chosen to support other locales: translating functions but not cell letters. This means that if my local layout is different from the English one, I have to switch between layouts about 10 times per short equation. Accidental switching makes typing equations even harder. So it would be better to translate either everything including columns naming or only documentation.

Example: `=СЧЁТЕСЛИМН(A2:A7;"<6";A2:A7;">1")` is `=COUNTIFS(A2:A7;"<6";A2:A7;">1")` in Engish.

### Finite ranges
All ranges must have a finite size in **Excel**. But that can be bad if you want to be able to proceed with any number of items. For example, You have some numbers in column `A`, and you want to have them powered with 2 in column `B`. You have to know a maximal number of elements in the first column by the moment of creating the equation for cells in column `B`.

### Predicates
Predicates (an example is above) have very ugly and not extendable syntax.
* They are ugly because they are specified just as string literals with no validation.
* They are not extendable because they only work in this case: if I want some more complex condition (or just `num -> num < 3 || num > 5`), I have to use intermediate cells.

A lot better way is to use lambda functions.

### Oneliners
All cell formulas are expressions with no intermediate named values inside the cell, that is why you have to use additional cells to contain intermediate variables to make the main cell content readable. By the way, the `LET` function is going to be added to the stable release so the value of the point is a lot less now. However, such a function is still quite ugly.

### Extra cells usage
Extra cells are used as a workaround for the 2 previous problems.
But this leads to:
* Lack of incapsulation.
* Lack of ability to use the intermediate cells for some other purpose.

### Interop & libraries
The only interoperability between Excel and side libraries comes from macroses that are
* Written with unpopular VB.Net
* Unsafe
* Not as easy to create and call from cells as standard functions.

## Solution

My solution is making some computable cell-based notebook whose formula syntax is the syntax of some programming language.

### Requirements

#### Necessary:
* __Simplicity__

  A user of the app should be able to write a simple program (**Excel**-like formula) without knowledge of some programming-related stuff)
* __Brevity & Expressiveness__

  It must be not verbose: the formulas would become too complicated.
  They should be at least not longer than **Excel**'s ones in most cases.
  There should be no boilerplate for simple things.
  *This also includes having some helpful operators such as range-operators.*

#### Highly wanted:
* __Fast compilation (if needed)__

  Delay between printing a formula and its evaluation should be small (as it is in **Excel**)
* __Static typing__

  Type-safety is a guarantee that everything will continue working even if the conditional branch in some formula would be changed because of some data changing.

#### Wanted, but not necessary:
* __High speed__
  
  **Excel** computations are slow so high speed would be a good bonus.

------

Here is an approximate list of languages with their comparison in the context of the app.

**WARNING**: The following table is just a reasoned opinion of the author.


<table>
  <tr>
    <th>Name</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Simplicity&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Brevity&nbsp;&&nbsp;Expressiveness&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Safety&nbsp;&&nbsp;Static&nbsp;typing&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th>
    <th>High&nbsp;speed <i>(<a href='https://github.com/kostya/benchmarks'>Benchmark</a>)</i></th>
    <th>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fast&nbsp;compilation&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</th><th>Sum</th>
  </tr>
  <tr>
    <th>C++</th>
    <td align='center' valign='top'>
      <pre lang='diff'>- 0 points</pre>
      <p align='justify'>
        C++ is a very difficult (for non-programmers) language as it is system programming language.
      </p>
    </td>
    <td valign='top'>
      <pre lang='diff' align='center'>! 4 points</pre>
      <p align='justify'>
        C++ syntax is very verbose in some cases.
        Simple example is <a href="https://en.cppreference.com/w/cpp/utility/forward">perfect forwarding</a>:

```cpp
class B {
public:
    template<class T1, class T2, class T3>
    B(T1&& t1, T2&& t2, T3&& t3) :
        a1_{std::forward<T1>(t1)},
        a2_{std::forward<T2>(t2)},
        a3_{std::forward<T3>(t3)}
    {
    }
 
private:
    A a1_, a2_, a3_;
};
```
</p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 4 points</pre>
      <p align='justify'>
        In spite of a powerful type system, it is easy to get a memory leak in C++. Having undefined behaviour is also a big disadventage. You cannot guarantee that some code would behave the same on different platforms. That does not suit <b>Excell</b>-like sheets.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 10 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>- 0 points</pre>
      <p align='justify'>
        C++ is known for its extremely slow non-iterative compilation that is not suitible for computable notebooks. <i><a href='https://stackoverflow.com/questions/1062140/c-sharp-compilation-time-for-large-projects-compared-to-c'>Comparison with C# compilation</a></i>
      </p>
    </td>
    <th>18 points</th>
  </tr>
  <tr>
    <th>Python</th>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 10 points</pre>
      <p align='justify'>
        Python is one of the easiest languages to learn that takes control of both memory management and arithmetic overflows. That is why it is usually learnt as the first language or as the only one by people who need some programming skills for non-programming area.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
      <ul align='justify'>
        <li>Python has intuitive, simple and expressive syntax.</li>
        <li>However, lambda syntax is very verbose. That is quite important <u>for such app</u> where different simple code (such as <pre lang='csharp'>.Where(t => t > 0)</pre>) would be popular.</li>
        <li>Good point is that Python makes a user follow the correct indentation. However, that is possible to notify about bad ones in other languages. It is not built in the other languages, but the our context is the app.</li>
      </ul>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>- 3 points</pre>
      <p align='justify'>
        Python is a dynamicly typed language. There are type annotations, but they cannot be used in declaration of lambdas and polymorhic methods.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 4 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 10 points</pre>
      <p align='justify'>
        The most common Python implementations (CPython, PyPy) are interpretable so no compilation is needed.
      </p>
    </td>
    <th>35 points</th>
  </tr>
  <tr>
    <th>Java</th>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 7 points</pre>
      <p align='justify'>
        Java memory management is based on GC. All classes instances are references so are easy to be effectively taken as a function argument (without the necessity to think about reference type, moving, copying as in C++). That makes programming a lot easier for newbies like app users. Java uses OOP as the main paradigm. However, it may be a bit difficult for those who have no programming skills to deal with it.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>- 3 points</pre>
      <p align='justify'>
        Java is known as a very verbose language because of many reasons. Some of them are:
        <ul align='justify'>
          <li>Verbose getters and setters</li>
          <li>Checked exceptions<br/>Reality showed that this leads to lots of rethrowing of checked ones hidden in the unchecked ones. It makes the concept useless as checked exceptions are no more checked.</li>
          <li>No operator overloading<br/>Java's approach was to limitate it because the overloaded behaviour may be unexpected.<br/>However, that led to not ability to define (existing) operator even if its behaviour is well-known. And this leads to verbosity. Good example of such problem is BigInteger. It has no operators in java, so <pre>(a + b) * (a - b) * (2 a - b)</pre> becomes <pre lang ='java'>a.plus(b).multiply(a.subtract(b)).multiply(BigInteger.valueOf(2).multiply(a).minus(b))</pre> which is definetely verbose.<br/>This is an important example because cell index in the app is expected to be some <code lang='java'>BigInteger</code> as user may want to use just 2 cells <code>A1</code> and <code>A1000000000000000000000</code> and there is no reason to forbid it.</li>
          <li><i>(NO MORE VALID)</i> No record classes</li>
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 7 points</pre>
      <p align='justify'>
        Type system of Java is powerful enough for most of practical usages, including (probably, simple) ones that are needed in the computations of cell formulas.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 6 points</pre>
      <p align='justify'>
        Compilation even of simple formulas takes so much time that it is still not really significant for singular computations but maybe bad for a sequence of them. Does incremental compilation.
      </p>
    </td>
    <th>31 points</th>
  </tr>
  <tr>
    <th>Kotlin</th>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 7 points</pre>
      <p align='justify'>
        Same as Java
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 10 points</pre>
      <p align='justify'>
        Kotlin is compatible with Java but got rid of its disadvantages. It also contains lots of syntax sugar that simplifies coding. Lots of it is specific for Kotlin.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
      <p align='justify'>
        Same with Java + Null safety + Better Collection Interfaces Hierarchy
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 6 points</pre>
      <p align='justify'>
        Same as Java: <a href='https://habr.com/ru/company/badoo/blog/329026/'>proof</a>
      </p>
    </td>
    <th>39 points</th>
  </tr>
  <tr>
    <th>C#</th>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 7 points</pre>
      <p align='justify'>
        Same as Java
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
      <p align='justify'>
        Same as Kotlin (but Java compatibility). As Kotlin has some specific advantages, C# has its own ones.<br/>
        But there is a advantage of Kotlin that is important for <b>the app</b>: its lambda syntax is a good instrument to create DSLs. The app has lots of such domains: simple example is describing diagram structure when creating it manually.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
      <p align='justify'>
        Same with Java + Null safety
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 6 points</pre>
      <p align='justify'>
        Same as Java
      </p>
    </td>
    <th>37 points</th>
  </tr>
  <tr>
    <th>Haskell</th>
    <td align='center' valign='top'>
      <pre lang='diff'>- 3 points</pre>
      <p align='justify'>
        Haskell has automatic memory management too. But standard mathematical class types (Functors, Applicatives, Monads, Arrow, ...), advanced Hindley–Milner type system are supposed to be a bad choise for beginners.
      </p>
    </td>
    <td valign='top'>
      <pre align='center' lang='diff'>! 5 points</pre>
<p align='justify'>
        Haskell is a lazy functional language so some things are easily coded:
        <pre lang='haskell'>
primes = filterPrime [2..]
  where filterPrime (p:xs) =
          p : filterPrime [x | x <- xs, x `mod` p /= 0]
        </pre>
        But lots of algorithms are easily coded only in the imperative non-pure style (<a href="https://en.wikipedia.org/wiki/Sequential_minimal_optimization">SMO</a>, <a href="https://www.researchgate.net/publication/2295532_Lazy_Depth-First_Search_and_Linear_Graph_Algorithms_in_Haskell">DFS</a>). 
</p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 10 points</pre>
      <p align='justify'>
        The most powerful type system. It covers a lot more cases than those that can probably be met in this app.
      </p>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>! 6 points</pre>
    </td>
    <td align='center' valign='top'>
      <pre lang='diff'>+ 8 points</pre>
      <p align='justify'>
        Using interpreter GHCI instead of compiler GHC is suitable for the purpose. GHCI is fast but a bit slower than ones for Python. However, benchmark from the link above was done with GHC used.
      </p>
    </td>
    <th>32 points</th>
  </tr>
</table>

So the chosen language was Kotlin.
A solution to the compilation speed problem would be given in the corresponding paragraph of the article.

## Implementation

### Indexing
* Indexing is expected to be done with some `BigInteger`s as there should be no problems with the usage of cells with big indexes.
* This `BigInteger` oughts to support negative numbers as they can be used as a term for some other address.
* Operators should also support indexing by `Int`, `Long` instances because they are easier to write.
* `java.math.BigInteger` type shouldn't be used as the `BigInteger` itself because used implementation may change:
  * It has slow multiplication
  * It is JVM only
  So its wrapper should be used instead now.

### Infinity
* There are `Infinity` objects: `PositiveInfinity`, `NegativeInfinity`.
* They are used as arguments to create infinite ranges and progressions.
* `Infinity` should be used just as a pseudonym for either `PositiveInfinity` or `NegativeInfinity` object depending on the context.
* Each of the objects has its readable string representation (`∞`, `+∞`, `-∞`).
* Each of them has unary minus and unary plus operators. For the `Infinity` object it just specifies its type (`PositiveInfinity` or `NegativeInfinity`) as they behave for unsigned infinity in both mathematics and string representation.
* There should be `typealias Inf = Infinity` alias.

### Ranges
* As all numeric types, these `BigInteger`s have corresponding `Progression` and `Range` class.
* However, I expect these classes to be a bit wider than there `stdlib` analogues, they should:
  1. Support infinity ranges/progressions
  2. Consequently, be `Sequence` but not `Iterable` by default (be lazy).
  3. Have effective `size`, `contains`, `intersect`, `union` and other operations that have generalized implementation for `Iterable`s and `Sequence`s. They should have the same semantics when possible.
* `Range` instances, but not `Progression` ones need to be returned from the functions, so `Progression` constructor should be internal or even private, static abstract method `fromClosedRange` and `progression`/`range` functions should be used instead of it from externals of the `Progression` implementation.

### Regions
* Regions are expected to be only rectangles to make it easy to find intersections and cell owners. Also, other regions are rarely used in real life.
* Indexation is only via numbers, not letters. As it is easier to do from code and unifies column and row naming.
* However, there should be a `RegionSet` class that
  * Contains regions in some specific order.
  * Supports adding new ones
  * Removing ones
  * Comparing with other `RegionSet` instances
  * Iterating over regions inside it.
  * If some region is already contained by some of the regions, it shouldn't be inside. Example: There are two rectangles (vertical and horizontal lines inside)
    ```
    ▤▤
    ▤▤▥
    ▤▤▥	
    ▤▤
    ```
    and the rectangle to add is (marked with points inside)
    ```
    ▢▢
    ▢▣▣
    ▢▣▣
    ▢▢
    ```
    No rectangle contains the new one itself but their union does so we don't add the new rectangle.
* There is the following hierarchy of region classes: `Cell, Row, Column, Sheet, EmptyRegion : Region`.
* Each `Region` can be named.
* Each region can be  used to address relatively: `someRegion[Cell(23, 1)..Cell(35, 100)][region(56..76, 1)]`
* All they have internal constructors that are used in the `range` function whose return type is as most specific as possible *in runtime*.

  Example:
  ```kotlin
  (cell1..cell1) /* : Region*/ is Cell // true
  range(row1, column1..column3) /* : Region*/ is Cell // false
  ```

* `Cell` has properties `row` and `column`.
* Region constructor should be internal. An abstract function `region` should be used instead. It is done to make all cell-, row-, column-, sheet-like, empty regions be instances of the corresponding classes.
* Regions can be infinite
* Regions can be converted to `Sequence` or `Iterable` with row-by-row and column-by-column order.
* Regions have `width` and `height` properties.
* Regions should contain such (extension) methods of `Iterable`s and `Sequence`s that apply to them too.

### Region content
* Region content has explicit typing (the reason was noticed in the intro paragraph).
* One of the possible types is __Executable code__.
* You can specify its type and then input data that would be parsed with the corresponding parser.
* You should have the ability to set your data type and parsers for it.
* Such an approach also helps to support custom input types out from the box. An example is equation input using LaTeX or using GUI (as in Microsoft Office Word, Excel).
* You can specify how to render your data by deriving the `Renderable` interface. By default, the rendered content is just a rendered `toString`. Good examples are diagrams and equation input. *Both are expected to be builtins.*
* Iterables should have a suitable `Renderable` instance.
* Alternative approach is making all content be executable code. But it has the following disadvantages:
  * It is impossible to have custom inputs.
  * If you want to input just a text snippet, you have either to wrap it with `"""` or to use escape sequences.
  * The code is compilable in the Kotlin. Unfortunately, the author doesn't know any suitable way to decrease the start-up time of executing. The compilation is much slower than data parsing (with something like `Date.parse`).
* All regions know what code depends on the value in it. If type changes, all dependent scripts are recompiled and rerun, if only content changes (for non-code regions), they are just rerun.
* Compiled code and data are stored in the project files.
* Loops when found are explicitly shown and all depending code is stopped.
* There should be some cache that would help to revert some actions without recompilation.
* Lots of features such as Diagrams and Equations must have also a wizard that makes the same thing as code generating, but with better UI (but worse automatisation). It is better for users.

### Code support
* Evaluation is lazy because cell number may be infinite.
* `this` is a `Region` for the code in it.
* Code highlighting and auto-completion should work during editing.
* It may be better to highlight first indexes and second indexes differently.
* Code from the cell is private by default to motivate not to depend on its changes.
* There is also a common code for each sheet that is visible only from its regions.
* There is also a common global code for the whole document.
* There are 2 approaches:
  1. Code is compiled into `.jar` files. Each region with code has its `.jar`. You need to redownload the classes with the same classloader because otherwise the same classes would be recognised as different ones. Proxies should be used to determine who depends on who in runtime. So if you use some property of a region, you would use a proxy instance that would memoize the dependency existence. Scripts are recompiled sequentially to be able to find the loops.
  2. Use incremental compilation. It is more convenient but has a problem that all not compiled code needs to continue working during the recompilation of other parts.
* Libraries can be downloaded from scripts in cells or Settings of the app.
* The should be preloaded libraries that contain different economics functions and some other helpful ones.
* Safety is expected to be guaranteed by policy files. There should be a good UI to choose permissions or to add some custom ones.
* Default policy should forbid everything.
* It should be possible to specify a policy for some projects
* It should be possible to add a certificate verifying to the authors and choose the policy depending on the author.
* There should be an ability to set some periodic actions.
* There should be the ability to stop all evaluating scripts.

## Extra feature
* There should be a way import Excel files.

## Problems
* One of the main problems was slow compilation (comparing to interpretable languages with zero-time compilation). But it was solved by using code type only when it is a code, not data.
