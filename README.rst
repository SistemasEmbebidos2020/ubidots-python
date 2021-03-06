===================================
Ubidots Python API Client
===================================

The Ubidots Python API Client makes calls to the `Ubidots Api <http://things.ubidots.com/api>`_.  The module is available on `PyPI <https://pypi.python.org/pypi/ubidots/>`_ as "ubidots".

To follow this quickstart you'll need to have python 2.7 in your machine (be it a computer or an python-capable device), which you can download at `<http://www.python.org/download/>`_.


Installing the Python library
-----------------------------

Ubidots for python is available in PyPI and you can install it from the command line:

.. code-block:: bash

    $ pip install ubidots==1.6.1

Don't forget to use sudo if necessary.

You can install pip in Linux and Mac using this command:

.. code-block:: bash

    $ sudo easy_install pip

If you don't have *easy_install*, you can get it through *apt-get* on Debian-based distributions:

.. code-block:: bash
    
    $ sudo apt-get install python-setuptools

If you are using Microsoft Windows you can install pip from `here <http://www.lfd.uci.edu/~gohlke/pythonlibs/#pip>`_.


Connecting to the API
----------------------

Before playing with the API you must be able to connect to it using your private API key, which can be found `in your profile <http://app.ubidots.com/userdata/api/>`_.

If you don't have an account yet, you can `create one here <http://app.ubidots.com/accounts/signup/>`_.

Once you have your API key, you can connect to the API by creating an ApiClient instance. Let's assume your API key is: "7fj39fk3044045k89fbh34rsd9823jkfs8323". Then your code would look like this:


.. code-block:: python

    from ubidots import ApiClient

    api = ApiClient('7fj39fk3044045k89fbh34rsd9823jkfs8323')


Now you have an instance of ApiClient ("api") which can be used to connect to the API service.

Saving a new Value to a Variable
--------------------------------

Retrieve the variable you'd like the value to be saved to:

.. code-block:: python

    my_variable = api.get_variable('56799cf1231b28459f976417')

Given the instantiated variable, you can save a new value with the following line:

.. code-block:: python

    new_value = my_variable.save_value({'value': 10})

You can also specify a timestamp (optional):

.. code-block:: python

    new_value = my_variable.save_value({'value': 10, 'timestamp': 1376061804407})

If no timestamp is specified, the API server will assign the current time to it. We think it's always better for you to specify the timestamp so the record reflects the exact time the value was captured, not the time it arrived to our servers.

Creating a DataSource
----------------------

As you might know by now, a data source represents a device or a virtual source.

This line creates a new data source:

.. code-block:: python

    new_datasource = api.create_datasource({"name": "myNewDs", "tags": ["firstDs", "new"], "description": "any des"})


The 'name' key is required, but the 'tags' and 'description' keys are optional. This new data source can be used to track different variables, so let's create one.


Creating a Variable
--------------------

A variable is a time-series containing different values over time. Let's create one:


.. code-block:: python

    new_variable = new_datasource.create_variable({"name": "myNewVar", "unit": "Nw"})

The 'name' and 'unit' keys are required.

Saving Values in Bulk
---------------------

Values may also be added in bulk. This is especially useful when data is gathered offline and connection to the internet is limited.

.. code-block:: python

   new_variable.save_values([
       {'timestamp': 1380558972614, 'value': 20},
       {'timestamp': 1380558972915, 'value': 40},
       {'timestamp': 1380558973516, 'value': 50},
       {'timestamp': 1380558973617, 'value': 30}
   ])


Getting Values
--------------

To get the values of a variable, use the method get_values in an instance of the class Variable. This will return a list like object with an aditional attribute items_in_server that tells you how many values this variable has stored on the server.

If you only want the last N values call the method with the number of elements you want.

.. code-block:: python

    # Getting all the values from the server. Note that this could result in a
    # lot of requests, and potentially violate your requests per second limit.
    all_values = new_variable.get_values()
    
    # If you want just the last 100 values you can use:
    some_values = new_variable.get_values(100)
    

Getting a group of Data sources
--------------------------------

If you want to get all your data sources you can a method on the ApiClient instance directly. This method return a Paginator object which you can use to iterate through all the items.

.. code-block:: python
    
    # Get all datasources
    all_datasources = api.get_datasources()
    
    # Get the last five created datasources
    some_datasources = api.get_datasources(5)


Getting a specific Data source
-------------------------------

Each data source is identified by an ID. A specific data source can be retrieved from the server using this ID.

For example, if a data source has the id 51c99cfdf91b28459f976414, it can be retrieved as follows:


.. code-block:: python

    my_specific_datasource = api.get_datasource('51c99cfdf91b28459f976414')


Getting a group of  Variables from a Data source
-------------------------------------------------

With a data source. you can also retrieve some or all of its variables:

.. code-block:: python

    # Get all variables
    all_variables =  datasource.get_variables()
    
    # Get last 10 variables
    some_variables =  datasource.get_variables(10)


Getting a specific Variable
------------------------------

As with data sources, you can use your variable's ID to retrieve the details about it:

.. code-block:: python

    my_specific_variable = api.get_variable('56799cf1231b28459f976417')


Managing HTTP Exceptions
-------------------------

Given that possibility that a request to Ubidots could result in an error, the API client bundles some exceptions to make easier to spot the problems. All exceptions inherit from the base UbidotsError. The full list of exceptions is:

UbidotsError400, UbidotsError404, UbidotsError500, UbidotsForbiddenError, UbidotsBulkOperationError

Each error has an attribute 'message' (a general message of the error) and 'detail' (usually JSON from the server providing more detail).

You can gaurd for these exceptions in this way:

.. code-block:: python

    try:
        my_specific_variable = api.get_variable('56799cf1231b28459f976417')
    except UbidotsError400 as e:
        print "General Description: %s; and the detail: %s" % (e.message, e.detail)
    except UbidotsForbiddenError as e:
        print "For some reason my account does not have permission to read this variable"
        print "General Description: %s; and the detail: %s" % (e.message, e.detail)

Other Exceptions
----------------

There is anoter exception UbidotsInvalidInputError wich is raised when the parameters to a function call are invalid. The required fields for the parameter of each resource in this API version are:

Datasource:
   Required:
       name: string.
   Optional:
       tags: list of strings.

       description: string.

Variables:
    Required:
        name: string.
        
        unit: string.

Values:
    Required:
        value: number (integer or float).
        
        variable: string with the variable of the id id.
    Optional:
        timestamp: unix timestamp.
