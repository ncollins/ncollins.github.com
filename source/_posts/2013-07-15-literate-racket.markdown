---
layout: post
title: "Literate Racket"
date: 2013-07-15 16:00
comments: false
categories: [racket, scheme, literate programming]
---

I started messing around with literate programming in Racket recently using
the [Scribble](http://docs.racket-lang.org/scribble/) tool. I had a little
trouble getting the basic example working, because you need a base
Scribble document in addition to the literate program file:

```racket Base Scribble Document https://github.com/ncollins/literate-fizzbuzz/blob/gh-pages/index.scrbl index.scrbl
#lang scribble/base

@(require scribble/lp-include)

@title{Fizzbuzz in Literate Racket}

@(lp-include "fizzbuzz.scrbl")
```

Once this template is set up the main program can be written in
`fizzbuzz.scrbl`:

```racket Main Program https://github.com/ncollins/literate-fizzbuzz/blob/gh-pages/fizzbuzz.scrbl fizzbuzz.scrbl
#lang scribble/lp

...
```

Finally, to compile with
[cross-reference](http://docs.racket-lang.org/scribble/running.html#%28part._xref-flags%29)
links correctly it is necessary to use, 
`scribble +m --redirect-main http://docs.racket-lang.org/ index.scrbl`.
As `+m` links against local documentation by default `--redirect-main` is
used to point to the main racket site.
