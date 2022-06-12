---

title: go实现UNIX command
tags: [go, unix]
copyright: true
date: 2018-08-22 19:27:41
permalink:
categories: golang
description: 使用go实现UNIX环境下的命令行工具
image:
---
<p class="description"></p>





```go
package main

import (
	"bufio"
	"errors"
	"fmt"
	"os"
	"os/exec"
	"strings"
)

func main() {
	reader := bufio.NewReader(os.Stdin)
	for {
		fmt.Print("> ")
		// 读取键盘的输入.
		input, err := reader.ReadString('\n')
		if err != nil {
			fmt.Fprintln(os.Stderr, err)
		}

		// 执行并解析command.
		err = execInput(input)
		if err != nil {
			fmt.Fprintln(os.Stderr, err)
		}
	}
}

// 如果cd命令没有路径的话，就报下面的错误
var ErrNoPath = errors.New("path required")

func execInput(input string) error {
	// 移除换行符.
	input = strings.TrimSuffix(input, "\n")

	// 将输入分割成参数.
	args := strings.Split(input, " ")

	// 对cd命令的情况进行区分.
	switch args[0] {
	case "cd":
		// 暂时不支持cd加空格进入home目录.
		if len(args) < 2 {
			return ErrNoPath
		}
		err := os.Chdir(args[1])
		if err != nil {
			return err
		}
		// Stop further processing.
		return nil
	case "exit":
		os.Exit(0)
	}

	// Prepare the command to execute.
	cmd := exec.Command(args[0], args[1:]...)

	// Set the correct output device.
	cmd.Stderr = os.Stderr
	cmd.Stdout = os.Stdout

	// Execute the command and save it's output.
	err := cmd.Run()
	if err != nil {
		return err
	}
	return nil
}
```



```
//执行并测试
go run main.go
```



暂时不支持tab键自动补全命令，只是提供一种简单的思路。

<hr />
