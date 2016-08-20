---
published: true
title: A Journey with Http
layout: post
---

I was in a journey with http for last few days,  was started from one of my old programs which was written in my college days.
When I have just gone through my old github repositories, one of those programs was came to my attention, especially a couple of lines.

```c
if(listen(welcomeSocket,5)==0)
   printf("Listening on 7000\n");

```
It was a tcp server. I have'nt written any socket programs in C since my college days. But still it looks like very familiar to me. I could recollect where I have used the same kind of code, within a fraction of second. It may be familiar for you,  if you are a web developer.

If you can't recollect where it have been used, take a look at these code snippets.

```golang
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```
and one more

```js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

These two are simple http servers. The first one is written in golang and the second is written in node.js. So why that C code snippet caught my attention ? Its because of that function name 'Listen'.

I was aware of that http is an application layer protocol running over tcp. Then I think, our tcp server can communicate with a web browser. I decided to try it out, and my  goal is to send a message to web browser.

```c
/****************** SERVER CODE ****************/

#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(){
  int welcomeSocket, newSocket;
  char buffer[1024];
  struct sockaddr_in serverAddr;
  struct sockaddr_storage serverStorage;
  socklen_t addr_size;

  /*---- Create the socket. The three arguments are: ----*/
  /* 1) Internet domain 2) Stream socket 3) Default protocol (TCP in this case) */
  welcomeSocket = socket(PF_INET, SOCK_STREAM, 0);

  /*---- Configure settings of the server address struct ----*/
  /* Address family = Internet */
  serverAddr.sin_family = AF_INET;
  /* Set port number, using htons function to use proper byte order */
  serverAddr.sin_port = htons(7000);
  /* Set IP address to localhost */
  serverAddr.sin_addr.s_addr = inet_addr("127.0.0.1");
  /* Set all bits of the padding field to 0 */
  memset(serverAddr.sin_zero, '\0', sizeof serverAddr.sin_zero);

  /*---- Bind the address struct to the socket ----*/
  bind(welcomeSocket, (struct sockaddr *) &serverAddr, sizeof(serverAddr));

  /*---- Listen on the socket, with 5 max connection requests queued ----*/
  if(listen(welcomeSocket,5)==0)
    printf("Listening on 7000\n");
  else
    printf("Error\n");


  /*---- Accept call creates a new socket for the incoming connection ----*/
  addr_size = sizeof serverStorage;
  newSocket = accept(welcomeSocket, (struct sockaddr *) &serverStorage, &addr_size);

  /*---- Send message to the socket of the incoming connection ----*/
  buffer = "Hello World"

  send(newSocket,buffer,strlen(buffer),0);


  close(newSocket);  
  close(welcomeSocket);

  return 0;
}


```


This was our tcp server code. I had run it and tried to connect with my browser, by providing url as localhost:7000.
But unfortunately nothing was happened.
I have tried it again for few more times but still, there was no result :(.  

One thing I have noticed is our server was stopping  while we are connecting to browser. So something is happening there. I have tried to debug the code, but I could'nt find any errors,then I have tried to connect  a tcp client to the server and It was working perfectly.

So why it is not connecting with my browser ?

Is there anything wrong with my assumption ?

Or is it problem with my browser ?

I have lot of questions in my mind. I have decided to simply analyse how a web browser communicates with server.


### Step1 : browser makes a requests to server.

![Image of Yaktocat](public/images.jpg)

### Step2 : server will respond to the request.

Ohhhh! yeah I could find out the bullshit!!!!

Actually what is happening here, the web browser is sending a request. So first our server needs to be receive it,  here the server is not trying to receive anything but trying to send some data. So thats the problem !!

see the code

```c
    addr_size = sizeof serverStorage;
    newSocket = accept(welcomeSocket, (struct sockaddr *) &serverStorage, &addr_size);

    /*---- Send message to the socket of the incoming connection ----*/
    buffer = "Hello World"

    send(newSocket,buffer,strlen(buffer),0);


    close(newSocket);  
    close(welcomeSocket);

```

So we have to rewrite it as receive the request first then send the response.

So Let's try.

I have introduced a new buffer for sending response.

```c
char sendBuff[2048] ="";
```

then I had made a few more more changes in our server.

```c
    /*---- Accept call creates a new socket for the incoming connection ----*/
    addr_size = sizeof serverStorage;
    newSocket = accept(welcomeSocket, (struct sockaddr *) &serverStorage, &addr_size);

    /*---- Send message to the socket of the incoming connection ----*/
    recv(newSocket, buffer, 1024, 0);
    strcat(sendBuff,"Hello World");

    send(newSocket,sendBuff,strlen(sendBuff),0);

    close(newSocket);
    close(welcomeSocket);

```

Will it works ? I have checked, unfortunately it was also not working.

So what is the problem ?

Got an Idea !!!!!!

Lets check what the browser is sending to server, lets log that buffer content

so I made a small change in our code.

```c
    /*---- Accept call creates a new socket for the incoming connection ----*/
    addr_size = sizeof serverStorage;
    newSocket = accept(welcomeSocket, (struct sockaddr *) &serverStorage, &addr_size);

    /*---- Send message to the socket of the incoming connection ----*/
    recv(newSocket, buffer, 1024, 0);
    // print the request
    printf("%s\n", buffer);

    strcat(sendBuff,"Hello World");

    send(newSocket,sendBuff,strlen(sendBuff),0);

    close(newSocket);
    close(welcomeSocket);

```

Wow!!!!! Its printing something.

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

we can analyse it tomorrow.
