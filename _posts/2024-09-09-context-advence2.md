---
layout: single
title: "Golang Timeout Context with Interupt"
categories : Golang
tag: [Golang, Context]
---

Golang Timeout Context with Interupt

### Context 사용

```golang
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"
)

const maxDuration = time.Second * 2

func main() {
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	// 타임아웃 1초를 지정해준다.
	ctx, cancel := context.WithTimeout(context.Background(), maxDuration)

	go func() {
		fmt.Println("Signal:", <-exit)
		cancel()
	}()

	// 결과보다 먼저 타웃아웃이 발생되기 때문에 예외처리를 하면 된다.
	start := time.Now()
	result, err := longFuncWithCtx(ctx)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)

	// 출력
	// 예외사항 발생...!!!
	// 2021/11/29 13:35:09 context deadline exceeded
}

func longFuncWithCtx(ctx context.Context) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-ctx.Done():
		fmt.Println("예외사항 발생...!!!")
		return "Fail", ctx.Err()
	case result := <-done:
		fmt.Println("정상 종료...!!!")

		return result, nil
	}
}
func longFunc() string {
	// 3초후에 Success를 전달하지만 이미 타임아웃이 된 상태가 된다.
	<-time.After(time.Second * 3)
	return "Success"
}
```

### 채널 사용 (context X)

```golang
package main

import (
	"errors"
	"fmt"
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	start := time.Now()
	result, err := longFuncWithoutCtx(exit)
	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)
	if err != nil {
		log.Fatal(err)
	}

}

func longFuncWithoutCtx(quit <-chan os.Signal) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-quit:
		return "Fail", errors.New("Force quit")
	case <-time.After(time.Second):
		return "Fail", errors.New("Timeout")
	case result := <-done:
		return result, nil
	}
}
func longFunc() string {
	<-time.After(time.Second * 3)
	return "Success"
}
```