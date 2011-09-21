List Management API
===================

The list management API allows applications to create and manage
Fiesta lists.

Authentication
--------------

For details on authenticating with the API, see
:doc:`authentication`. All endpoints state whether they require
:ref:`client <client-auth>` or :ref:`user <user-auth>` auth. User auth
is strictly stronger than client auth (if an endpoint requires client
auth and you have a user auth token that will work, but the opposite
won't). Endpoints that require user auth also state the required
scope.

Groups and Users
----------------

The most important concept for working with the Fiesta API is that of
a list. Conceptually, a list is just a group of members, so the API
uses the word **group** (and group_id, group_uri, etc.) when talking
about lists. A **user** is an individual person who can belong to a
Fiesta list.

The connection between groups and users is represented using a third
core structure, called a **membership**. This extra layer of
indirection is required because users can belong to groups in
different ways. E.g. each member of a list can use a different list
address (we call that a **group_name**) for the same list. The
group_name is stored in each membership, rather than in the group
itself.

Creating a Group
----------------

Let's start by creating a new list. This is a two step process: first
we'll create a group and then we'll add memberships to the
group. We'll start with :http:post:`/group`:

.. http:post:: /group

    Create a new mailing list. Requires :ref:`user-auth` with "create"
    scope.

    Input (as JSON POST data with *Content-Type*
    ``application/json``):

    .. code-block:: js

      {
        creator: {
                   group_name: STRING,
                   address: EMAIL_ADDRESS,
                   display_name: STRING (optional),
                   welcome_message: WELCOME_MESSAGE (optional)
                 },
        domain: STRING (optional),
        description: STRING (optional)
      }

    `group_name` is the group name that will be used for the group's
    creator. If you're creating a list called "family\@fiesta.cc",
    `group_name` should be "family".

    `address` is the email address of the group's creator. This
    address must be owned by the authenticated user.

    `display_name` (optional) is the name that will be displayed for
    the group's creator throughout the Fiesta UI. If it's included and
    the creator does not yet have a display name, it will be set.

    `welcome_message` (optional) is a :ref:`custom welcome message
    <message>`. If not present, the default Fiesta welcome message
    will be sent to the creator: this includes some basic info about
    the list and the list description. If `welcome_message` is a
    :ref:`Message <message>` instance, the specified message will be sent to the
    creator instead. If it's ``null`` or ``false`` no welcome message
    will be sent.

    `domain` (optional) is the domain to use for the list address. The
    default is "fiesta.cc". To use a custom domain your client must
    have permission for that domain: contact api@corp.fiesta.cc for
    information on using custom domains.

    `description` (optional) is a short (maximum of 200 characters)
    description of the list. It is included in the default welcome
    message that is sent to new list members, and elsewhere in the
    Fiesta UI.

    Returns the following JSON data in the response body:

    .. code-block:: js

      {
        status: {
                  code: INT,
                  message: STRING (sometimes present)
                },
        location: URI,
        data: {
                group_id: STRING,
                group_uri: URI,
                domain: STRING,
                description: STRING,
                members: URI
              }
      }

    The status `code` is a numeric code that will match the response's
    `HTTP status code
    <http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html>`_. It
    will be ``201`` if the group was created successfully. It will be
    ``202`` if the group was created but is still pending activation
    by the group's owner (they'll need to click a link in an email
    they were sent).

    `message` will be included if there is an additional explanation
    of the status code.

    `group_id` is a unique string that Fiesta has assigned as an
    identifier for the group. This is the handle you'll need for
    subsequent interactions with the group, so it's often a good idea
    to store it somewhere.

    `location` and `group_uri` is the endpoint to use to get
    information about the group. This value will also be present as
    the HTTP *Location* header.

    `members` is the endpoint to use to get a list of group members or
    add another member to this group.

    `description` and `domain` are as described above for the method's
    input.

