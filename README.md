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
All ranges must have a finite size in **Excel**. But that can be bad if you want to be able to proceed with any number of items. For example, You have some numbers in column `A`, and you want to have them powered with 2 in the column `B`. You have to know a maximal number of elements in the first column by the moment of creating the equation for cells in column `B`.

### Predicates
Predicates (an example is above) have very ugly and not extendable syntax.
* They are ugly because they are specified just as string literals with no validation.
* They are not extendable because they only work in this case: if I want some more complex condition (or just `num -> num < 3 || num > 5`), I have to use intermediate cells.

A lot better way is to use lambda functions.

### Oneliners
All cell formulas are expressions with no intermediate named values inside the cell, that is why you have to use additional cells to contain intermediate variables to make the main cell content readable. By the way, `LET` function is going to be added to the stable release so the value of the point is a lot less now. However, such a function is still quite ugly.

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
* __Brevity__

  It must be not verbose: the formulas would become too complicated.
  They should be at least not longer than **Excel**'s ones in most cases.
* __Expressiveness__

  Simple things should be simply coded and simply understood. Hard things should be simply understood.
  No boilerplate for simple things. *This also includes having some helpful operators such as range-operators.*

#### Highly wanted:
* __Fast compilation (if needed)__

  Delay between printing a formula and its evaluation should be small (as it is in **Excel**)
* __Static typing__

  Type-safety is a guarantee that everything will continue working even if conditional branch in some formula would be changed because of some data changing.

#### Wanted, but not necessary:
* __High speed__
  
  **Excel** computations are slow so high speed would be a good bonus.

------

Here is approximate list of languages with their comparison in the context of the app.

<table>
  <tr>
    <th>Name</th><th>Simplicity</th><th>Expressiveness</th><th>Safety & Static typing</th><th>High speed</th><th>Fast compilation</th><th>Sum</th>
  </tr>
  <tr>
    <th>C++</th>
    <th><pre lang='diff'>- 0 points</pre></th>
    <th><pre lang='diff'>! 4 points</pre></th>
    <th><pre lang='diff'>! 5 points</pre></th>
    <th><pre lang='diff'>+ 10 points</pre></th>
    <th><pre lang='diff'>- 0 points</pre></th>
    <th>19 points</th>
  </tr>
  <tr>
    <th>Python</th>
    <th><pre lang='diff'>+ 10 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>- 3 point</pre></th>
    <th><pre lang='diff'>! 4 points</pre></th>
    <th><pre lang='diff'>+ 10 points</pre></th>
    <th>35 points</th>
  </tr>
  <tr>
    <th>Java</th>
    <th><pre lang='diff'>+ 7 points</pre></th>
    <th><pre lang='diff'>- 3 points</pre></th>
    <th><pre lang='diff'>+ 7 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>! 6 points</pre></th>
    <th>31 points</th>
  </tr>
  <tr>
    <th>Kotlin</th>
    <th><pre lang='diff'>+ 7 points</pre></th>
    <th><pre lang='diff'>+ 10 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>! 6 points</pre></th>
    <th>39 points</th>
  </tr>
  <tr>
    <th>C#</th>
    <th><pre lang='diff'>+ 7 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>! 6 points</pre></th>
    <th>37 points</th>
  </tr>
  <tr>
    <th>Haskell</th>
    <th><pre lang='diff'>- 3 points</pre></th>
    <th><pre lang='diff'>! 5 points</pre></th>
    <th><pre lang='diff'>+ 10 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th><pre lang='diff'>+ 8 points</pre></th>
    <th>34 points</th>
  </tr>
</table>

So the chosen language was Kotlin.
Solution of the compilation speed problem would be given in the corresponding paragraph of the article.

## Implementation

## Problems
