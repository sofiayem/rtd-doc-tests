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

    .. code-block::

      // Convert the public key into PEM format:
      x509PublicKey, err := x509.MarshalPKIXPublicKey(&publicKey)
      if err != nil {
         log.Fatalln(err)
      }
      pemPublicKey := pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: x509PublicKey})

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
