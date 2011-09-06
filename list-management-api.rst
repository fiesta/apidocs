List Management API
===================

The list management API allows applications to create and manage
Fiesta lists.

Basics
------

All API access is done over HTTPS.

Unless otherwise noted, all documented endpoints use the domain
``api.fiesta.cc``.

All transmitted data is JSON.


Authentication
--------------

For details on authenticating with the API, see
:doc:`authentication`. All endpoints state whether they require
:ref:`client <client-auth>` or :ref:`user <user-auth>`
authentication. User authentication is strictly stronger than client
authentication.

Datatypes
---------

.. _address:

Address
~~~~~~~

.. code-block:: js

  {
    address: ADDRESS,
    user: USER
  }

:ADDRESS: An email address
:USER: URI of the address' owner.

.. _user:

User
~~~~

.. code-block:: js

  {
    address: ADDRESS,
    name: NAME
  }

:ADDRESS: Email address of the user.
:NAME: Name of the user.

.. _group:

Group
~~~~~

.. code-block:: js

  {
    creator: CREATOR,
    name: GROUP_NAME,
    members: [ MEMBERS ],
    domain: DOMAIN
  }

:CREATOR: A :ref:`user` representing the creator of this list.
:GROUP_NAME: The name of the list. Must be <= 30 characters long.
:MEMBERS: An array of list members, as :ref:`user` instances.
:DOMAIN: An optional hostname, if this list is using Fiesta for custom domains.

Endpoints
---------

.. http:get:: /hello

    Say hello.

    This method exists for testing and documentation
    examples. Requires no authentication.

.. http:get:: /hello/client

    Say hello to the authorized client.

    This method exists for testing and documentation
    examples. Requires :ref:`client-auth`.

.. http:get:: /hello/user

    Say hello to the authorized user.

    This method exists for testing and documentation
    examples. Requires :ref:`user-auth`.

.. http:get:: /address/(string:address)

    Get information about an email address (`address`).

    Requires :ref:`client-auth`.

    :param address: Email address to look up
    :status 200: Returns an :ref:`address` object.
    :status 404: The given address doesn't belong to any Fiesta user.

.. http:post:: /group

    Create a new list. The request body is a :ref:`group`.

    If `creator` is not an existing Fiesta user, :ref:`client-auth` is
    required and a verification email will be sent to the creator to
    confirm list creation.

    If `creator` is an existing Fiesta user, :ref:`user-auth` is
    required and a verification email will be sent to the creator to
    confirm list creation.

    If the API client is a *trusted client*, `creator` is not
    required. If `creator` is not present, only :ref:`client-auth` is
    required. Contact `api@corp.fiesta.cc
    <mailto:api@corp.fiesta.cc>`_ for information on becoming a
    trusted client.
