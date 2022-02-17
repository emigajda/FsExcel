# Welcome!

Welcome to FsExcel, a library for generating for generating Excel spreadsheets (`.xlsx` files), using very simple code.

FsExcel is based on [ClosedXML](https://github.com/ClosedXML/ClosedXML) but abstracts away many of the complications of building spreadsheets cell by cell.

> This tutorial is also available as an [interactive notebook](https://github.com/misterspeedy/FsExcel/blob/main/src/Notebooks/Tutorial.dib)! Download it, open in Visual Studio Code, and start generating spreadsheets for real!

---
## Hello World

Here's the complete code to generate a spreadsheet with a single cell containing a string!

Run this and you should find a spreadsheet called `HelloWorld.xlsx` in your `/temp` folder. (Change the path to suit.)

```fsharp
// For scripts only; for programs, use NuGet to install FsExcel:
#r "nuget: FsExcel"

open FsExcel

[
    Cell [ String "Hello world!" ]
]
|> render "HelloWorld"
|> fun wb -> wb.SaveAs "/temp/HelloWorld.xlsx"

```
| | *A* |
| --- | --- |
| *1* | Hello world! |

This example already embodies the three main stages of building a spreadsheet using FsExcel:

1) Build a list using a list comprehension: `[ ... ]`
2) In the list make cells using `Cell`
3) Each cell gets a list of properties, in this case just the cell content, which here is a string: `String "Hello world!"`


If you've used `Fable.React` you'll already be familiar with the concepts so far.

4) Send the resulting list to `FsExcel.render`.  Also provide a name for the worksheet tab.  (FsExcel currently only supports one worksheet per workbook.)
5) The result is a `ClosedXML` workbook which you can save with its `.SaveAs` method.

---
## Multiple Cells

```fsharp
open FsExcel

[
    for i in 1..10 do
        Cell [ Integer i ]
]
|> render "MultipleCells"
|> fun wb -> wb.SaveAs "/temp/MultipleCells.xlsx"

```
| | *A* | *B* | *C* | ... | *J* |
| --- | ---: | ---: | ---: | --- | ---: |
| *1* | 1 | 2 | 3 | ... | 10 |

