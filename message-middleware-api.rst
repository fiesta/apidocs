Message Middleware API
======================

The Message Middleware API allows developers to create applications
that can modify or intercept messages as they are sent to existing
Fiesta lists.

A valid set of client credentials is required to use the message
middleware API. To get client credentials, contact
`api@corp.fiesta.cc <mailto:api@corp.fiesta.cc>`_.

.. _message-hook:

Message Hook
------------

The main point of interaction for the API is a *message hook*
specified by the application developer. The message hook is a URI that
Fiesta will post message data to. To set a message hook, visit `the
settings page <https://fiesta.cc/settings>`_ and click the "Manage"
link for your client.

Activation
----------

In order for a message to be posted to your application's message
hook, your application must be *activated* for that message. There are
several ways that an application can be activated:

- Per-message activation occurs when a message is sent to a group
  using a registered *plustag*. An example: your application has
  registered the plustag **+example** and a user sends a message to
  **list_name+example@fiesta.cc**.

- Per-list activation occurs when an application has been installed on
  a given Fiesta list and a message is sent to that list.

- Per-domain activation occurs when a message is sent to a list using
  a Fiesta custom domain, and one of the domain's managers has
  installed the application on that domain. Follow these steps to
  install your application on a domain you can manage:

  #. Be sure you have a :ref:`message-hook` set for your application.
  #. Visit `the settings page <https://fiesta.cc/settings>`_ and click
     the "Manage" link for your domain.
  #. Select your application from the drop-down and click "Add app".

Incoming Messages
-----------------

Incoming messages are posted to your application's message hook as
JSON data. Here's an example:

.. code-block:: js

  {
    group_id: GROUP_ID,
    group_uri: GROUP_URI,
    group_name: "family",
    sender_id: USER_ID,
    sender_uri: USER_URI,
    subject: "Hi guys",
    text: "This is an example message."
  }

`group_id` is the group id of the group the message was sent to.

`group_uri` is a URI you can use to get more information about the
group.

`group_name` is the name used by the sender for the group.

.. note:: Different users might have different names for the same
   group, and different groups can also share the same name for
   different users. In short, use `group_name` for display purposes
   but don't use it as an ID.

`sender_id` is the user id of the message's sender.

`sender_uri` is a URI that you can use to get more information about
the sender.

`subject` is the message's subject.

`text` is a text representation of the message's content. The message
text is processed to remove signatures and quoted text.

The messages are posted as raw JSON, not the default
``application/x-www-form-urlencoded``. To get the data in PHP, you can
do something like this:

.. code-block:: php

  $message = json_decode(file_get_contents("php://input"))

Responses
---------

There are several valid responses to an incoming message.

Responding with HTTP status code **204** (No Content) means that your
application does not wish to make any changes to the normal Fiesta
message-handling workflow. The message will be handled by Fiesta
as-is.

Responding with HTTP status code **202** (Accepted) means that your
application has handled the message, and Fiesta does not need to take
further action. The message will not be sent to the list or processed
by any downstream applications.

Responding with HTTP status code **200** (OK) means that your
application is responding with a (possibly modified) message to be
sent to the list. The response body is a JSON document describing the
message that Fiesta should send:

.. note:: Be sure to set the *Content-Type* header to ``application/json``.

.. code-block:: js

  {
    subject: "Hi again",
    text: "This is another example message."
  }

If either `subject` or `text` is not present the default is the
corresponding value as originally posted to your application. If you
want to include rich content (like links or basic styling), include a
`markdown` key instead of `text`:

.. code-block:: js

  {
    subject: "Hi again",
    markdown: "Hi. **This part is bold.**"
  }

`markdown` will be processed by a `Markdown
<http://daringfireball.net/projects/markdown/syntax>`_ processor to
generate an HTML version of the email.

Errors
------

If your message hook returns any non-2xx status code, or if our
attempt to reach your message hook causes a time-out, the message will
be processed by the normal Fiesta message-handling workflow.

Security / Authorization
------------------------

The use of HTTPS for your message hook is recommended, but not
required.

Fiesta signs all of its requests to your message hook, so you can
verify that posted messages are actually from Fiesta. There are three
relevant header fields included with each request:

- `X-Fiesta-Timestamp`: A UNIX timestamp (seconds since the epoch,
  UTC) generated by Fiesta before posting a message to your message
  hook.

- `X-Fiesta-Nonce`: A nonce generated for the request. Guaranteed to
  be unique per second.

- `X-Fiesta-Signature`: A hexadecimal HMAC-SHA256 signature.

The signature is constructed using your client secret as the HMAC
key. The message that gets signed is the concatenation of the nonce,
timestamp, and POST body. To verify the message, construct the HMAC
signature (using SHA-256 mode) and verify that the resulting hexdigest
matches the value of the `X-Fiesta-Signature` header. To prevent
replay attacks, you can optionally check that the timestamp is recent
and that the (timestamp, nonce) pair has not been used before.
