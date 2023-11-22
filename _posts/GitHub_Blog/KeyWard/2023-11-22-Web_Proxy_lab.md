---
title: "Week06_Proxy_Lab"
last_modified_at: "2023-11-22T21:30:00"
categories:
  - 크래프톤 정글
  - 네트워크
tags:
  - 크래프톤 정글
  - Proxy lab
---

## Proxy Lab
  개인적으로 정말 어려웠던 것 같다<br>
  ~~(당장 다음주부터 핀토스 시작이지만)~~<br>

  일단 proxy의 개념인<br>
  클라 -> 프록시 -> 서버(tiny) <br>
  클라 <- 프록시 <- 서버(tiny) <br>
  를 이해하는 것이 생각보다 까다로웠다<br>

  프록시는 보안 강화(포워드 : 클라 , 리버스 : 서버 의 IP를 숨긴다, 또한 특정 IP에 대한 차단이 가능)<br>
  로드 밸런싱(요청을 관리하여 백 서버의 부하를 줄임)<br>
  캐싱(이미 요청된 동일한 요청을 캐싱에서 반환하여 백 서버의 부하를 줄이고, 응답 시간 향상)<br>
  의 역할을 한다<br>
  (다만 이번에 구현한 것이 보안 강화와 로드 밸런싱 역할을 하는지는 잘 모르겠다)<br>

  다만 프록시의 역할 중,<br>
  '캐시'에 해당하는 부분은 시간 상 구현하지 못하였다<br>

  concurrency 테스트 를 진행하느라<br>
  CSAPP 12 장의 동시성 프로그래밍 부분을 참고하였던 부분은 아주 흥미로웠다<br>

  간략하게 정리하자면,<br>
  쓰레드(Thread)는 '프로세스'의 실행 단위이며<br>
  프로세스의 자원을 공유하며 독립적으로 실행이 된다<br>
  (쓰레드는 프로세스의 실행 흐름을 담당하며, 일반적으로 커널에서 관리된다)<br>
  
  세마포어(Semaphore)는 상호 배제와 동기화를 위한 도구 중 하나이다<br>
  프로세스 및 스레드의 '공유 자원'에 대한 접근을 제어 및 조정 하는데 사용된다<br>

  P 연산 으로 세마포어 값을 감소시키고, 이 떄 값이 0(혹은 음수)가 되면 대기 상태로 만든다<br>
  V 연산 으로 세마포어 값을 증가시키고, P로 인해 대기가 된 스레드 중 하나를 실행 상태로 만든다<br>

  세마포어는 '임계 영역(Critical Section)'을 보호하며,<br>
  경쟁 상태(Race Condition) 및 데드락(Dead Lock)을 예방할 수 있다<br>
  (경쟁 상태 : 둘 이상의 프로세스 or 스레드가 공유 자원에 접근하여 예측이 불가능한 상황을 초래)<br>
  (데드락 : 둘 이상의 프로세스가 서로의 자원을 필요로 하여 무한정 대기하는 상태)<br>

  뮤텍스(Mutex)는 세마포어와 마찬가지로 상호 배제와 동기화를 위한 도구 중 하나이다<br>
  주로 공유 자원에 대한 '동시 접근'을 막기 위해 사용하는 개념으로<br>
  종종 '이진 세마포어(Binary Semaphore)'로 구현되기도 한다<br>

  스레드는 임계영역에 들어갈 때 락(lock)을 통하여,<br>
  다른 스레드가 접근하지 못하게 한다<br>
  이후, 임계영역에서 작업을 완료할 동안, 다른 스레드는 접근하지 못하기에<br>
  해당 변수 등의 변화는 현재 스레드만이 건들게 되므로<br>
  '동기화'가 된다고 해석할 수 있다<br>
  이후 임계영역에서 벗어날 때, 락을 해제함으로서<br>
  다른 스레드가 접근할 수 있도록 한다<br>

  github : <https://github.com/hnjog/webproxy-lab>

