.. _migration_test:

Testing Token Migration
=======================

This test walks you though the process of acquiring test INS tokens and swapping them for test XNS coins.

During the testing period of the Insolar MainNet, all operations will be performed with test INS tokens and test XNS coins (native for MainNet).

.. _needs_for_migration_test:

What You Will Need
------------------

You need to have:

* **Ethereum wallet** that supports Ethereum test networks (Ropsten in particular) and custom tokens. This test scenario uses `MetaMask <https://metamask.io/>`_ as an example.
* **Insolar Wallet** in which you can store XNS coins native for the .

Additionally, you need to understand the following:

* **What coins and tokens are**. Coin is a native cryptocurrency of any blockchain and token is a unit of value that may be built on top of it. Coins operate independently, while tokens have specific uses. In the Ethereum network, the coin is the ether (ETH) and the token standard (upon which the INS token is built) is called ERC-20. Test tokens and coins are those that operate in test networks. For testing purposes, we are going to use the Ropsten test network.
* **What a crypto faucet is.** The faucet is a quick and easy way of creating and distributing test coins and tokens in test networks.
* **What a token contract is.** Essentially, a token contract is a smart contract that contains a map of account addresses and their balances. The unit of the balance is commonly called a token. When tokens are transferred from one account to another, the token contract updates the balances of the two accounts.

.. _test_overview:

Test Scenario Overview
----------------------

To grasp what you will be doing, take a look at the steps of the test scenario:

#. Create a MetaMask wallet and, inside, switch to Ropsten test network and add the INS custom token.
#. Get Ropsten test ETH coins.
#. Swap the coins to test INS tokens using the INS token contract.
#. Create an Insolar Wallet and, automatically, receive a migration address.
#. Send the test INS tokens from the MetaMask wallet to the migration address. They will automatically swap to XNS native coins and appear in the Insolar Wallet.

All the above steps are described in detail in subsequent sections.

.. _creating_metamask:

Creating and Setting Up Ethereum Wallet
---------------------------------------

To create and set up a MetaMask Ethereum wallet:

#. Go to `metamask.io <https://metamask.io>`_ and add its extension to your browser.
#. Open the extension, click :guilabel:`Get Started`, :guilabel:`Create a Wallet`, and agree or disagree to help MetaMask improve itself.
#. Create a new password, confirm it, and agree to the "Terms of Use".
#. Reveal the backup phrase, copy it, and click :guilabel:`Next`.
#. Clicking the words of the backup phrase in order and click :guilabel:`Confirm`.
#. Receive congratulations from MetaMask and click :guilabel:`All Done`. You will see the wallet's user interface.
#. In the top right corner, select :guilabel:`Ropsten Test Network` from the drop-down list:

   .. image:: imgs/selecting-ropsten.png
      :width: 700px

   |

#. In the bottom left corner, click :guilabel:`Add Token`:

   .. image:: imgs/add-token.png
      :width: 700px

   |

#. On the **Add Tokens** screen, open the :guilabel:`Custom Token` tab:

   .. image:: imgs/custom-token.png
      :width: 300px

   |

#. Copy the INS token contract address -- click the copy icon |copy-icon| in the right corner of the following code block:

   .. |copy-icon| image:: imgs/copy-icon.png
      :width: 20px

   .. code-block::

      0x7e94f2be613c6846c40325b0f2712269a0d61d10

#. In the :guilabel:`Token Contract Address` field, paste the copied INS token contract address:

   .. image:: imgs/ins-token.png
      :width: 300px

   MetaMask will find the INS token symbol and decimals of precision for you. Click :guilabel:`Next`.

#. On the next screen, click :guilabel:`Add Tokens`:

   .. image:: imgs/add-ins.png
      :width: 300px

   With that, the MetaMask wallet is set up to operate the test ETH coins and INS tokens:

   .. image:: imgs/wallet-setup.png
      :width: 700px

.. note:: To be continued...
