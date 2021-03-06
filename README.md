# This is a [beanstalkd](https://github.com/kr/beanstalkd) client for Go.
[中文文档] (./README-zh_CN.md)

Now it's a quite simple implementation. I'll keep improving and refactoring it.

# Introduction
[beanstalkd](https://github.com/kr/beanstalkd) is a fast, general-purpose work queue.
Difference with crontab jobs:
Contab job run with specified period or at some point. But beanstalk can run with a delayed time.

Some use scenarios:
* Check whether user finish the order in 5 minutes.
* Start a process in one minutes.

# How to use

## Mac
### Install And Start Beanstalk server
```
git clone https://github.com/kr/beanstalkd
cd beanstalkd
make
./beanstalkd
```

# Examples

```
package main

import (
	"fmt"
	"beanstalk-go/client"
)

func main() {
	addr := "localhost:11300"  // define server address
	newConn := client.NewConnection(addr)   // create new connection
	channel := make(chan int)   // create int channel
	putFunc := func() {       // define a function which put some message to one tube
		id, _ := newConn.PutWithTube("hello", "test2", 1)
		channel <- id
	}
	go putFunc()   // run previous function in a go-routine
	id := <-channel  // wait until we finish putting
	fmt.Printf("Receive from channel message of another goroutine %d\n", id)
	listenChannel := make(chan string)  // make a listen channel for receiving results
	dealFunc := func(body string) bool{  // define a function to deal with tube messages
		fmt.Printf("receive %s\n", body)
		listenChannel <- body
		return true
	}
	go newConn.Listen("test2", dealFunc)  // run deal function in a specified go-routing
	body := <- listenChannel     // wait our message
	fmt.Printf("Listen once %s\n", body)
	newConn.Close()   // Close connection
}

```