## Code (proxy.c)
```
#include <stdio.h>
#include "csapp.h"
#include "sbuf.h"

/* Recommended max cache and object sizes */
#define MAX_CACHE_SIZE 1049000
#define MAX_OBJECT_SIZE 102400

#define NTHEADS 4
#define SBUFSIZE 16

/* You won't lose style points for including this long line in your code */
static const char *user_agent_hdr =
    "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 "
    "Firefox/10.0.3\r\n";

void doit(int fd);
void parse_uri(char *uri, char *request_ip, char *port, char *filename);
void serve_Proxy(int servefd, int clientfd);
void clienterror(int fd, char *cause, char *errnum, char *shortmsg,
                 char *longmsg);

void make_header(char *method, char *request_ip, char *user_agent_hdr, char *version, int requested_fd, char *filename);

void* threadWork(void *vargs);

sbuf_t sbuf;

// 일단 프록시 서버는 클라에서 요청하는 정적인 컨텐츠를 처내는 용도임을 잊지 말자
// 일단 tiny에서 사용한 녀석들 중
// static 녀석들만 가져와서 작동시켜 보자
// 그리고 tiny로 보내는 방식을 취하는듯 함

// 요청을 서버로 보낸다(tiny)

void doit(int fd)
{
  int requested_fd;
  char buf[MAXLINE], method[MAXLINE], uri[MAXLINE], version[MAXLINE];
  char request_ip[MAXLINE], filename[MAXLINE], port[MAXLINE];
  rio_t rio;

  // 요청 라인과 헤더 읽기
  Rio_readinitb(&rio, fd);
  Rio_readlineb(&rio, buf, MAXLINE);
  printf("Request headers:\n");
  printf("%s", buf);
  sscanf(buf, "%s %s %s", method, uri, version); // buf 문자열에서 method, uri, version 읽기

  if (strcmp(method, "GET") && strcmp(method, "HEAD"))
  {
    clienterror(fd, method, "501", "Not implemented", "Tiny does not implement this method");
    return;
  }

  // uri를 나누어
  // 요청 ip , port, filename으로 나누다
  // 내부에서 return
  parse_uri(uri, request_ip, port, filename);

  printf("request IP : %s\r\n", request_ip);
  printf("Port : %s\r\n", port);
  printf("filename : %s\r\n", filename);
  requested_fd = Open_clientfd(request_ip, port);

  // 서버로 전송함
  make_header(method, request_ip, user_agent_hdr, version, requested_fd, filename);

  // 어차피 서버에서
  serve_Proxy(requested_fd, fd);
  Close(requested_fd);
}

// uri 분할 역할의 함수
void parse_uri(char *uri, char *request_ip, char *port, char *filename)
{
  // http:// 가 앞에 붙어서 오네??
  char tempUri[MAXBUF] = {
      0,
  };
  char *ptr = strchr(uri, '/');
  size_t templen = strlen(ptr + 2);
  strncpy(tempUri, ptr + 2, templen);
  ptr = strchr(tempUri, ':');

  // port 존재 (:이 있다)
  if (ptr != NULL)
  {
    *ptr = '\0';                 // 기준점에 null 문자 넣어주고
    strcpy(request_ip, tempUri); // 전반부는 요청 IP에
    strcpy(port, ptr + 1);       // 이후는 port번호로 넣어준다

    ptr = strchr(port, '/');
    // port 번호 이후 요청하는 파일 이름이 있다면
    if (ptr != NULL)
    {
      // filename에 넣어주고
      strcpy(filename, ptr);
      // null문자로 바꿔준다
      *ptr = '\0';
      // port가 배열이기에
      // null문자가 있는 부분까지만 유지하고
      // 나머지는 버리기 위함
      strcpy(port, port);
    }
  }
  else // 포트번호가 없는 요청이다
  {
    // uri 를 요청 IP 취급한다
    strcpy(request_ip, tempUri);

    ptr = strchr(request_ip, '/');
    if (ptr != NULL)
    {
      strcpy(filename, ptr + 1);
    }
    port = "80";
  }
}

//
void make_header(char *method, char *request_ip, char *user_agent_hdr, char *version, int requested_fd, char *filename)
{
  char buf[MAXLINE];

  sprintf(buf, "%s %s %s\r\n", method, filename, "HTTP/1.0");
  sprintf(buf, "%sHost: %s\r\n", buf, request_ip); // 목적지 IP입력
  sprintf(buf, "%s%s", buf, user_agent_hdr);
  sprintf(buf, "%sConnection: %s\r\n", buf, "close");
  sprintf(buf, "%sProxy-Connection: %s\r\n\r\n", buf, "close");

  Rio_writen(requested_fd, buf, strlen(buf));
}

void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg)
{
  char buf[MAXLINE], body[MAXBUF];

  // HTTP response body
  sprintf(body, "<html><title>Proxy Error</title>");
  sprintf(body, "%s<body bgcolor = "
                "ffffff"
                ">\r\n",
          body);
  sprintf(body, "%s%s: %s\r\n", body, errnum, shortmsg);
  sprintf(body, "%s<p>%s: %s\r\n", body, longmsg, cause);
  sprintf(body, "%s<hr><em> The Proxy Web server</em><html>\r\n", body);

  // HTTP response header
  sprintf(buf, "HTTP/1.0 %s %s \r\n", errnum, shortmsg);
  Rio_writen(fd, buf, strlen(buf));
  sprintf(buf, "Content-type: text/html\r\n");
  Rio_writen(fd, buf, strlen(buf));
  sprintf(buf, "Content-length: %d\r\n\r\n", (int)strlen(body));
  Rio_writen(fd, buf, strlen(buf));

  Rio_writen(fd, body, strlen(buf));
}

void serve_Proxy(int servefd, int clientfd)
{
  int src_size;
  char *srcp, *p, content_length[MAXLINE], buf[MAXBUF];
  rio_t server_rio;

  // Rio 에 server 파일 디스크립터를 넣음
  Rio_readinitb(&server_rio, servefd);

  // 첫 번째 라인을 읽음
  Rio_readlineb(&server_rio, buf, MAXLINE);

  // 클라이언트 에 현재 읽은 첫번째 라인 써주기
  Rio_writen(clientfd, buf, strlen(buf));

  // 헤더 처리 구문
  // tiny에서 헤더 마지막에 "\r\n"을 쓰므로
  // 그 전까지 읽는다
  while (strcmp(buf, "\r\n"))
  {
    // 하나의 라인을 읽었는데 그 라인에 길이 관련 내용이 있다
    if (strstr(buf, "Content-length:") != NULL)
    {
      printf("find it!\r\n");
      // 해당 라인의 공백으로 (Content-length:) 의 이후
      p = index(buf, ' ');
      // 이후 부분부터 content_length에 복사
      strcpy(content_length, p + 1);
      // atoi를 통해 src_size를 구한다
      src_size = atoi(content_length);
    }

    // 한줄 한줄 읽어준다
    Rio_readlineb(&server_rio, buf, MAXLINE);

    // 헤더는 한줄 읽을 때마다 보내준다
    Rio_writen(clientfd, buf, strlen(buf));
  }

  // 본문 보내기
  srcp = Malloc(src_size);
  Rio_readnb(&server_rio, srcp, src_size);
  Rio_writen(clientfd, srcp, src_size);
  Free(srcp);
}

int main(int argc, char **argv)
{
  int listenfd, connfd;
  char hostname[MAXLINE], port[MAXLINE];
  socklen_t clientlen;
  struct sockaddr_storage clientaddr;
  pthread_t tid;

  /* Check command line args */
  if (argc != 2)
  {
    fprintf(stderr, "usage: %s <port>\n", argv[0]);
    exit(1);
  }

  listenfd = Open_listenfd(argv[1]);

  // 쓰레드 생성
  sbuf_init(&sbuf,SBUFSIZE);
  for(int i = 0; i < NTHEADS;i++)
  {
    Pthread_create(&tid,NULL,threadWork,NULL);
  }
  
  while (1)
  {
    clientlen = sizeof(clientaddr);
    connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen); // 연결 요청 접수
    Getnameinfo((SA *)&clientaddr, clientlen, hostname, MAXLINE, port, MAXLINE, 0);
    printf("Accepted connection from (%s, %s)\n", hostname, port);
    sbuf_insert(&sbuf,connfd);
  }

  sbuf_deinit(&sbuf);
}

void* threadWork(void *vargs)
{
  Pthread_detach(pthread_self());

  while (1)
  {
    int connfd = sbuf_remove(&sbuf);
    doit(connfd);  // 트랜잭션 수행
    Close(connfd); // 연결 닫기
  }
  
}
```