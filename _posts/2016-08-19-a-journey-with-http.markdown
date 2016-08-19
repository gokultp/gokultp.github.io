---
published: true
title: A Journey with Http
layout: post
---
```c
/****************** SERVER CODE ****************/

#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>
#include <time.h>

char *timestamp()
{
    time_t ltime; /* calendar time */
    ltime=time(NULL); /* get current cal time */
    return asctime( localtime(&ltime) );
}

int main(){
  int welcomeSocket, newSocket;
  char buffer[1024];
  char sendBuff[2048] ="";
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
  serverAddr.sin_port = htons(7896);
  /* Set IP address to localhost */
  serverAddr.sin_addr.s_addr = inet_addr("127.0.0.1");
  /* Set all bits of the padding field to 0 */
  memset(serverAddr.sin_zero, '\0', sizeof serverAddr.sin_zero);

  /*---- Bind the address struct to the socket ----*/
  bind(welcomeSocket, (struct sockaddr *) &serverAddr, sizeof(serverAddr));

  /*---- Listen on the socket, with 5 max connection requests queued ----*/
  if(listen(welcomeSocket,5)==0)
    printf("Listening\n");
  else
    printf("Error\n");

  while(1){
      /*---- Accept call creates a new socket for the incoming connection ----*/
      addr_size = sizeof serverStorage;
      newSocket = accept(welcomeSocket, (struct sockaddr *) &serverStorage, &addr_size);

      /*---- Send message to the socket of the incoming connection ----*/
      recv(newSocket, buffer, 1024, 0);
      printf("Data received: %s",buffer);

      strcat(sendBuff,"HTTP/1.1 200 OK\r\n");
      strcat(sendBuff,"Date: Thu, 19 Feb 2009 12:27:04 GMT");

      strcat(sendBuff,"\r\nConnection: close");
      strcat(sendBuff,"\r\nServer: Gokul/1.0.0");
      strcat(sendBuff,"\r\nAccept-Ranges: bytes");
      strcat(sendBuff,"\r\nContent-Type: text/html");
      strcat(sendBuff,"\r\nContent-Length: 14\r\n");
      strcat(sendBuff,"\r\n<h1>Gokul</h1>\r\n");

      send(newSocket,sendBuff,strlen(sendBuff),0);
  }

  close(newSocket);










  return 0;
}
```
