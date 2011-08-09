Authentication
==============

Fiesta uses `OAuth 1.0 <http://tools.ietf.org/html/rfc5849>`_ for
authentication. API clients must possess a valid set of *client
credentials* in order to interact with the API. Contact
`api@corp.fiesta.cc <mailto:api@corp.fiesta.cc>`_ to get client
credentials to use for your application.

Details
-------

Here are some high-level details about Fiesta's OAuth implementation:

Parameter Transmission

  The only currently supported mechanism for transmitting OAuth
  protocol parameters is the HTTP Authorization header.

Realm

  The only currently supported value for the *realm* parameter is
  **all**, which will grant full API access.

Signature Method

  The only currently supported signature method is **PLAINTEXT**.

Temporary Credential Request URI

  When requesting temporary credentials, use :http:method:`initiate`.

User Authorization URI

  When requesting user authorization, use
  :http:method:`authorize`.

Token Request URI

  When requesting token credentials, use :http:method:`token`.

Endpoints
---------

.. http:method:: POST /oauth/initiate
  :label-name: initiate
  :title: /oauth/initiate

  Get a form-encoded set of temporary credentials.

.. http:method:: GET https://fiesta.cc/oauth/authorize?oauth_token
  :label-name: authorize
  :title: https://fiesta.cc/oauth/authorize

  :param oauth_token: ID of temporary credentials to be authorized.

  The URI that a user should be redirected to for authorization.

  .. note:: This URI uses the **fiesta.cc** domain rather than
    **api.fiesta.cc**.

.. http:method:: POST /oauth/token
  :label-name: token
  :title: /oauth/token

  Exchange a set of temporary credentials and a verifier for a set of
  token credentials.

.. _two-legged:

Two-legged Authentication
-------------------------

Two-legged authentication allows an API client to access resources on
behalf of itself. To authenticate, construct an HTTP Authorization
header with the following parameters, each of which should be URL
encoded. Include the constructed header in your request.

- `oauth_version`: ``"1.0"``

- `oauth_consumer_key`: Your client id.

- `oauth_signature_method`: ``"PLAINTEXT"``

- `oauth_signature`: The concatenation of your client secret and the
  ampersand (``'&'``) character.

You may optionally include the `realm` parameter. If present, the
`realm` must be ``"all"``.

Here is an example Authorization header, which is valid for a client
whose id is *example* and secret is *secret*:

``OAuth realm="all", oauth_signature="secret%26", oauth_consumer_key="example", oauth_signature_method="PLAINTEXT", oauth_version="1.0"``

.. _three-legged:

Three-legged Authentication
---------------------------

Three-legged authentication is required when accessing resources on
behalf of a Fiesta user. Any three-legged access requires valid token
credentials in the Authorization header. The process of acquiring
token credentials is thoroughly documented in `RFC 5849
<http://tools.ietf.org/html/rfc5849>`_, but the following is quick
overview:

#. Client retrieves temporary credentials from :http:method:`initiate`.

#. Client redirects user to :http:method:`authorize`.

#. User authorizes client, and is redirected to the client's callback URI.

#. Client exchanges temporary credentials and verifier for token credentials: :http:method:`token`.

#. Client uses token credentials to construct an Authorization header that allows access to the user's resources.
