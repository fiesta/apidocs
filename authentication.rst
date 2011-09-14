Authentication
==============

Any use of the Fiesta API is done using an **API client**. You might
have a client set up for your company or for an individual
project. Each client has a set of **client credentials**: a **client
id** and a **client secret**. Those credentials are presented to
Fiesta to authenticate your client.

To get client credentials to use for your application contact
`api@corp.fiesta.cc <mailto:api@corp.fiesta.cc>`_.

The Fiesta API has two levels of authentication: **client auth** and
**user auth**. Client auth uses your client credentials to prove the
identity of your client to Fiesta. User auth is used when you want to
access resources on behalf of a specific user: you need to both prove
the identity of your client *and* prove that the user has given your
client permission to act on their behalf. We'll talk about each type
of auth in detail below.

OAuth
-----

Fiesta uses a recent draft of the `OAuth 2.0
<http://tools.ietf.org/html/draft-ietf-oauth-v2-21>`_ protocol for
authentication. We intend to update the API to use final versions of
both the OAuth 2.0 and `bearer tokens
<http://tools.ietf.org/html/draft-ietf-oauth-v2-bearer-08>`_ standards
as they are released.

Here are some high-level details about Fiesta's OAuth implementation:

Supported Grant Types

  Fiesta supports both authorization code and client credentials grants.

User Authorization Endpoint

  When requesting user authorization, use ``https://fiesta.cc/authorize``.

Token Endpoint

  When requesting token credentials, use ``https://api.fiesta.cc/token``.

.. _client-auth:

Client Auth
-----------

Client auth allows your API client to access resources on behalf of
itself. Let's go through an example - we'll access the resource
:http:get:`/hello/client`:

.. http:get:: /hello/client

   Say "hello" to an API client. Requires client auth.

First, let's try it without any authentication just like we did with
:http:get:`/hello`:

.. code-block:: console

  $ curl -i https://api.fiesta.cc/hello/client
  HTTP/1.1 401 UNAUTHORIZED
  WWW-Authenticate: Bearer realm="fiesta"
  Content-Length: 0

We get a **401 UNAUTHORIZED** response: we need client auth to access
the resource. The simplest way to use client auth is by including our
credentials using `HTTP Basic Auth <http://www.ietf.org/rfc/rfc2617.txt>`_
or URL parameters. Let's try Basic Auth first (replace *CLIENT_ID* and
*CLIENT_SECRET* with your client credentials here and throughout the
rest of this document):

.. code-block:: console

  $ curl --user CLIENT_ID:CLIENT_SECRET  -i https://api.fiesta.cc/hello/client
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "hello": "Your Client Name"
  }

If it's easier we can use URL parameters instead:

.. code-block:: console

  $ curl -i 'https://api.fiesta.cc/hello/client?client_id=CLIENT_ID&client_secret=CLIENT_SECRET'
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "hello": "Your Client Name"
  }

That's pretty much all there is to it! If you're an OAuth nerd we also
support the client credentials grant workflow. We think the above
methods are a bit easier to work with, but if you'd rather use the
OAuth flow we support the spec (no need to read on if you're happy
with the Basic Auth / URL parameters methods described above).

First use your client credentials to get a token:

.. code-block:: console

  $ curl --user CLIENT_ID:CLIENT_SECRET --data "grant_type=client_credentials" -i https://api.fiesta.cc/token
  HTTP/1.1 200 OK
  Content-Type: application/json;charset=UTF-8

  {"access_token": "...", "token_type": "bearer", "expires_in": 3600, "scope": "..."}

To get the token, we *POST* to ``https://api.fiesta.cc/token``. We
specify the **grant_type** as "client_credentials", and include our
client credentials using HTTP Basic Auth. Instead of using Basic Auth,
we could have included the credentials by including `client_id` and
`client_secret` parameters as POST data.

