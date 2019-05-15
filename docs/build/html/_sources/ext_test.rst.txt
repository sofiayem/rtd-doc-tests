Testing Sphinx extensions
-------------------------

Tabs example
~~~~~~~~~~~~

.. tabs::

   .. tab:: Apples

      Apples are green, or sometimes red.

   .. tab:: Pears

      Pears are green.

   .. tab:: Oranges

      Oranges are orange.


PlantUML example
~~~~~~~~~~~~~~~~

.. uml::

   @startuml
   Alice -> Bob: Hi!
   Alice <- Bob: How are you?
   @enduml

Go code block example
~~~~~~~~~~~~~~~~~~~~~

.. code:: go

   package main
   import "fmt"
   func main() {
       fmt.Println("hello world")
   }