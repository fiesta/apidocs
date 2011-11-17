API Sandbox
===========

We maintain an API "sandbox" at ``sandbox.fiesta.cc``. This allows API
clients to test interacting with Fiesta, without actually sending any
emails or modifying any real groups or user accounts.

Usage
-----

To use the sandbox, just replace all references to ``api.fiesta.cc``
with ``sandbox.fiesta.cc``. Your client will still need to
authenticate, and you'll use the same credentials as you would with
the production API server. All calls should still use HTTPS.

Special Endpoints
-----------------

The sandbox supports several endpoints that aren't present in the
production API:

.. http:get:: /mailbox

   Get information about the messages that would have been sent as a
   result of your API calls, had this been running out of the sandbox.

   Returns a JSON array with keys for each email address that has been
   messaged. The values are arrays of messages, each with a ``"text"``
   and ``"subject"`` field:

   .. code-block:: js

     {
       mike@corp.fiesta.cc: [
         {
           text: "Hello world.",
           subject: "Saying hello"
         },
         ...
       ],
       ...
     }

.. http:post:: /reset

   Reset any sandbox state changes made by your client.

   This will empty your client's "mailbox" and remove any sandbox
   groups or users that have been created as a result of your client's
   calls.

   Response:

   .. code-block:: js

     {
       "reset": true
     }

Notes
-----

The sandbox does share some state among clients, so it's possible to
see some changes in behavior based on activities of other clients.

Sandbox state should be treated as temporary. We will try to only
erase state after days of inactivity or an explicit call to
:http:post:`/reset`, but we reserve the right to erase any sandbox
state at any time if it makes our lives significantly easier :).
