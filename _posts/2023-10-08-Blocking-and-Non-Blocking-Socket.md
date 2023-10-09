---
title: Blocking and Non-Blocking Socket
author:
  name: Dennis
categories: [C++, Network]
tags: [C++, Network]
pin: true
use_math: true
---

I was recently studying Overlapped I/O and I came across the concepts of blocking and non-blocking socket. I have already known the concepts of blocking socket but I didn’t know for non-blocking socket. So, today I gonna talk about difference between blocking and non-blocking socket.
<br>

<h2>What is Blocking and Non-Blocking?</h2>
We usually call the function to process our specific request. At that time, the function processes the request and returns the answer that we want to. What happens if the function takes a long time to process?  You will have to wait for the function to return and feel like the flow has stopped. In this way, when the processing time of a function is long and it feels like the flow has stopped, it is called blocking. Otherwise, If you don’t need to wait for function to return,  It is called non-blocking.
<br><br>


<h2>Why a Socket is Blocked?</h2>

![blockin_io_model]({{site.url}}/assets/img/2023-10-08-Blocking-and-Non-Blocking-Socket/blockin_io_model.png){: width="600" height="600"){: .center}

<i>http://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html</i>

When you call the recvfrom function, control is transferred to the system. if there is no data to read, the recvfrom function don’t return and you need to wait for the function to return. At this time, the thread of application becomes Waiting. When the system receives the data from remote computer, the system copies received data from receive buffer to the memory of application (from kernel space to user space) the control is transferred from the system to application and thread becomes Running.

<h2> The Problem of Blocking I/O Medel </h2>
The socket handle that is made when you create socket is blocking mode basically.

```cpp

SOCKET socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

```
The server sends the packet received back to the client and outputs the size of packet received every second. But there is a problem in this code.

```cpp
while (1) {
if (clientSocket != INVALID_SOCKET)
{
	char messageBuffer[MAX_BUFFER];
	int receiveBytes = recv(clientSocket, messageBuffer, MAX_BUFFER, 0);
	if (receiveBytes > 0)
	{
		printf("TRACE - Receive message : %s (%d bytes)\n", messageBuffer, receiveBytes);

		int sendBytes = 0;
		do
		{
			sendBytes = send(clientSocket, messageBuffer, static_cast<int>(strlen(messageBuffer)), 0);
			if (sendBytes > 0)
			{
				printf("TRACE - Send message : %s (%d bytes)\n", messageBuffer, sendBytes);
			}

			if (SOCKET_ERROR == sendBytes)
			{
				int error = WSAGetLastError();
				if (error == WSAEWOULDBLOCK)
				{
					printf("TRACE - Recv error WSAEWOULDBLOCK(%d)\n", error);
					Sleep(1); //wait for 1 Milliseconds.
				}
				else
				{
					break;
				}
			}
		} while (0 < sendBytes);

		receiveBytesPerSec += receiveBytes;
	}

	if (timeGetTime() - startTime >= 1000)
	{
		printf("TRACE - receiveBytes: %d bytes)\n", receiveBytesPerSec);
		startTime = timeGetTime();
		receiveBytesPerSec = 0;
	}
}
}
```
If the client does not send the packet, The server waits for it, which means that the server is blocked in the recv() function. The recv() does not return until the packet is received  because the socket is in blocking mode. So the server can’t output the size of packet received every second.
<br><br>

<h2> Non-Blocking Socket </h2>
Making non-blocking mode socket is easy. You can change a socket from blocking mode to non-blocking mode using the ioctlsocket() function.
```cpp
//https://learn.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-ioctlsocket
//-------------------------
// Set the socket I/O mode: In this case FIONBIO
// enables or disables the blocking mode for the 
// socket based on the numerical value of iMode.
// If arg = 0, blocking is enabled; 
// If arg != 0, non-blocking mode is enabled.
u_long arg = 1;
ioctlsocket(clientSocket, FIONBIO, &arg);
```
The code above means to change the blocking mode of client socket to non-blocking mode. Now If there are no packets in the socket’s receive buffer, recv() will returns SOCKET_ERROR(-1). The WSAGetLastError() function returns WSAEWOULDBLOCK.

