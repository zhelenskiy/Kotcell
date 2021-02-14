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

### Bad syntax
* extra cells usage
* lambdas
* oneliners
* finite ranges
* partial translation

### Interop & libraries

### Solutions
