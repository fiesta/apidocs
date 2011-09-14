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

Endpoints (URIs)
----------------

.. http:post:: /group

    Create a new mailing list. Requires :ref:`user-auth` with READ scope.

    Input:

    .. code-block:: js

      {
        creator: {
                   address: EMAIL_ADDRESS,
                   name: STRING (optional)
                 },

        domain: STRING (optional),
        description: STRING (optional)
      }

    Members are added separately with a different API call.

    The ``address`` is the email address for the creating user.

    The ``name`` is an optional display name to be attached to the
    inputted email address.

    The ``domain`` is to be supplied if the mailing list is for a
    whitelabeled domain instead of using fiesta.cc. Contact 
    api@corp.fiesta.cc for information on becoming whitelabeled.

    A ``description`` is used in place of the standard Fiesta notification
    when adding new members.

    Returns:

    .. code-block:: js

      {
        status: {
                  code: INT
                  message: STRING (optional)
                },
        group_id: STRING,
        uri: URI,
        description: STRING,
        members: URI
      }

    The ``group_id`` is the identifier Fiesta has assigned to the group.
    The ``group_id`` is your handle for retrieving or modifying any group 
    related information.

    The ``status`` is a code whose meaning can be found _here_.

    The ``uri`` is a location which the group information cab be found.

    The ``description`` will be echoed if it was provided.

    ``Members`` is a location where members of the group can be retrieved.

.. http:get:: /group/(string: group_id)

   Retrieve information of a group. This call requires :ref:`client-auth`
   to be the creator of the group or :ref:`user-auth` of a member from the
   group with READ scope.

   The returned information models the group datatype.

.. http:post:: /membership/(string: group_id)

   Add a new membership linking a user and a group. The request body
   consists of a JSON `user` and a `group_id`.

   A custom welcome message is optional by adding a `welcome_message` dict
   that may have the following fields: `subject`, `text` and/or `markdown`.

   The returned information models the `membership` datatype.

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
