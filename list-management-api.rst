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

The Fiesta API uses `OAuth 1.0 <http://tools.ietf.org/html/rfc5849>`_ for authentication. Contact `api@corp.fiesta.cc <mailto:api@corp.fiesta.cc>`_ to get client credentials to use for your application.

Many of resources require only *two-legged* OAuth, meaning that the API client is acting on behalf of itself rather than an individual Fiesta user (resource owner). The flow for two-legged OAuth is significantly simpler than the traditional *three-legged* flow, which is required when accessing user-specific resources.

Signature Method

  The only currently supported signature method is **PLAINTEXT**.

Temporary Credential Request URI

  When requesting temporary credentials, use :http:method:`initiate`.

User Authorization URI

  When requesting user authorization, use
  :http:method:`authorize`.

Token Request URI

  When requesting token credentials, use :http:method:`token`.

Parameter Transmission

  The only currently supported mechanism for transmitting OAuth
  protocol parameters is the HTTP Authorization header.

Realm

  The only currently supported value for the *realm* parameter is
  **all**, which will grant full API access.


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


