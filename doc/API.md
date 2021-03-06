# User Management API

This document provides protocol-level details of the User Management API.

___

# HTTP API

This library exposes an HTTP API to register a single user and obtain session tokens for that user.

In the future, the API will evolve to support multiple users.

## URL Structure

All requests will be to URLs of the form:

    https://<host-url>/<api-endpoint>

Note that:

* All API access must be over a properly-validated HTTPS connection.
* The API endpoints may be mounted on a path different than the host root. For instance, for [Project Linked](https://github.com/fxbox/foxbox) we are mounting this API at `/users`, so the URLs end up being `https://<host-url>/users/<api-endpoint>`

## Request Format

All POST requests must have a content-type of `application/json` with a utf8-encoded JSON body.

### Authentication

For the first iteration of this project, requests that require user authentication must contain a header including a signed [JWT](https://jwt.io/). We may want to use [HAWK](https://github.com/hueniverse/hawk) in the future.

Use the JWT with this header:

```js
{
    "Authorization": "Bearer <jwt>"
}
```

## Response Format

All successful requests will produce a response with HTTP status code of "20X" and content-type of "application/json".  The structure of the response body will depend on the endpoint in question.

Failures due to invalid behavior from the client will produce a response with HTTP status code in the "4XX" range and content-type of "application/json".  Failures due to an unexpected situation on the server side will produce a response with HTTP status code in the "5XX" range and content-type of "application/json".

To simplify error handling for the client, the type of error is indicated both by a particular HTTP status code, and by an application-specific error code in the JSON response body.  For example:

```js
{
  "code": 400, // matches the HTTP status code
  "errno": 777, // stable application-level error number
  "error": "Bad Request", // string description of the error type
  "message": "the value of salt is not allowed to be undefined"
}
```

Responses for particular types of error may include additional parameters.

The currently-defined error responses are:
* status code 400, errno 400: Bad request.
* status code 400, errno 100: Invalid user name. Missing or malformed user name.
* status code 400, errno 101: Invalid email. Missing or malformed email.
* status code 400, errno 102: Invalid password. The password should have a minimum of 8 chars.
* status code 400, errno 103: Missing or malformed authentication header.
* status code 401, errno 401: Unauthorized. If credentials are not valid.
* status code 409, errno 409: Conflict. The user is already registered.
* status code 410, errno 410: Gone. The resource is no more available. Don't insist.
* status code 501, errno 501: Internal server error.
* any status code, errno 999: Unknown error

# API Endpoints

* Setup
    * [POST /setup](#post-setup)
    * [POST /login](#post-login)

## POST /setup
Allow to initiate the box by registering an admin user. CORS is not allowed for this endpoint.
### Request
___Parameters___
* email - Admin email.
* username - User name. Defaults to "admin".
* password - Admin password.
```ssh
POST /setup/ HTTP/1.1
Content-Type: application/json
{
  "email": "user@domain.org",
  "username": "Pepe",
  "password": "whatever"
}
```
### Response
Successful requests will produce a "201 Created" response with a session token in the body.
```ssh
HTTP/1.1 201 OK
Connection: close
{
  "session_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEiLCJuYW1lIjoidXNlcm5hbWUifQ.IEMuCIdMp53kiUUoBhrxv1GAPQn2L5cqhxNmCc9f_gc"
}
```

**Once the admin user is created, this route will return a 410 error, Gone.**.

Failing requests may be due to the following errors:
* status code 400, errno 100: Invalid user name. Missing or malformed user name.
* status code 400, errno 101: Invalid email. Missing or malformed email.
* status code 400, errno 102: Invalid password. The password should have a minimum of 8 chars.
* status code 400, errno 400: Bad request.
* status code 409, errno 409: Already exists.
* status code 410, errno 410: Gone. There is already an admin user registered.

## POST /login
Authenticates a user.
### Request
Requests must include a [basic authorization header](https://en.wikipedia.org/wiki/Basic_access_authentication#Client_side) with `username:password` encoded in Base64 according to [RFC2617](http://www.ietf.org/rfc/rfc2617.txt)
```ssh
POST /setup/ HTTP/1.1
Content-Type: application/json
Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l
```
### Response
Successful requests will produce a "201 Created" response with a session token in the form of a [JWT](https://jwt.io/introduction/) with the following data:
```js
{
    "id": 1,
    "name": "username"
}
```
The token is provided in the body of the response:
```ssh
HTTP/1.1 201 OK
Connection: close
{
  "session_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEiLCJuYW1lIjoidXNlcm5hbWUifQ.IEMuCIdMp53kiUUoBhrxv1GAPQn2L5cqhxNmCc9f_gc"
}
```

Failing requests may be due to the following errors:
* status code 400, errno 103: Missing or malformed authentication header.
* status code 400, errno 400: Bad request.
* status code 401, errno 401: Unauthorized. If credentials are not valid.
