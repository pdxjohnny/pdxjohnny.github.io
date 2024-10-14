+++
date = 2020-07-24T06:00:00Z
lastmod = 2020-07-24T06:00:00Z
title = "CORS"
subtitle = "Cross Origin Resource Sharing. aka why aren't my ajax requests working?"
+++

Hey you know JavaScript, that thing everyone loves and hates.

Well chances are you've written some of it. And you don't understand what these
bullshit CORS messages are in the console when you try to make a request to a
service hosted on a different domain or port than the one your webpage is served
from.

CORS (Cross Origin Resource Sharing) is a security feature which makes it so
that websites can't send request to other websites without the permission of the
site the request is being sent to. Makes sense, but it can be a pain when
developing.

You need control over the web server you are *requesting* in order to make CORS
requests work (or a proxy between your client and the server).

## Fetch

You might have run across this if you've tried using the
["Using Fetch"](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
example from the Mozilla JavaScript documentation. Since it's about CORS.

```javascript
fetch(url, {
  method: 'POST',
  mode: 'cors',
  cache: 'no-cache',
  credentials: 'same-origin',
  headers: {
    // Triggers the sending of `Access-Control-Request-Headers: content-type`
    'Content-Type': 'application/json',
  },
  redirect: 'follow',
  referrer: 'no-referrer',
  body: JSON.stringify(data), // body data type must match "Content-Type" header
})
```

## Example Request

When your JavaScript makes a `POST` request to `/url/path/requested/by/client`,
the following is the interaction between the server and the client (for the
*preflight* request, in the successful case).

Since we've issued a `POST` request instead of a `GET` request, the browser is
going to make a request before the `POST` called the *preflight* request. This
request has a HTTP method of `OPTIONS`. The purpose is for the browser to check
if this JavaScript is allowed to make a `POST` request to the server.

```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> OPTIONS /url/path/requested/by/client HTTP/1.1
> Host: 127.0.0.1:8080
> Accept-Encoding: deflate, gzip
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
> Accept: */*
> Accept-Language: en-US,en;q=0.5
> Access-Control-Request-Method: POST
> Access-Control-Request-Headers: content-type
> Referer: http://127.0.0.1:5000/no-referrer
> Origin: http://127.0.0.1:5000
> DNT: 1
> Connection: keep-alive
> Pragma: no-cache
> Cache-Control: no-cache
> 
< HTTP/1.1 200 OK
< Access-Control-Allow-Origin: http://127.0.0.1:5000
< Access-Control-Allow-Methods: POST
< Access-Control-Allow-Headers: CONTENT-TYPE
< Content-Length: 0
< Content-Type: application/octet-stream
< Date: Wed, 31 Jul 2019 20:35:24 GMT
< Server: Python/3.7 aiohttp/3.5.4
< 
* Connection #0 to host 127.0.0.1 left intact
```

There's a couple important parts to this.

- `OPTIONS`
  - Since the request is for the `OPTIONS` method. We need to make sure whatever
    server we're using, or library we're using to make a webserver, will respond
    to `OPTIONS` requests on the URL requested.
- `Access-Control-Request-Headers`
  - When the server responds to the `OPTIONS` request, it should include the
    `Access-Control-Allow-Headers` header in the response. This tells the
    browser what headers the JavaScript is allowed to send.

## Fixing it

You likely need to find some middleware for whatever framework your using, or
search for how to enable CORS with whatever your working with.

For example

- https://enable-cors.org/server.html

  - List of how to enable for various servers and clients

- https://github.com/aio-libs/aiohttp-cors

  - If you're using aiohttp server
