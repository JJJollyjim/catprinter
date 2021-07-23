# Cat Printer
> "cat printer 🥺"

*A central repository to link to documentation and projects related to cat-shaped bluetooth thermal printers. PRs encouraged.*

# Protocol implementations

In chronological order:

| Name                                                                                        | Comments                                                                                            | Bluetooth Library | Platforms verified working |
|---------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|-------------------|----------------------------|
| [WerWolv / PythonCatPrinter](https://github.com/WerWolv/PythonCatPrinter)                   | WerWolv's original implementation bassed on their [blog post](https://werwolv.net/blog/cat_printer) | Bleak (Python)    |                            |
| [amber-sixel / PythonCatPrinter](https://github.com/amber-sixel/PythonCatPrinter)           | AmberSixel's fork of WerWolv's implementation.                                                      | Bleak (Python)    |                            |
| [the6p4c / catteprinter](https://github.com/the6p4c/catteprinter)                           | the6p4c's Rust library                                                                              | btleplug (Rust)   | **Windows**<sup>1</sup>              |
| [JJJollyjim / PyCatte](https://github.com/JJJollyjim/PyCatte)                               | A barebones Python implementation                                                                   | Bleak (Python)    | **Linux**                  |
| [xssfox / print_server.py](https://gist.github.com/xssfox/b911e0781a763d258d21262c5fdd2dec) | Featureful Python implementation with an HTTP API, text rendering, and PostScript printing.         | Bleak (Python)    | **Linux**                  |

<sup>1</sup>: Appears to be broken on Linux due to an issue where btleplug never
correctly recieves the dbus property-change message which notifies it that a
connection has been established, even though "Connected" and "Services Resolved"
are displayed in `bluetoothctl`. I'm guessing this is a locking issue involving
the CondVar, or something like that?

# Protocol docs

WerWolv's [blog post](https://werwolv.net/blog/cat_printer) contains initial docs from RE of the Android app, however more is now known.

TODO: consolidate what is now known.

# Hardware

Three different model names (included in bluetooth advertisements) have been observed:

| Name | Known owners (colour variant)    |
|------|----------------------------------|
| GB01 | WerWolv (Blue)                   |
| GB02 | the6p4c (Pink)                   |
| GT01 | JJJollyjim (Pink), xssfox (Pink) |

