Usage
=====

.. _installation:

Installation
------------

To use Lumache, first install it using pip:

.. code-block:: console

   (.venv) $ pip install lumache

Creating recipes
----------------



### MakerDAO v1 {#makerdao-v1}

Any entity may pay off a vault’s debt and receive a proportional amount of collateral plus a bonus known as a liquidation fee (e.g. 10%).

Pros:



* Simple to build

Cons:



* Price-inefficient, small liquidators cannot participate as no partial liquidations

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']