After creating the group, our list will have a single membership: the
group's creator. Let's add another member using the `members` URI that
was returned above:

    .. note:: Trusted clients may create a group without user access. To do
              this the client must not supply a creator. The only available
              paramters are a domain and a group description. If a creator
              is supplied, a user access token is expected.

.. http:post:: /membership/(string: group_id)

    Add a group membership. Requires :ref:`user-auth` with "modify"
    scope.

    The authenticated user must be a member of the group identified by
    `group_id`.

    Input (as JSON POST data with *Content-Type*
    ``application/json``):

    .. code-block:: js

      {
        group_name: STRING,
        address: EMAIL_ADDRESS,
        display_name: STRING (optional),
        welcome_message: WELCOME_MESSAGE (optional)
      }

    `group_name` is the group name that will be used for the new
    member. If you're creating a list called "family\@fiesta.cc",
    `group_name` should be "family".

    `address` is the email address of the new member.

    `display_name` (optional) is the name that will be displayed for
    the new member throughout the Fiesta UI. If it's included and the
    member does not yet have a display name, it will be set.

    `welcome_message` (optional) is a :ref:`custom welcome message
    <message>`. If not present, the default Fiesta welcome message
    will be sent: this includes some basic info about the list and the
    list description. If `welcome_message` is a :ref:`Message <message>`
    instance, the specified message will be sent to the new member
    instead. If it's ``null`` or ``false`` no welcome message will be
    sent.

    Returns the following JSON data in the response body:

    .. code-block:: js

      {
        status: {
                  code: INT,
                  message: STRING (sometimes present)
                },
        location: URI,
        data: {
                membership_uri: URI
                group_id: STRING,
                group_uri: URI,
                user_id: STRING,
                user_uri: URI,
                group_name: STRING,
              }
      }

    The status `code` is a numeric code that will match the response's
    `HTTP status code
    <http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html>`_. It
    will be ``201`` if the member was added successfully. It will be
    ``202`` if the member was added but the group is still pending
    activation by the group's owner (they'll need to click a link in
    an email they were sent). It will be ``204`` if the member was not
    added (generally because the address is already a group member).

    `message` will be included if there is an additional explanation
    of the status code.

    `location` and `membership_uri` is the endpoint to use to get
    information about the membership. This value will also be present
    as the HTTP *Location* header.

    `group_id` and `group_uri` are the ID and URI of the group.

    `user_id` and `user_uri` are the ID and URI of the (possibly newly
    created) user.

    `group_name` is the name of the group as used by this user.

    .. note:: Trusted clients do not need :ref:`user-auth` for this API
              call. A trusted client only needs to have created the group
              to be able to add members to the group.

.. _message:

Messages
--------

A message is a representation of an email for Fiesta to send. It
should be a JSON object with one or more of the following fields:

.. code-block:: js

  {
    subject: STRING,
    text: STRING,
    markdown: STRING
  }

`subject` is the subject to use for the message.

`text` is a plain-text body to use for the message. It will be used if
present.

`markdown` is a `Markdown
<http://daringfireball.net/projects/markdown/syntax>`_ formatted body
to use for the message. If it is present and `text` is absent,
`markdown` will be used for the the body of the message. An HTML
version of the email, generated from the Markdown, will also be
included.

Removing a List Member
----------------------

To remove a member from the list just issue a DELETE request on the membership URI:

.. http:delete:: /membership/(string: group_id)/(string: user_id)

    Remove a group membership. Requires :ref:`user-auth` with "modify"
    scope.

    The authenticated user must be a member of the group identified by
    `group_id`.

    Responds with status code ``200`` if the membership either didn't
    exist or was successfully removed.

Getting Group/User Information
------------------------------

.. http:get:: /group/(string: group_id)

   Retrieve information of a group. This call requires :ref:`client-auth`
   to be the creator of the group or :ref:`user-auth` of a member from the
   group with READ scope.

   The returned information models the group datatype.

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
