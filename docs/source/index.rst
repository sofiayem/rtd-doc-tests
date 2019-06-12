.. Insolar documentation master file, created by
   sphinx-quickstart on Tue May 14 19:35:14 2019.

Insolar Documentation
=====================

.. note::

   Insolar's documentation is under development, its structure is not yet finalized. Expect updates soon.

To get a grip on how Insolar works, take a look at its :ref:`architecture overview <architecture>`.

To connect to TestNet 1.1, go through :ref:`step-by-step instructions <connecting_to_testnet>`.

Go snippet:

.. code-block:: go

   // _Functions_ are central in Go. We'll learn about
   // functions with a few different examples.

   package main

   import "fmt"

   // Here's a function that takes two `int`s and returns
   // their sum as an `int`.
   func plus(a int, b int) int {

       // Go requires explicit returns, i.e. it won't
       // automatically return the value of the last
       // expression.
       return a + b
   }

   // When you have multiple consecutive parameters of
   // the same type, you may omit the type name for the
   // like-typed parameters up to the final parameter that
   // declares the type.
   func plusPlus(a, b, c int) int {
       return a + b + c
   }

   func main() {

       // Call a function just as you'd expect, with
       // `name(args)`.
       res := plus(1, 2)
       fmt.Println("1+2 =", res)

       res = plusPlus(1, 2, 3)
       fmt.Println("1+2+3 =", res)
   }

.. // TODO: Develop a documentation entry point (main page) upon the content's readiness.

.. toctree::
   :maxdepth: 2
   :caption: Overview

   about
   features
   architecture
   glossary

.. // "Integration" below is a [WIP] title later to be expanded into a collection of how-to guides and renamed appropriately (if required).
 
.. toctree::
   :maxdepth: 2
   :caption: Integration

   integration
