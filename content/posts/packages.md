+++
title = 'packages'
date = 2024-02-26
draft = true
math = true
tags = ['computing']
summary = 'Some neat packages.'
+++

## Essentials

Generally useful packages that offer commonly used macros.

1. [`fancyhdr`](https://ctan.org/pkg/fancyhdr) - provides tools for constructing headers and footers with all kinds of customizable options
2. [`geometry`](https://ctan.org/pkg/geometry) - changes page dimensions and removes the unnecessarily large default margins
3. [`hyperref`](https://ctan.org/pkg/hyperref) - support for external hyperlinks and internal document references
4. [`mathtools`](https://ctan.org/pkg/mathtools) - an extension that internally loads the [`amsmath`](https://ctan.org/pkg/amsmath) package along with various fixes and improvements
5. [`parskip`](https://ctan.org/pkg/parskip) - allows for zero `\parindent` and nonzero `\parskip`, which makes for much more readable documents

## Other Useful Packages

Packages I use depending on the type and scale of the document itself.

1. `amssymb` - provides an extended symbol collection as a superset of [`amsfonts`](https://ctan.org/pkg/amsfonts) that includes some more obscure symbols
   - `amsfonts` is included and loads things like `\mathbb` for characters like $\mathbb{R}$
   - If the heirarchy of `ams` packages is confusing, check [this](https://tex.stackexchange.com/questions/32100/what-does-each-ams-package-do) thread out
2. [`amsthm`](https://ctan.org/pkg/amsthm) - enhanced version of $\LaTeX$'s `\newtheorem` command for defining customizable theorem-like environments
3. [`biblatex`](https://ctan.org/pkg/biblatex) - works in tandem with a `.bib` file to compile references
4. [`bm`](https://ctan.org/pkg/bm) - bold symbols in math mode that are safer than those from `\boldsymbol`
5. [`booktabs`](https://ctan.org/pkg/booktabs) - improves the look of tables
6. [`braket`](https://ctan.org/pkg/braket) - provides macros to typeset bra-ket notation
7. [`cancel`](https://ctan.org/pkg/cancel) - places lines through cancelled terms in math equations
8. [`caption`](https://ctan.org/pkg/caption) - customized captions in figure and table environments, I like my caption headings **bolded** for readability
9. [`derivative`](https://ctan.org/pkg/derivative) - derivatives and differentials via short, easy to remember macros
10. [`enumitem`](https://ctan.org/pkg/enumitem) - provides additional numbering and structuring control over the enumerate, itemize, and description environments
11. [`esint`](https://ctan.org/pkg/esint) - useful for typesetting line and surface integrals
12. [`float`](https://ctan.org/pkg/float) - sometimes necessary for the placement of stubborn figures and tables
13. [`graphicx`](https://ctan.org/pkg/graphicx) - allows for graphics, images, etc. to be added
14. [`mathrsfs`](https://ctan.org/pkg/mathrsfs) - provides the `\mathscr` command for formal uppercase script letters like $\mathscr{P}$
      - The internal $\LaTeX$ script letters from `\mathcal` look like $\mathcal{P}$
15. [`mhchem`](https://ctan.org/pkg/mhchem) - typesetting for chemical molecular formulae and equations
16. [`minted`](https://ctan.org/pkg/minted) - syntax highlighting for all sorts of source code using the [Pygments](https://pygments.org) library
17. [`multicol`](https://ctan.org/pkg/multicol) - can be used to have multiple columns of text, I primarily use it for 2-column lists
18. [`natex`](https://github.com/nategphillips/natex) - shameless plug, who knew I would use my own package
19. [`pgfplots`](https://ctan.org/pkg/pgfplots) - draws high-quality function plots in normal or logarithmic scaling with a user-friendly interface
20. [`siunitx`](https://ctan.org/pkg/siunitx) - a comprehensive SI unit package that includes macros for scientific notation
21. [`tcolorbox`](https://ctan.org/pkg/tcolorbox) - nicely formatted and customizable boxes for examples and worked problems
22. [`tikz`](https://ctan.org/pkg/tikz) - adds support for native diagram and graph creation

### An Aside on Upright Math

I have strong opinions on clear and proper notation, and I think everyone should.
One oft-overlooked aspect of writing I've seen in books is the tendency to strictly use italicized characters in math mode.
Besides the obvious aesthetic failings, this approach tends towards a lack of clarity due to ambiguities brought upon by the sole use of italics.
Below, I explain where upright characters should be used to remedy these issues.

The ground rules for using upright characters are as follows (list pulled from [here](https://tex.stackexchange.com/questions/33120/should-subscripts-in-math-mode-be-upright)):

- upright function names $(\sin{x}$, $\cos{x}$, $\ln{x}$, etc.$)$
- upright differentials $(\mathrm{d}x$ and $\int x \ \mathrm{d}x)$
- upright dimensionless numbers $(\mathrm{Re}$, $\mathrm{Pr}$, etc.$)$
- upright descriptive text
- upright descriptive variable indices (that don't correspond to other variables)

I think the first bullet point is clear enough, and is generally well followed.
After all, most commonly used functions are already defined as built-in macros within $\LaTeX$, so there's no reason *not* to use them.

Typesetting differentials is mainly a matter of personal taste, but I find upright differentials to be more clear.
From my cursory research, it seems like most mathematicians still prefer the italic text.
Older textbooks and papers almost exclusively use italics as well.
In newer texts, I have started seeing a marked increase in the upright variation, so it seems to be catching on.

Upright dimensionless numbers are more important.
For instance, using $Re$ instead of $\mathrm{Re}$ to denote the Reynold's number is unclear, since $R$ and $e$ could easily be interpreted as two separate variables.
In most cases, the meaning will be clear from context, but it's always better to adopt a standard and follow it for the sake of consistency.
Descriptive text falls into the same category.

Variable indices are the most tricky to nail down.
When the subscript is descriptive, like $a_\mathrm{profile}$, it should be upright.
For index counters as in $\sum_i x_i$, italic is correct.
Technically, according to ISO and NIST, one-letter subscripts that refer to a name, such as $k_\mathrm{B}$ for the Boltzmann constant, or $m_\mathrm{e}$ for the mass of an electron, should be upright.
However, the American Physical Society recommends all single-letter subscripts to be italic.
I've seen both in books and in papers, but I prefer the APS guidelines.
To make upright subscripts less of a pain, I found a neat little command on StackExchange at some point, which I've included in [my package](/posts/natex).

By the way, if you're following the official ISO standards, even the imaginary unit $i$, Euler's number $e$, and $\pi$ should be upright.
This practice is absolutely atrocious and I don't recommend following it.
