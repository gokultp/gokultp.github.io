---
published: true
title: A Journey with Http day 2
layout: post
---
Yesterday we could log the message sent by browser.

```
Listening on 7000
GET / HTTP/1.1
Host: localhost:7000
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us
Connection: keep-alive
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/601.5.17 (KHTML, like Gecko) Version/9.1 Safari/601.5.17


```

This message content is very familiar to me. It looks like a http header.

Lets check it out, open terminal and make a request to google.com

```sh
$curl google.com
```

Ohh its returning an html content.

```html
<HTML>
	<HEAD>
		<meta http-equiv="content-type" content="text/html;charset=utf-8">
		<TITLE>302 Moved</TITLE>
	</HEAD>
	<BODY>
		<H1>302 Moved</H1>
		The document has moved
		<A HREF="http://www.google.co.in/?gfe_rd=cr&amp;ei=c7u3V-3VI-rI8AeVj4GACg">here</A>.
	</BODY>
</HTML>
```

So lets try a header request using curl.

```sh
$curl -I google.com
```

Yeah got a result, but not the same
