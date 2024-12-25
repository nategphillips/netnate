+++
title = 'natex'
date = 2024-02-28
draft = false
tags = ['computing']
summary = 'My custom $\LaTeX$ package.'
[params]
    math = true
+++

I love wasting time trying to perfect my notation when I'm writing things in $\LaTeX$. In fact, I would call it a hobby at this point, which is somewhat concerning. Before describing why I made a package and what it offers, I'll begin with an introduction to macros and packages for those unfamiliar.

## Background

### Macros

One of the more annoying things about writing technical documents is standardizing the notation---which really boils down to the question of "what macros can I use to make this stuff require less keystrokes?" Ideally, this question is answered by finding a suitable package that includes the desired features and learning the basics. However, there comes a point where you will inevitably find yourself reusing a string of characters that isn't offered in a package and think, "I really wish there was a command for this." Take the following example of typesetting an order of magnitude:

```latex
\begin{equation}
    \mathcal{O}\!\left(x^2\right)
\end{equation}
```

Who in their right mind wants to write that string of **pure garbage** every time an order of magnitude symbol is needed? This is a clear case where a custom macro is a good idea; one possible implementation is shown below:

```latex
\newcommand{\order}[1]{\mathcal{O}\!\left(#1\right)}

...

\begin{equation}
    \order{x^2}
\end{equation}
```

Just place the new command in your preamble, and you're ready to use the `\order` command in math environments. Everyone who's used $\LaTeX$ has run into this problem and defined their own macros. It's a fairly common occurrence.

Macros are important beyond their brevity. Say you wanted to use a different script for the $\mathcal{O}$ in the order of magnitude across your *entire* document. With a custom macro, this is trivial; change the `\mathcal` command in the definition and you're done. If that long string has been copied and pasted all across your document, the task becomes tedious at best. With more complex changes, or in a document with multiple files, the problems only escalate.

### Packages

If your experience is anything like mine, there will come a point where you've built up a nest egg of familiar macros which then get copied and pasted into all newly created documents ad nauseam. Why not abstract the process again and create a custom package? Just import the package and use the included macros without all the copy-paste nonsense. Beautiful.

## [natex](https://github.com/nategphillips/natex)

In the past, I used the `physics` package quite regularly since it had a bunch of macros that were generally useful and easy to remember. It covered most of my use cases and had notation that I liked, particularly for vectors and derivatives. It's somewhat of a controversial package in the $\LaTeX$ community because of a myriad of issues, some of which are expounded upon in [this](https://tex.stackexchange.com/questions/471532/alternatives-to-the-physics-package) post.

In general, it's better for a package to do one thing well rather than a bunch of things with varying success---this is the primary reason why I stopped using the `physics` package (it also has an annoying tendency to overwrite commands present in other packages, namely `amsmath` and `siunitx`). Funnily enough, `natex` also violates this rule inherently because I designed it to replace `physics`. The difference is, I can control my own package. That's literally it. It's an important distinction and shouldn't be overlooked.

I describe `natex` as "a collection of commands focused on consistent notation for engineering and physics applications," which is an accurate description of what I'm trying to do. To be clear, this package isn't meant for others to use, it's just something which allows me to save time and effectively standardize my writing process. Having a consistent look and feel across documents is underrated and dramatically improves the quality of techincal writing. If you're already using $\LaTeX$, you likely understand the importance of typesetting and have devoted a nontrivial amount of time learning an archaic system just to look down on Word users. Invest the extra time to create a custom package, then relish in the glory of having better looking papers than 90% of academic journals.

The basic structure of `natex` involves shorthand commands for various things, including vectors, automatic bracing, and matrices. I use a collection of packages to achieve this: `amssymb`, `bm`, `braket`, `mathtools`, and `physics2`. The package is easy to understand, and the `.sty` file is only 69 lines long. I've made a nice PDF explaining all the included commands, including their use and example output. Source code is [here](https://github.com/nategphillips/natex).
