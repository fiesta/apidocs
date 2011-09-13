List Management API
===================

The list management API allows applications to create and manage
Fiesta lists.

Basics
------

All API access is done over HTTPS.

Unless otherwise noted, all documented endpoints (URIs) use the domain
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

.. _user:

User
~~~~

.. code-block:: js

  {
    address: ADDRESS,
    name: NAME
  }

:ADDRESS: Email address of the user.
:NAME: Name of the user. (Optional)

.. _group:

Group
~~~~~

.. code-block:: js

  {
    creator: CREATOR,
    members: [ MEMBERSHIPS ],
    domain: DOMAIN,
    description: DESCRIPTION
  }

:CREATOR: A :ref:`user` representing the creator of this list.
:MEMBERSHIPS: An array of list memberships, as :ref:`membership` instances.
:DOMAIN: An optional hostname, if this list is using Fiesta for custom domains.
:DESCRIPTION: An optional description for the group.

.. _membership:

Membership
~~~~~~~~~~

.. code-block:: js

  {
    user_id: USER_ID,
    group_id: GROUP_ID,
    list_name: LIST_NAME,
    tags: [ TAG ]
  }

:USER_ID: A :ref:`user` representing a member of this list.
:GROUP_ID: A :ref:`group` representing the group.
:LIST_NAME: The group name the user uses to mail the list.
:TAGS: An array of optional tags that may apply to a member e.g. muted.

Endpoints (URIs)
----------------

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

.. http:post:: /group

    Create a new list. The request body consists of a JSON :ref:`group`
    without the `members` field. Members are added with a different
    endpoint.

    If `creator` is not an existing Fiesta user, :ref:`client-auth` is
    required and a verification email will be sent to the creator to
    confirm list creation.

    If `creator` is an existing Fiesta user, :ref:`user-auth` is
    required and a verification email will be sent to the creator to
    confirm list creation.

    The `domain` is to be supplied if the mailing list is for a
    whitelabeled domain instead of using fiesta.cc. Contact 
    api@corp.fiesta.cc for information on becoming whitelabeled.

    A `description` is used in place of the standard Fiesta notification
    when adding new members.

    *IMPORTANT* The returning JSON contains a copy of the created group
    including the `group_id` Fiesta has assigned to the group. The 
    `group_id` is your handle for retrieving or modifying any group 
    related information.

.. http:get:: /group/(string: group_id)

   Retrieve information of a group. This call requires :ref:`client-auth`
   to be the creator of the group or :ref:`user-auth` of a member from the
   group with READ scope.

   The returned information models the :ref:`group` datatype.

.. http:post:: /membership/(string: group_id)

   Add a new membership linking a user and a group. The request body
   consists of a JSON :ref:`user` and a `group_id`.

   A custom welcome message is optional by adding a `welcome_message` dict
   that may have the following fields: `subject`, `text` and/or `markdown`.

   The returned information models the :ref:`membership` datatype.

.. http:get:: /user/(string: user_id)

   Retrieve information for a user. This call requires :ref:`user-auth`
   with the READ scope.

   The returned JSON object includes a name, a list of email addresses and
   a URI linking to the list of memberships.

.. http:get:: /groups_for/(string: user_id)

   Returns a list of all the membership URIs for a particular user.

   This call requires :ref:`user-auth` with a READ scope.

.. http:get:: /users_for/(string: group_id)

   Returns a list of all the membership URIs for a particular group.

   This call requires the same authentication as getting information
   for the group.
