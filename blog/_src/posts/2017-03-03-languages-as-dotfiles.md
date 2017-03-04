    Title: Languages as Dotfiles
    Date: 2017-03-03T13:18:04
    Tags:

*posted by Leif Andersen and Ben Greenman*

Tired of writing `(require (for-syntax syntax/parse))`[racket] at the top of your
Racket programs?
This post shows how to make a `#lang`[racket] to customize your default programming
environment.

<!-- more -->

Let's build a language `#lang scratch`[racket] that:

* loads the `racket/base`[racket], `racket/format`[racket],
  `racket/list`[racket], and `syntax/parse`[racket]
  (at [phase 1](http://docs.racket-lang.org/guide/phases.html))
  libraries; and

* enables Scribble's [@-syntax](http://docs.racket-lang.org/scribble/reader.html).

We'll follow a three-step recipe:

1. build an empty `scratch` library,

2. load the libraries, and

3. change the reader.

At the end, we'll see how to make `scratch` your default language in
DrRacket.


## Getting Started

First we need to make a `scratch/` directory with `info.rkt` and `main.rkt` files:

```bash
$ mkdir scratch; cd scratch
$ touch info.rkt
$ touch main.rkt
```

Inside the `info.rkt` file, write:

```racket
#lang info
(define name "scratch")
(define deps '("base"))
(define version "0.0")]
```

Inside the `main.rkt` file, write:

```racket
#lang racket/base
```

Now from inside the `scratch/` directory, install the package:

```bash
$ raco pkg install
```

You are now the proud parent of a new Racket package.

> [raco-pkg-new]() is a shortcut for starting a new package.

> For more information on the `info.rkt` file format, see
> [the pkg documentation]() and [the raco documentation]().

## Combining Libraries

Any program can now `(require scratch)` to import all bindings provided
by the `main.rkt` file.
Our next step is to reprovide bindings from other libraries in `main.rkt`.

Since we want to use `scratch` as a language, we also need to specify how to
[read]() a `scratch` program.
The `syntax/module-reader`[racket] language provides a shorthand for doing so.

Here is the updated `main.rkt` file:

```racket
#lang racket/base

(require racket/format racket/list
         (for-syntax racket/base syntax/parse))

(provide (all-from-out racket/format racket/list racket/base)
         (for-syntax (all-from-out racket/base syntax/parse)))

(module* reader syntax/module-reader
scratch)
```


The `provide`[racket] form declares the exports of the `scratch` module.
In other words, if another module contains the form `(require scratch)`
then that module will import bindings from `racket/base`[racket],
`racket/format`[racket], `racket/list`[racket], and `syntax/parse`[racket].

The `reader` submodule is written in the `syntax/module-reader`[racket]
language.
This submodule imports all bindings from its enclosing module
(`scratch`, or to be slightly more precise "the toplevel module in the
file `scratch/main.rkt`") and defines a language that provides those
bindings and uses the reader from `racket/base`[racket].

In short, this code does what we want.

```racket
scratch

(define-syntax (did-it-work? stx)
  (syntax-parse stx
    [_ #'(first '(yes it did))]))

(did-it-work?)
```

Yes it does.

> Annoyed that the `require`[racket] and `provide`[racket] forms are so
> similar? [There's a library for that]()


## Changing the Reader

Next, we want to enable the [@-expression]() reader. This involves
rexporting the scribble `read`[racket] and `read-syntax`[racket]
functions in the `reader` submodule in `main.rkt`:

```racket
#lang racket/base

(require racket/list
         (for-syntax racket/base syntax/parse))

(provide (all-from-out racket/list racket/base)
         (for-syntax (all-from-out racket/base syntax/parse)))

(module* reader syntax/module-reader
  scratch
  #:read s:read
  #:read-syntax s:read-syntax
  (require (prefix-in s: scribble/reader)))
```

To test that it works, let's embed some C syntax in our Racket program:

```racket
#lang scratch

(define-syntax (did-it-work? stx)
  (syntax-parse stx
    [_ #'(first '(yes it did))]))

(did-it-work?)

@~a{
 int main() {
   return 0;
 }}
```

> At this point, running `$ raco setup --check-pkg-deps scratch`[bash] will
> report an undeclared dependency on `at-exp-lib`.
> Make sure to add `at-exp-lib` to the `deps`[racket] list in your
> `info.rkt` file.

> Using `prefix-in`[racket] is not necessary; it just clarifies where
> `read`[racket] and `read-syntax`[racket] come from.}

> If you think inline C @emph{strings} are interesting, you should
> definitely watch Jay McCarthy's RacketCon 2016
> [talk](https://www.youtube.com/watch?v=r1kE3R26dK0) on
> [remix](https://github.com/jeapostrophe/remix").


## DrRacket's Automatic `#lang`[racket]

To make `scratch` the default language for new files in DrRacket:

1. Click "Language" in the menu bar.

2. Click "Choose Language" in the drop-down menu.

3. Click the radio button for "The Racket Language", then click the "Show Details" button at the bottom of the window.

4. Type `#lang scratch` in the text box labeled "Automatic `#lang line`''.

Click "Ok", and that's the end. Enjoy.


## The End

You can and should engineer the `#lang`[racket] line of your Racket programs to
remove unnecessary boilerplate and/or enforce a project-specific development
environment.

Notes:

* Feel free to pubish your custom language on the Racket
  [package server](https://pkgn.racket-lang.org").

Make sure to run `$ raco setup --check-pkg-deps scratch`[bash] beforehand!

* Our personal "dotfiles" are
  [racket-scratch](https://github.com/LeifAndersen/racket-scratch")
  and [agile](https://github.com/bennn/agile).

* The title "Languages as Dotfiles" is a reference to
  [Languages as Libraries]("/2011/03/languages-as-libraries-pldi-2011.html)

