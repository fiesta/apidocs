List Management API
===================

The list management API allows applications to create and manage
Fiesta lists.

Basics
------

All API access is done over HTTPS.

Unless otherwise noted, all endpoints use the domain
``api.fiesta.cc``.

All transmitted data is JSON.


Authentication
--------------

The Fiesta API uses `OAuth 1.0 <http://tools.ietf.org/html/rfc5849>`_
for authentication. See :doc:`authentication` for details on Fiesta's
OAuth implementation.

Many resources require only :ref:`two-legged <two-legged>` OAuth,
meaning that the API client is acting on behalf of itself rather than
an individual Fiesta user (resource owner). The flow for two-legged
OAuth is significantly simpler than the traditional :ref:`three-legged
<three-legged>` flow, which is required when accessing user-specific
resources. This documentation will always state whether an endpoint
requires two- or three-legged OAuth.

Email Addresses
---------------

.. http:method:: GET /address/{address}

  :arg address: Email address to look up
  :response 200: An :http:response:`address`.
  :response 404: The given address doesn't belong to any Fiesta user.

  Look up information about an email address. Requires
  :ref:`two-legged <two-legged>` authentication.

.. http:response:: Address

  .. code-block:: js

    {
      address: ADDRESS,
      user: USER
    }

  :data string ADDRESS: An email address
  :data string USER: User id / URI of the address' owner.
  :format: JSON

List Creation
-------------

.. http:method:: POST /group

  Create a new list. The request body is a :http:response:`group`.

  If `creator` is not an existing Fiesta user, :ref:`two-legged
  <two-legged>` authentication is required and a verification email
  will be sent to the creator to confirm list creation.

  If `creator` is an existing Fiesta user, :ref:`three-legged
  <three-legged>` authentication is required and a verification email
  will be sent to the creator to confirm list creation.

  If the API client is a *trusted client*, `creator` is not
  required. If `creator` is not present, only :ref:`two-legged
  <two-legged>` authentication is required. Contact
  `api@corp.fiesta.cc <mailto:api@corp.fiesta.cc>`_ for information on
  becoming a trusted client.

.. http:response:: Group

  .. code-block:: js

    {
      creator: CREATOR,
      name: GROUP_NAME,
      members: [ MEMBERS ],
      domain: DOMAIN
    }

  :data CREATOR: A :http:response:`user` representing the creator of this list.
  :data string GROUP_NAME: The name of the list. Must be <= 30 characters long. Must contain only ASCII characters, digits, '.', '-', or '_'. Must start and end with an ASCII character or digit.
  :data MEMBERS: An array of list members, as :http:response:`user` instances. Must be non-empty.
  :data string DOMAIN: An optional hostname, if this list is using Fiesta for custom domains.
  :format: JSON

.. http:response:: User

  .. code-block:: js

    {
      address: ADDRESS,
      name: NAME
    }

  :data string ADDRESS: Email address of the user.
  :data string NAME: Name of the user.
  :format: JSON