Here we use a `for...` comprehension to build multiple cells. (Don't panic: we could have used `List.map` instead!)

By default each new cell is put on the right of its predecessor.

---
## Vertical Movement

If we want the next cell to be rendered below instead of to the right, we can add a `Next(DownBy 1)` property to the cell:

```fsharp
open FsExcel
open System.Globalization

[
    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
            Next(DownBy 1)
        ]
]
|> render "VerticalMovement"
|> fun wb -> wb.SaveAs "/temp/VerticalMovement.xlsx"

```
| | *A* |
| --- | --- |
| 1 | January |
| 2 | February |
| 3 | March |
| | ... |
| 12 | December |

The `Next` property overrides the default behaviour of rendering each successive cell one to the right. In this case we override it with a 'go down by 1' behaviour.

But what if we want a table of cells? Use the default behaviour for each cell in a row except the last. In the last cell use `Next(NewRow)`. This causes the next cell to be rendered in column 1 of the next row.

```fsharp
open FsExcel
open System.Globalization

[
    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
            Next(NewRow)
        ]
]
|> render "Rows"
|> fun wb -> wb.SaveAs "/temp/Rows.xlsx"

```
| | *A* | *B* |
| --- | --- | --- |
| 1 | January | 7 |
| 2 | February | 8 |
| 3 | March | 5 |
| | ... | |
| 12 | December | 8 |

Maybe you don't like the idea of saying where to go next in the properties of a cell. No problem, you can have standalone position-control with the `Go` instruction:

```fsharp
open FsExcel
open System.Globalization

[
    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
        ]
        Go(NewRow)
]
|> render "RowsGo"
|> fun wb -> wb.SaveAs "/temp/RowsGo.xlsx"

```
| | *A* | *B* |
| --- | --- | --- |
| 1 | January | 7 |
| 2 | February | 8 |
| 3 | March | 5 |
| | ... | |
| 12 | December | 8 |

---
## Indentation

Maybe you want a series of rows that don't start in column 1.  Use `Indent`:

```fsharp
open FsExcel
open System.Globalization

[
    Go(Indent 2)

    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
        ]
        Go(NewRow)
]
|> render "Indentation"
|> fun wb -> wb.SaveAs "/temp/Indentation.xlsx"

```
| | *A* | *B* | *C* |
| --- | --- | --- | --- |
| 1 | | January | 7 |
| 2 | | February | 8 |
| 3 | | March | 5 |
| | | ...  ||
| 12 | | December | 8 |

Now each row begins at column 2.

Indents apply to all `NewRow` operations until some other indent value is set using `Go(Indent n)`. Specify no indenting with `Go(Indent 1)`.

You can specify indents relative to the current indent level using `Go(IndentBy n)` where _n_ can be a positive or negative integer.

---
## Border and Font Styling

You can add border and font emphasis (bold or italic) styling using additional cell properties.  The border style values are in `ClosedXML.Excel.XLBorderStyleValues`.

```fsharp
open FsExcel
open System.Globalization
open ClosedXML.Excel

[
    for heading in ["Month"; "Letter Count"] do
        Cell [
            String heading
            Border(Border.Bottom(XLBorderStyleValues.Medium))
            FontEmphasis(Bold)
            FontEmphasis(Italic)
        ]
    Go(NewRow)
    
    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
        ]
        Go(NewRow)
]
|> render "Styling"
|> fun wb -> wb.SaveAs "/temp/Styling.xlsx"

```
| | *A* | *B* |
| --- | --- | --- |
| 1 | *Month* | *Letter Count* |
| 2 | January | 7 |
| 3 | February | 8 |
| 4 | March | 5 |
| | ... | |
| 13 | December | 8 |

As they are just list items, styles can be composed and applied together as a list. You'll need a `yield!` to include these multiple elements in your cell property list.

```fsharp
open FsExcel
open System.Globalization
open ClosedXML.Excel

let headingStyle = 
    [
        Border(Border.Bottom(XLBorderStyleValues.Medium))
        FontEmphasis(Bold)
        FontEmphasis(Italic)
    ]

[
    for heading in ["Month"; "Letter Count"] do
        Cell [
            String heading
            yield! headingStyle
        ]
    Go(NewRow)
    
    for m in 1..12 do
        let monthName = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
        ]
        Go(NewRow)
]
|> render "ComposedStyling"
|> fun wb -> wb.SaveAs "/temp/ComposedStyling.xlsx"

```
Number styling can be applied using standard Excel format strings.  You can also apply horizontal alignment.

```fsharp
open FsExcel
open System.Globalization
open ClosedXML.Excel

let r = System.Random()

let headingStyle = 
    [
        Border(Border.Bottom(XLBorderStyleValues.Medium))
        FontEmphasis(Bold)
        FontEmphasis(Italic)
    ]

[
    for heading, alignment in ["Stock Item", Left; "Price", Right ; "Count", Right] do
        Cell [
            String heading
            yield! headingStyle
            HorizontalAlignment(alignment)
        ]
    
    Go(NewRow)

    for item in ["Apples"; "Oranges"; "Pears"] do
        Cell [
            String item
        ]
        Cell [
            Float ((r.NextDouble()*1000.))
            FormatCode "$0.00"
        ]
        Cell [
            Integer (int (r.NextDouble()*100.))
            FormatCode "#,###"
        ]
        Go(NewRow)
]
|> render "NumberFormatAndAlignment"
|> fun wb -> wb.SaveAs "/temp/NumberFormatAndAlignment.xlsx"

```
| | *A* | *B* | *C* |
| --- | --- | ---: | ---: |
| 1 | *Stock Item* | *Price* | *Count* |
| 2 | Apples | $124.16 | 41 |
| 3 | Oranges | $755.89 | 40 |
| 4 | Pears | $679.50 | 88 |