```cpp
while (1) {
if (clientSocket != INVALID_SOCKET)
{
	char messageBuffer[MAX_BUFFER];
	int receiveBytes = recv(clientSocket, messageBuffer, MAX_BUFFER, 0);
	if (receiveBytes > 0)
	{
		//sending packet
		receiveBytesPerSec += receiveBytes;
	}

	if (SOCKET_ERROR == receiveBytes)
	{
		int error = WSAGetLastError();
		if (error == WSAEWOULDBLOCK)
		{
			printf("TRACE - Recv error WSAEWOULDBLOCK(%d)\n", error);
		}
	}

	if (timeGetTime() - startTime >= 1000)
	{
		printf("TRACE - receiveBytes: %d bytes)\n", receiveBytesPerSec);
		startTime = timeGetTime();
		receiveBytesPerSec = 0;
	}
}
}
```
Because the socket is in non- blocking mode, it can output the size of packets received normally. But there is still a problem. The recv() function always returns, whether there are packets in the receive buffer or not, so you need to periodically call the recv() function. In the code above, the recv() function is continuously being called within a while loop, which can result in a highly busy state for the thread. If you reduce the frequency of recv() function calls, the response sent to the client will slow down, whereas increasing the call frequency will result in the thread being in a highly busy state. This problem can be resolved with iocp.
<br><br>

<h2> Non-Blocking Socket(Sending packet) </h2>
When does the send() function become blocking? To understand this, you need to know first how the send() function is processed internally when sending data with block mode socket.

![An_established_TCP_connection]({{site.url}}/assets/img/2023-10-08-Blocking-and-Non-Blocking-Socket/An-established-TCP-connection.png){: width="500" height="500"){: .center}

<i>https://www.researchgate.net/figure/An-established-TCP-connection_fig1_220926307</i>

A kernel socket has two buffers: the first one is a send buffer to send packet and the second one is a receive buffer to receive packet. When the send() is called, the data you want to send is copied from user space to kernel space then it is appended to the end of send buffer to send data in order.

In most cases, when you call the send() function, it returns promptly. However, if there is network latency or the remote host fails to process packets, the packets in the send buffer may not be sent and the send buffer to become full. In such cases, the send() function enters a blocking state until space becomes available in the buffer.

```cpp
//sending packet
int sendBytes = 0;
do
{
	sendBytes = send(clientSocket, messageBuffer, static_cast<int>(strlen(messageBuffer)), 0);
	if (sendBytes > 0)
	{
		printf("TRACE - Send message : %s (%d bytes)\n", messageBuffer, sendBytes);
	}

	if (SOCKET_ERROR == sendBytes)
	{
		int error = WSAGetLastError();
		if (error == WSAEWOULDBLOCK)
		{
			printf("TRACE - Recv error WSAEWOULDBLOCK(%d)\n", error);
			Sleep(1); //wait for 1 Milliseconds.
		}
		else
		{
			break;
		}
	}
} while (0 < sendBytes);
```
In the code above, the clientSocket is set to non-blocking mode. As a result, the send() function returns promptly, whether the send buffer is full or not. Typically, the send buffer returns the size of the packet to be sent, but if the send buffer is full, it will return SOCKET_ERROR(-1), and the WSAGetLastError() function will return WSAEWOULDBLOCK. In this code, when the send buffer is full, it waits for 1 second and then attempts to send again. When using a non-blocking mode socket like this,  the logic to detect sending or receiving data failure and try it again.
<br><br>

<h2> Conclusion </h2>
Today, I talked about the concept of blocking and non-blocking socket. Infact,  this post is intended to understand overlapped I/O which will be posted in the near future. I hope this article will help you.
<br><br>

<h2> References </h2>
[블록킹,논블록킹,동기 I/O,비동기 I/O(Overlapped I/O)](https://marmelo12.tistory.com/287)