The response is JSON. The important bit is the **access_token**
field - let's use it to try our :http:get:`/hello/client` request again
(replace *ACCESS_TOKEN* with the token from the above response):

.. code-block:: console

  $ curl -H "Authorization: Bearer ACCESS_TOKEN" -i https://api.fiesta.cc/hello/client
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "hello": "Your Client Name"
  }

Now that we have included the access token our request works as
expected. We can continue to use the same token until it expires
(we'll get a **401 UNAUTHORIZED** when trying to use it), at which
point we just repeat the above process to get a new token.

.. _user-auth:

User Authentication
-------------------

User authentication is required when accessing resources on behalf of
a Fiesta user. At a high level, it works the same way as Client
authentication: you get a token and then include that token in the
*Authorization* header when accessing the protected resource. The
difference is in the process of acquiring the token to use - we need
to get permission from the User in question.

.. note:: To use User authentication, your client needs to specify a *Redirect URI*. This is the URI that Fiesta will redirect the user to after they authorize your application. To set a Redirect URI, visit `the settings page <https://fiesta.cc/settings>`_ and click on the "Manage" link for your client.

Let's run through an example; we'll access the resource
``https://api.fiesta.cc/hello/user``, which requires User
authentication. Let's try it first without any authentication:

.. code-block:: console

  $ curl -i https://api.fiesta.cc/hello/user
  HTTP/1.1 401 UNAUTHORIZED
  WWW-Authenticate: Bearer realm="fiesta"

We can try using a :ref:`Client authentication <client-auth>` token
(replace *ACCESS_TOKEN* with yours), too:

.. code-block:: console

  $ curl -H "Authorization: Bearer ACCESS_TOKEN" -i https://api.fiesta.cc/hello/user
  HTTP/1.1 401 UNAUTHORIZED
  WWW-Authenticate: Bearer realm="fiesta", error="invalid_token", error_description="User authentication required"

In this second case we get a specific error message - we can see we
need to use User authentication to access the resource.

Let's get a User authentication token. The first step is to redirect
the user to the authorization endpoint, including our client_id as and
``response_type=code`` as parameters. The fully constructed URL is
``https://fiesta.cc/authorize?response_type=code&client_id=client_id``
(with your client id included appropriately). When they are
redirected, the user will see a screen like this:

.. image:: authorize.png
  :align: center

If the user clicks "Deny" they will be redirected to your Redirect
URI. Fiesta will add the parameter ``error=access_denied`` to the
URI's query string, so you'll know the request was denied.

If the user clicks "Accept" they will be also be redirected to the
Redirect URI. In this case, however, the query string will include a
**code** parameter, which we can exchange for an access token (replace
*CODE* with the code you receive):

.. code-block:: console

  $ curl --user Ti8I_vyFsWj3AAAA:l1l9CbfizBBzRtkkXRUqE680G8cOW5CSp94Gb1DN --data "grant_type=authorization_code&code=CODE" -i https://api.fiesta.cc/token
  HTTP/1.1 200 OK
  Content-Type: application/json;charset=UTF-8
  Cache-Control: no-store
  Pragma: no-cache

  {"access_token": "...", "token_type": "bearer", "expires_in": 3600, "scope": "..."}

Now, let's use the access token to try our request for ``/hello/user``
again (replace *ACCESS_TOKEN* with the value you received above):

.. code-block:: console

  $ curl -H "Authorization: Bearer ACCESS_TOKEN" -i https://api.fiesta.cc/hello/user
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
    "hello": "User Name"
  }

If the user revokes your client's access, your API requests will
return **401 UNAUTHORIZED**, and you'll need to re-authorize:

.. code-block:: console

  $ curl -H "Authorization: Bearer ACCESS_TOKEN" -i https://api.fiesta.cc/hello/user
  HTTP/1.1 401 UNAUTHORIZED
  WWW-Authenticate: Bearer realm="fiesta", error="invalid_token", error_description="Revoked token"
