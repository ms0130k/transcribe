# XMLHttpRequest

`XMLHttpRequest` (XHR) objects are used to interact with servers.
You can retrieve data from a URL without having to do a full page refresh.
This enables a Web page to update just part of a page without disrupting what the user is doing.

Despite its name, `XMLHttpRequest` can be used to retrieve any type of data, not just XML.

If you communication needs to invlove receiving event data or message data from a server, consider using server-sent-events through the EventSource interface.
For full-duplex communication, WebSockets may be a better choice.

## Constructor

`XMLHttpRequest()`

The constructor initializes an `XMLHttpRequest`.
It must be called before any other method calls.

## Instance properties

This interface also inherits properties of `XMLHttpRequestEventTarget` and of `EventTarget`.

`XMLHttpRequest.readyState` Read only

Returns a number representing the state of the request.

`XMLHttpRequest.response` Read only

Returns an `ArrayBuffer`, a `Blob`, a `Document`, a JavaScript object, or a string, depending on the value of `XMLHttpRequest.responseType`, that contains the response entity body.

`XMLHttpRequest.responseText` Read only

Returns a string that contains the response to the request as text, or `null` if the request was unsuccessful or has not yet been sent.

`XMLHttpRequest.responseType` Read only

Specifies the type of the response.

`XMLHttpRequest.responseURL` Read only

Returns the serialized URL of the response or the empty string if the URL is null.

`XMLHttpRequest.rexponseXML` Read only

Returns a `Document` containing the response to the request, or `null` if the request was unsuccessful, has not yet been sent, or cannot be parsed as XML or HTML.
Not available in `Web Workers`.

`XMLHttpRequest.status` Read only
Returns the HTTP response status code of the request.

`XMLHttpRequest.statusText` Read only

Returns a string containing the response string returned by the HTTP server.
Unlike `XMLHttpRequest.status`, this includes the entire text of the response message (`"OK"`, for example).

> Note: According to the HTTP/2 specification RFC 7540, section 8.1.2.4:
> Response Pseudo-Header Fields, HTTP/2 does define a way to carry the version or reason phrase that is included in an HTTP/1.1 status line.

`XMLHttpRequest.timeout`

The time in milliseconds a request can take before automatically being terminated.

`XMLHttpRequest.upload` Read only

A `XMLHttpRequestUpload` representing the upload process.

