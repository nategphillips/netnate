+++
title = 'Packages for typesetting'
date = 2026-04-25
draft = false
math = true
tags = ['computing']
summary = 'A collection of the $\LaTeX$ packages that I use.'
+++

## Package listing

These days, Lua$\LaTeX$ is the [officially recommended](https://www.texdev.net/2024/11/05/engine-news-from-the-latex-project) compiler over PDF$\LaTeX$, primarily due to the Unicode and tagging support it offers.
I use Lua$\LaTeX$ whenever possible, so keep that in mind when reading this list.
For users of PDF$\LaTeX$, take a look at the second section.

- [`amsthm`](https://ctan.org/pkg/amsthm)
    - Used for defining customizable theorem-like environments
    - `\newtheorem`
- [`biber`](https://ctan.org/pkg/biber)
    - A UTF-8 supporting backend for Bib$\LaTeX$ that reads and processes entries
- [`biblatex`](https://ctan.org/pkg/biblatex)
    - Provides Unicode-aware formatting of bibliographies entirely controlled by LaTeX macros
- [`booktabs`](https://ctan.org/pkg/booktabs)
    - Publication-quality tables
    - `\bottomrule`, `\cmidrule`, `\midrule`, `\toprule`
- [`cancel`](https://ctan.org/pkg/cancel)
    - Places lines through cancelled terms in math equations
    - `\cancel`, `\cancelto`
- [`caption`](https://ctan.org/pkg/caption)
    - Improves caption alignment and allows for bold labels
- [`cleveref`](https://ctan.org/pkg/cleveref)
    - Automatic referencing for equations, figures, tables, and sections
    - `\cref`, `\crefrange`
- [`derivative`](https://ctan.org/pkg/derivative)
    - Simple and nice-looking derivatives and differentials
    - `\adif`, `\odif`, `\odv`, `\mdv`, `\pdv`
- [`enumitem`](https://ctan.org/pkg/enumitem)
    - Provides additional numbering and structuring control over the enumerate, itemize, and description environments
- [`fancyhdr`](https://ctan.org/pkg/fancyhdr)
    - Provides tools for constructing headers and footers with all kinds of customizable options
    - `\fancyfoot`, `\fancyhead`
- [`float`](https://ctan.org/pkg/float)
    - Sometimes necessary for the placement of stubborn figures and tables
- [`fontsetup`](https://ctan.org/pkg/fontsetup)
    - A wrapper package which loads `fontspec`, `unicode-math`, and a font of your choice
    - Only supports Lua$\LaTeX$
- [`geometry`](https://ctan.org/pkg/geometry)
    - Changes page dimensions and removes the unnecessarily large default margins
- [`graphicx`](https://ctan.org/pkg/graphicx)
    - Allows for graphics, images, etc. to be added
    - `\includegraphics`, `\resizebox`
- [`hyperref`](https://ctan.org/pkg/hyperref)
    - Support for external hyperlinks and internal document references
    - `\href`, `\url`
- [`mathtools`](https://ctan.org/pkg/mathtools)
    - An extension to [`amsmath`](https://ctan.org/pkg/amsmath) with various fixes and improvements
    - `\cases*`, `\DeclarePairedDelimiterX`
- [`minted`](https://ctan.org/pkg/minted)
    - Syntax highlighting for all sorts of source code using the [Pygments](https://pygments.org) library
    - `\mintinline`
- [`mhchem`](https://ctan.org/pkg/mhchem)
    - Provides commands for typesetting chemical molecular formulae and equations
    - `\ce`
- [`multirow`](https://ctan.org/pkg/multirow)
    - Creates cells that span more than one row in a table
    - `\multirow`
- [`natex`](https://github.com/nategphillips/natex)
    - My package for mathematics, physics, and engineering macros
    - Separate implementations for Lua$\LaTeX$ and PDF$\LaTeX$
- [`pagecolor`](https://ctan.org/pkg/pagecolor)
    - Recolors the page background and text
    - `\color`, `\pagecolor`
- [`pgf`](https://ctan.org/pkg/pgf)
    - Create PostScript and PDF graphics in $\TeX$, great for Matplotlib plots
- [`pgfplots`](https://ctan.org/pkg/pgfplots)
    - Draws high-quality function plots using the TikZ interface
    - Automatically loads `tikz`
- [`physics2`](https://ctan.org/pkg/physics2)
    - Provides a submodule for automatic braces
    - `\ab`
- [`siunitx`](https://ctan.org/pkg/siunitx)
    - A comprehensive SI unit package that includes macros for scientific notation
    - `\ang`, `\num`, `\unit`, `\qty`, `\qtyrange`
- [`tcolorbox`](https://ctan.org/pkg/tcolorbox)
    - Nicely formatted and customizable boxes for examples and worked problems
- [`tikz`](https://ctan.org/pkg/tikz)
    - A user-friendly frontend for `pgf` adds support for native diagram and graph creation
    - Automatically loads `pgf`
- [`threeparttable`](https://ctan.org/pkg/threeparttable)
    - Provides "footnotes" for tables
    - `\tnote`
- [`unicode-math`](https://ctan.org/pkg/unicode-math)
    - A comprehensive implementation of unicode math for Lua$\LaTeX$
    - `\setmathfont`
    - Only supports Lua$\LaTeX$
- [`xcolor`](https://ctan.org/pkg/xcolor)
    - Allows for defining and using colors
    - `\color`, `\definecolor`

## Packages for PDF$\LaTeX$ only

When using the Lua$\LaTeX$ compiler with `unicode-math` and a supported math font like [New Computer Modern](https://ctan.org/pkg/newcomputermodern), the following packages are not necessary since their features are already implemented.
Additionally, the Unicode variants are often much nicer since they can be directly copied into a browser or text editor and are supported by screen readers.
If you must use PDF$\LaTeX$, these are nice-to-haves.

- [`amsfonts`](https://ctan.org/pkg/amsfonts)
    - Adds blackboard math and Fraktur fonts
    - `\mathbb`, `\mathfrak`
- [`amssymb`](https://ctan.math.utah.edu/ctan/tex-archive/fonts/amsfonts/source/amssymb.dtx)
    - Provides an extended symbol collection and loads `amsfonts`; if the heirarchy of AMS packages is confusing, check [this](https://tex.stackexchange.com/questions/32100/what-does-each-ams-package-do) thread out
    - `\Box`
- [`bm`](https://ctan.org/pkg/bm)
    - Bold symbols in math mode that are safer than those from `\boldsymbol`
- [`esint`](https://ctan.org/pkg/esint)
    - Necessary for surface, volume, and direction contour integrals when using PDF$\LaTeX$
    - `\iiint`, `\oiint`, `\ointclockwise`
- [`mathrsfs`](https://ctan.org/pkg/mathrsfs)
    - Adds Raph Smith’s Formal Script font to math mode
    - `\mathscr`

### Select commands and the relationship between select packages

Commands provided by `amssymb`, but technically not `unicode-math`:
- `\Box`, `\Diamond`, `\leadsto`
- Note that `unicode-math` can use the following symbols instead: □, ◇, ⇝

Commands provided by both `amssymb` and `unicode-math`:
- `\barwedge`, `\boxdot`, `\boxminus`, `\boxplus`, `\boxtimes`, `\Cap`, `\Cup`

Commands provided by both `amsfonts` and `unicode-math`:
- `\mathbb`, `\mathfrak`

Commands provided by both `mathrsfs` and `unicode-math`:
- `\mathscr`

Commands provided only by `unicode-math`:
- `\mathbffrak`, `\mathbfscr`, `\symbfcal`
