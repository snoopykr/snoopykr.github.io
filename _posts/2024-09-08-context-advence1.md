---
layout: single
title: "Golang Cancel Context with Interupt"
categories : Golang
tag: [Golang, Context]
---

Golang Cancel Context with Interupt

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

func main() {
	// Ctrl-C : 인터럽트 발생을 감지.
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	ctx, cancel := context.WithCancel(context.Background())

	// 인터럽트 감지후 Cancel처리.
	go func() {
		fmt.Println("Signal:", <-exit)
		cancel()
	}()

	start := time.Now()
	result, err := longFuncWithCtx(ctx)
	if err != nil {
		// 예외처리가 발생된 경우 로그 출력...
		log.Fatal(err)
	}

	// 정상적인 종료가 되면 아래 부분이 실행.
	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)
}

func longFuncWithCtx(ctx context.Context) (string, error) {
	done := make(chan string)

	// 5초후 종료 체크
	go func() {
		done <- longFunc()
	}()

	select {
	// 예외사항 처리...
	case <-ctx.Done():
		fmt.Println("예외사항 발생...!!!")
		return "Fail", ctx.Err()
	// 정상 종료...
	case result := <-done:
		fmt.Println("정상 종료...!!!")
		return result, nil
	}
}

func longFunc() string {
	// 이렇게 사용해도 문제가 없다.
	//time.Sleep(time.Second * 5)

	// time.Sleep을 사용해도 되지만 채널을 통해 전달한다는 의미로 사용 가능할 것 같다.
	<-time.After(time.Second * 5)
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

	// struct를 사용하는 채널...???
	quit := make(chan struct{})

	// 인터럽트 발생...
	go func() {
		fmt.Println("Signal:", <-exit)
		// 빈 struct 전달...
		quit <- struct{}{}
	}()

	start := time.Now()
	result, err := longFuncWithoutCtx(quit)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("duration:%v result:%s\n", time.Since(start), result)
}

func longFuncWithoutCtx(quit <-chan struct{}) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-quit:
		return "Fail", errors.New("Force quit")
	case result := <-done:
		return result, nil
	}
}

func longFunc() string {
	<-time.After(time.Second * 3)
	return "Success"
}
```

### Context 사용 Multi Job

```golang
package main

import (
	"context"
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

const jobCount = 10

func main() {
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	ctx, cancel := context.WithCancel(context.Background())

	go func() {
		fmt.Println("Signal:", <-exit)
		cancel()
	}()

	start := time.Now()

	// Multi Job을 위해 WaitGroup 싱셩.
	var wg sync.WaitGroup
	for i := 0; i < jobCount; i++ {
		wg.Add(1)

		go func() {
			defer wg.Done()
			result, err := longFuncWithCtx(ctx)
			fmt.Printf("duration:%v result:%s\n", time.Since(start), result)
			if err != nil {
				// log을 사용하면 1건의 출력만 발생된다.
				// log.Fatal(err)
				fmt.Println(err)
			}
		}()
	}

	wg.Wait()

}

func longFuncWithCtx(ctx context.Context) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-ctx.Done():
		return "Fail", ctx.Err()
	case result := <-done:
		return result, nil
	}
}

func longFunc() string {
	<-time.After(time.Second * 5)
	return "Success"
}
```

### 채널 사용 Multi Job (context X)

```golang
package main

import (
	"errors"
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

const jobCount = 10

func main() {
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

	// 채널 사이즈가 10...!!!
	quits := make(chan struct{}, jobCount)

	go func() {
		fmt.Println("Signal:", <-exit)

		// 일부 채널(10 미만)에 인터럽트를 발생시켜 정지가 가능하다.
		//for i := 0; i < 5; i++ {
		//	quits <- struct{}{}
		//}

		for i := 0; i < jobCount; i++ {
			quits <- struct{}{}
		}
	}()

	start := time.Now()

	var wg sync.WaitGroup
	for i := 0; i < jobCount; i++ {
		wg.Add(1)

		// quits가 전달되며 quit <-chan struct{}를 통해 활당이 되는 것 같다.
		go func(quit <-chan struct{}) {
			defer wg.Done()
			result, err := longFuncWithoutCtx(quit)
			fmt.Printf("duration:%v result:%s\n", time.Since(start), result)
			if err != nil {
				fmt.Println(err)
			}
		}(quits)
	}

	wg.Wait()
}

func longFuncWithoutCtx(quit <-chan struct{}) (string, error) {
	done := make(chan string)

	go func() {
		done <- longFunc()
	}()

	select {
	case <-quit:
		return "Fail", errors.New("Force quit")
	case result := <-done:
		return result, nil
	}
}

func longFunc() string {
	<-time.After(time.Second * 5)
	return "Success"
}
```
