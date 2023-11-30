---
title: epoll source code
description: epoll poll select
categories:
 - Test
tags: 
---

# Introduction

# Difference among the epoll/poll/select

# Main method: epoll_create()、epoll_ctl()、epoll_wait()
1. int epoll_create(int size);
create eventpoll, initialization
2. int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
copy epoll_event into kernel space
3. int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
epoll_wait()
