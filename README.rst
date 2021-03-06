.. image:: https://img.shields.io/pypi/v/schwifty.svg?style=flat-square
    :target: https://pypi.python.org/pypi/schwifty
.. image:: https://img.shields.io/travis/mdomke/schwifty/master.svg?style=flat-square
    :target: https://travis-ci.org/mdomke/schwifty
.. image:: https://img.shields.io/pypi/l/schwifty.svg?style=flat-square
    :target: https://pypi.python.org/pypi/schwifty
.. image:: https://readthedocs.org/projects/schwifty/badge/?version=latest&style=flat-square
    :target: https://schwifty.readthedocs.io
.. image:: https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square
    :target: https://black.readthedocs.io/en/stable/index.html


Gotta get schwifty with your IBANs
==================================


Schwifty is a Python library for working with BICs and IBANs. It allows you to

* validate check-digits and the country specific format of IBANs
* validate format and country codes from BICs
* generate BICs from bank-codes (works for Germany for now)
* generate IBANs from country-code, bank-code and account-number.
* access all relevant components as attributes


Versioning
----------

Since the mapping from BIC to bank_code is updated from time to time, Schwifty uses
`CalVer <http://www.calver.org/>`_ with the scheme ``YY.0M.Micro``.

Usage
-----

IBAN
~~~~

Let's jump right into it:

.. code-block:: python

  >>> from schwifty import IBAN
  >>> iban = IBAN('DE89 3704 0044 0532 0130 00')
  >>> iban.compact
  'DE89370400440532013000'
  >>> iban.formatted
  'DE89 3704 0044 0532 0130 00'
  >>> iban.country_code
  'DE'
  >>> iban.bank_code
  '37040044'
  >>> iban.account_code
  '0532013000'
  >>> iban.length
  22
  >>> iban.bic
  <BIC=COBADEFFXXX>

So far so good. So you are able to create an ``IBAN``-object and to access all relevant components
of the IBAN as properties. As you can see on the last line, you can also get hold of the BIC number
associated to the bank-code of the IBAN. This currently only works for IBANs of german banks.

Behind the scenes the IBAN has been validated at the moment of instantiation. With respect to ISO
13616 compliance it is checked if the format of the account-code, the bank-code and possibly the
branch-code have the correct country-specific format. Whenever you pass an invalid IBAN to the
``__init__``-method, you'll get a ``ValueError`` with an appropriate error message.

.. code-block:: python

  >>> IBAN('DX89 3704 0044 0532 0130 00')
  ...
  ValueError: Unknown country-code DX

  >>> IBAN('DE99 3704 0044 0532 0130 00')
  ...
  ValueError: Invalid checksum digits


But what if you wan't to generate an IBAN from a bank-code and the account-code? Use the
``generate``-classmethod!

.. code-block:: python

  >>> iban = IBAN.generate('DE', bank_code='10010010', account_code='12345')
  <IBAN=DE40100100100000012345>
  >>> iban.checksum_digits
  '40'

Notice that even that the account-code has less digits than required (in Germany accounts should be
10 digits long), zeros have been added at the correct location. Additionally the checksum digits
have been calculated, which is good.


BIC
~~~

Besides the IBAN there is the Business Identifier Code (BIC). It is a unique identification code for
both financial and non-financial institutes. Schwifty also has a ``BIC``-object which more or less
has the same interface than the ``IBAN``-object.

.. code-block:: python

  >>> from schwifty import BIC
  >>> bic = BIC('PBNKDEFFXXX')
  >>> bic.bank_code
  'PBNK'
  >>> bic.branch_code
  'XXX'
  >>> bic.country_code
  'DE'
  >>> bic.location_code
  'FF'
  >>> bic.domestic_bank_codes
  ['10010010',
   '20010020',
   ...
   '86010090']

The ``domestic_bank_codes`` lists the country specific bank codes as you can find it in the IBAN.
This mapping is currently only available for German BICs and some Spanish and British banks.

The ``BIC``-object also does some basic validation on instantiation and raises a ``ValueError`` if
the country-code, the BIC´s length is invalid or if the structure doesn't match the ISO 9362
specification.

.. code-block:: python

  >>> BIC('PBNKDXFFXXX')
  ...
  ValueError: Invalid country code DX
  >>> BIC('PBNKDXFFXXXX')
  ...
  ValueError: Invalid length 12
  >>> BIC('PBN1DXFFXXXX')
  ...
  ValueError: Invalid structure PBN1DXFFXXXX

If Schwifty´s internal registry contains the BICs for your country (this again currently only works
for Germany), then you can use the ``exists``-property to check that the BIC is registered.



Installation
------------

To install Schwifty, simply:

.. code-block:: bash

  $ pip install schwifty


Development
-----------

We use the `black`_ as code formatter. This avoids discussions about style preferences in the same
way as ``gofmt`` does the job for Golang. The conformance to the formatting rules is checked in the
CI pipeline, so that it is recommendable to install the configured `pre-commit`_-hook, in order to
avoid long feedback-cycles.

.. code-block:: bash

   $ pre-commit install

You can also use the ``fmt`` Makefile-target to format the code or use one of the available `editor
integrations`_.


Name
----

Since ``swift`` and ``swiftly`` were already taken by the OpenStack-project, but we somehow wanted
to point out the connection to SWIFT, Rick and Morty came up with the idea to name the project
``schwifty``.

.. image:: https://i.cdn.turner.com/adultswim/big/video/get-schwifty-pt-2/rickandmorty_ep205_002_vbnuta15a755dvash8.jpg


.. _black:  https://black.readthedocs.io/en/stable/index.html
.. _pre-commit: https://pre-commit.com
.. _editor integrations:  https://black.readthedocs.io/en/stable/editor_integration.html
