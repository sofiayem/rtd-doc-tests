.. Insolar documentation master file, created by
   sphinx-quickstart on Tue May 14 19:35:14 2019.

Insolar Documentation
=====================

.. note::

   Insolar's documentation is under development, its structure is not yet finalized. Expect updates soon.

To get a grip on how Insolar works, take a look at its :ref:`architecture overview <architecture>`.

To connect to TestNet 1.1, go through :ref:`step-by-step instructions <connecting_to_testnet>`.

.. // TODO: Develop a documentation entry point (main page) upon the content's readiness.

Tabs extension:

.. tabs::

   .. tab:: Python

      .. code-block:: python

         import my-api

   .. tab:: Ruby

      .. code-block:: ruby

         require 'my-api'

   .. tab:: Golang

      .. code-block:: guess

         import {'smth'}

.. content-tabs::

    .. tab-container:: tab1
        :title: Tab title one

         .. code-block:: guess

            import {'smth'}

    .. tab-container:: tab2
        :title: Tab title two

         .. code-block:: guess

            import {'smth'}

.. toggle-header::
   :header: Example 1 **Show/Hide Code**

   .. code-block:: go

      // Create a function to get a new seed for each signed request:
      func getNewSeed() string {
        // Form a request body for getSeed:
        getSeedReq := platformRequest{
          JSONRPC: JSONRPCVersion,
          Method:  "node.getSeed",
          ID:      id,
        }
        // Increment the id for future requests:
        id++

        // Send the request:
        seedBody := sendInfoRequest(getSeedReq)

        // Unmarshal the response:
        var seed seedResponse
        err := json.Unmarshal(seedBody, &seed)
        if err != nil {
          log.Fatalln(err)
        }
        // Put the current seed into a variable:
        return seed.Result.Seed
      }

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
