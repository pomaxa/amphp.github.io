---
title: Stop streaming as soon as possible
description: Aerys is a non-blocking HTTP/1.1 and HTTP/2 application / websocket / static file server.
title_menu: Stream Promises
layout: tutorial
---

```php
(new Aerys\Host)->use(function(Aerys\Request $req, Aerys\Response $res) {
	$handle = \Amp\file\open("largefile.txt", "r");
	while (!$handle->eof()) {
		$chunk = yield $handle->read(8192);
		yield $response->stream($chunk); # it will just abort here, when client disconnects
	}
});
```

`Response::stream()` and `Websocket\Endpoint::send()` return a `Promise` which is fulfilled at the first moment where the buffers aren't full. That `Promise` may also fail with a `ClientException` if the clients write stream was closed.

This allows avoiding spending too much processing time when fetching and returning large data incrementally as well as having too big buffers.

Thus, this isn't relevant for most handlers, except the ones possibly generating very much data (on the order of more than a few hundred kilobytes - the lowest size the buffer typically is at least 64 KiB).