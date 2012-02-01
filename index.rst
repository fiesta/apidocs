Fiesta API Documentation
========================

Overview
--------

This documentation describes the official `Fiesta
<https://fiesta.cc>`_ API. The API is in active development; for
questions and support please sign up for the `Fiesta API mailing list
<https://fiesta.cc/~api>`_.

The purpose of the Fiesta API is to allow programmatic creation and
management of email groups, or lists. The members of a list can
interact with the list by sending emails to a list address. By
default, an email sent to a list address will be forwarded to the
other members of the list (just like a traditional mailing list). The
Fiesta API, however, gives developers the option of altering the way
messages are handled by changing the content of messages before they
are forwarded or turning off forwarding altogether.

Getting Started
---------------

Let's get started with a simple "hello world" example, which should
illustrate three things:

#. All API access is done using HTTPS.

#. All API **endpoints** (URIs) use the domain ``api.fiesta.cc``.

#. All transmitted data is `JSON <http://www.json.org/>`_.

We're going to use the endpoint :http:get:`/hello`:

.. http:get:: /hello

   Say "hello". Does not require authentication.

Let's use ``curl`` to make the API request (we're using the ``-i``
flag so that we'll see HTTP response headers, too):

.. code-block:: console

  $ curl -i https://api.fiesta.cc/hello
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "hello": "world"
  }

Congratulations, you just made your first Fiesta API request ☺.

Next Steps
----------

Developers should start by checking out the Fiesta :doc:`sandbox`,
which allows testing and experimentation in a low-impact environment.

The next step for developers is to learn about how Fiesta handles
:doc:`authentication` for API clients.

After that, learn about the :doc:`list-management-api`, which is the
API that enables creating new lists or updating existing ones.

Finally, to change the behavior of a list as messages are sent to it,
check out the :doc:`message-middleware-api`.

To see recent changes to the API, check out the :doc:`changelog
</changelog>`.

Contributing
------------

This documentation is a community effort; if you are interested in
contributing check out the source on `Github
<http://github.com/fiesta/apidocs>`_.


.. toctree::
   :hidden:

   sandbox
   authentication
   list-management-api
   message-middleware-api
   changelog

