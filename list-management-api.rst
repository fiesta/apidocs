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
    scope when creating on behalf of a specific user. When creating on
    behalf of your client requires only :ref:`client-auth`.

    Input (as JSON POST data with *Content-Type*
    ``application/json``):

    .. code-block:: js

      {
        creator: {
                   group_name: STRING,
                   address: EMAIL_ADDRESS,
                   display_name: STRING (optional),
                   welcome_message: WELCOME_MESSAGE (optional)
                 } (optional),
        domain: STRING (optional),
        description: STRING (optional)
      }

    `creator` is a block of information about the list's creator. If
    `creator` is present the list is being created on behalf of a
    specific user, so :ref:`user-auth` is required. If the list is
    being created on behalf of your client, the `creator` field should
    **not** be present. In that case, only :ref:`client-auth` is
    necessary.

    `group_name` is the group name that will be used for the group's
    creator. If you're creating a list called "family\@fiesta.cc",
    `group_name` should be "family".

    `address` is the email address of the group's creator
    (e.g. "mike@corp.fiesta.cc"). This address must be owned by the
    authenticated user.

    `display_name` (optional) is the name that will be displayed for
    the group's creator throughout the Fiesta UI (e.g. "Mike
    Dirolf"). If it's included and the creator does not yet have a
    display name, it will be set.

    `welcome_message` (optional) is a :ref:`custom welcome message
    <message>`. If not present, the default Fiesta welcome message
    will be sent to the creator: this includes some basic info about
    the list and the list description. If `welcome_message` is a
    :ref:`Message <message>` instance, the specified message will be sent to the
    creator instead. If it's ``null`` or ``false`` no welcome message
    will be sent.

    `domain` (optional) is the domain to use for the list address. The
    default is "fiesta.cc". To use a custom domain your client must
    have permission for that domain: check out `Fiesta Custom
    <https://fiesta.cc/custom>`_ for setting up custom domains.

    `description` (optional) is a short (maximum of 200 characters)
    description of the list. It is included in the default welcome
    message that is sent to new list members, and elsewhere in the
    Fiesta UI. If you are making a group for the Fantastic Four the
    group_name could be something like "f4" and the description could
    be "Fantastic Four".

    Returns the following JSON data in the response body:

    .. code-block:: js

      {
        status: {
                  code: INT,
                  message: STRING (sometimes present)
                },
        location: URI,
        data: {
                group_id: GROUP_ID,
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

Adding Members
--------------

After creating the group, our list will have a single membership: the
group's creator. Let's add another member using the `members` URI that
was returned above:

.. http:post:: /membership/(string: group_id)

    Add a group membership. Requires :ref:`user-auth` with "modify"
    scope. The authenticated user must be a member of the group
    identified by `group_id`. Alternatively, the group must have been
    created by the authorized client - in that case only
    :ref:`client-auth` is required.

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
                group_id: GROUP_ID,
                group_uri: URI,
                user_id: USER_ID,
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
    an email they were sent). It will be ``200`` if the member was not
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

Sending Messages
----------------

Now that we've added a few members to the group, let's see how to send
messages to them.

.. http:post:: /message/(string: group_id)

    Send a message to a group.

    Requires :ref:`user-auth` with "message" scope. The authenticated
    user must be a member of the group identified by `group_id`. The
    email is sent on behalf of the authenticated user. Alternatively,
    the group must have been created by the current client - in that
    case, only :ref:`client-auth` is required.

    Input (as JSON POST data with *Content-Type*
    ``aplication/json``):

    .. code-block:: js

      {
        message: MESSAGE
      }

    `message` is a :ref:`message` object.

    Returns:

    .. code-block:: js

      {
        status: {
                  code: INT,
                  message: STRING (sometimes present)
                },
        data: {
                group_id: GROUP_ID,
                group_uri: URI,
                message_id: STRING,
                thread_id: STRING,
                message: MESSAGE,
              }
      }

    The status `code` is a numeric code that will match the response's
    `HTTP status code
    <http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html>`_. It
    will be ``200`` if the message was sent successfully. It will be
    ``400`` if the message failed to send.

    `message (Status)` will be included if there is an additional explanation
    of the status code.

    `group_id` and `group_uri` are the ID and URI of the group.

    `message_id` is a unique identifier (as a string) assigned to the
    sent message.

    `thread_id` is a unique identifier (as a string) assigned to the
    thread created by the sent message.

    `message (Data)` will be a :ref:`Message <message>` representing
    the email sent.


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

.. note:: There is currently no way to include raw HTML in
   messages. The best alternative is to send the message as markdown,
   which will be used to generate HTML. If you feel you need raw HTML,
   please get in touch on the `support list
   <https://fiesta.cc/~api>`_.

Removing a List Member
----------------------

To remove a member from the list just issue a DELETE request on the membership URI:

.. http:delete:: /membership/(string: group_id)/(string: user_id)

    Remove a group membership.

    Requires :ref:`user-auth` with "modify" scope. The authenticated
    user must be a member of the group identified by
    `group_id`. Alternatively, the group must have been created by the
    current client - in that case, only :ref:`client-auth` is
    required.

    Responds with status code ``200`` if the membership either didn't
    exist or was successfully removed.

Getting Group/User Information
------------------------------

.. http:get:: /group/(string: group_id)

   Retrieve information about a group.

   Requires :ref:`user-auth` with "read" scope. The authenticated user
   must be a member of the group identified by
   `group_id`. Alternatively, the group must have been created by the
   current client - in that case, only :ref:`client-auth` is required.

   Returns:

   .. code-block:: js

     {
       group_id: GROUP_ID,
       group_uri: URI,
       domain: STRING,
       description: STRING,
       members: URI
     }

.. http:get:: /membership/(string: group_id)/(string: user_id)

   Retrieve specific information on a membership between a group and member.

   Requires :ref:`user-auth` with "read" scope. The authenticated user
   must be a member of the group identified by
   `group_id`. Alternatively, the group must have been created by the
   current client - in that case, only :ref:`client-auth` is required.

   Returns:

   .. code-block:: js

     {
       group_id: GROUP_ID,
       group_uri: URI,
       user_id: USER_ID,
       user_uri: URI
     }

   If the authorized user is the user has id `user_id`, the group name
   for the user is also included:

   .. code-block:: js

     {
       group_id: GROUP_ID,
       group_uri: URI,
       user_id: USER_ID,
       user_uri: URI,
       group_name: STRING
     }

.. http:get:: /membership/(string: group_id)

   Retrieve a list of all the membership URIs for a particular group.

   Requires :ref:`user-auth` with "read" scope. The authenticated user
   must be a member of the group identified by
   `group_id`. Alternatively, the group must have been created by the
   current client - in that case, only :ref:`client-auth` is required.

   Returns:

   .. code-block:: js

     {
       memberships: [{
                       group_id: GROUP_ID,
                       group_uri: URI,
                       user_id: USER_ID,
                       user_uri: URI,
                       membership_uri: URI
                     }, ...]
     }


.. http:get:: /user/(string: user_id)

   Retrieve information about a user.

   The response includes a list of scopes the user with id `user_id`
   has authorized for this client:

   .. code-block:: js

     {
       user_id: USER_ID,
       scopes: SCOPES
     }

   If the client has :ref:`user-auth` with "read" scope, and the
   authenticated user has id `user_id`, additional information is
   returned:

   .. code-block:: js

     {
       user_id: USER_ID,
       scopes: SCOPES
       name: STRING,
       memberships: URI,
     }


.. http:get:: /groups_for/(string: user_id)

   Returns a list of all the memberships for a particular user.

   Requires :ref:`user-auth` with "read" scope.

   Returns:

   .. code-block:: js

     {
       memberships: [{
                       group_id: GROUP_ID,
                       group_uri: URI,
                       user_id: USER_ID,
                       user_uri: URI,
                       membership_uri: URI
                     }, ...]
     }
