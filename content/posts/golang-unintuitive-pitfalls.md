---
title: "Golang's unintuitive pitfalls"
date: 2023-07-28T08:00:23+02:00
description: My personal opinion about what Golang does well and what it does not.
draft: false
tags: [go, programming]
---

## Introduction
Disclaimer: I have just recently picked up the language, so don't expect this post to be a deep analysis of advanced Golang concepts. Instead, think of it as the perception of Golang from someone who already knows other programming languages but just recently started to learn Go. As with most things, there is stuff that I really like about Golang, and there are quite a few things that truly annoy me. Let's do the feedback sandwich thing:

## The positive

Golang to me feels like a nice middle ground between Python and C. You have an okay-ish standard library that can do most things you want to do (Python-esque) and if you feel like it, you can go low-level and work with pointers and be memory-efficient (C-esque). I also like how Goroutines, channels and deferred functions make it relatively simple to coordinate multiple lightweight threads. The syntax feels alright most of the time and I like having the option to make use of automatic type inference or declare the type of something myself. The code seems to port well between different systems, and the required setup is minimal. I like that functions are able to return multiple values of different types; this makes error handling very simple and elegant. To summarize the strengths of Go:

1. Powerful concurrency

2. Intuitive error-handling (this is a bit controversial because there is no exception support and error handling seems to make up half of your program (if err != nil), so I would say it is ugly but intuitive)

3. Cross-platform support and fast compilation

4. Garbage collection that just works

5. Compiles to a single static binary

## The negative

1. I strongly dislike Golang IDEs and plugins for Golang. The root of the problem is that in Golang, unused variables are not a warning but an error. That's why, while writing code, you constantly see red-underlined text, and you get the feeling that you did something fundamentally wrong, but it always turns out that the IDE complains that the variable you have just defined is not being used yet. It gets even worse: Golang IDEs and plugins will ensure that all unused code is automatically deleted when  you save your code! So you can spend twenty minutes writing your custom structs and setting up the workflow of your program, and then you want to save your progress, and the IDE will delete half of your code (without warning!!!). This is not acceptable, IMO. There does not seem to be a way to disable this behavior in commonly used IDEs like Goland or in Visual Studio Code (which BTW, is a completely separate project from Visual Studio; why would Microsoft name it so similar?) with the Go plugin. If I press Ctrl+S I want to save my progress, but the IDE does the opposite and deletes code without my permission. And to my astonishment, there is a large community that supports this behavior, which blows my mind. To work around this I have been using Visual Studio (not "Code") which does not support Golang. This way, I get decent syntax highlighting, and it remembers opened tabs after I close the IDE without having to fight against "smart features".

2. Another thing I do not enjoy about Golang is how packet and dependency management work. It is the opposite of intuitive and forces you to enter commands in a shell that you can't possibly know or guess when you are new to the language. IMO, the least-bad workflow is to have a main.go file in a project folder like, e.g., "myproject". Now you just need to run "go mod init main", followed by "go mod tidy". Now you can make subdirectories with more .go files as you like, and you have a decent project structure. If you want to include one of your local .go files, you just provide the relative path from your root directory, e.g. "myproject/somemodule/test.go". To run the code, just type "go run ." in the root folder of your project (Golang is a compiled language, but it provides a run command that combines the compilation and the execution step while also hiding the resulting binary file from you, which makes it feel like it is an interpreted language). In many online examples I found, people will try to tell you to name your project as if it were a Github repo or a website (????). This is [not a joke](https://stackoverflow.com/questions/55442878/organize-local-code-in-packages-using-go-modules), let me quote the most upvoted answer: "Even for local-only modules, it is a good idea to give the module a globally unique name.. ..as if it will be published someday, even if that repo does not yet exist". Even for non-local, external packages, I find the idea of importing the Git URLS of the package dubious at best. Note: It also is unclear which URL you have to import in your code. Golang expects the Git URL (which is not reachable via Browser), but if you put the "reachable" URL (Github URL that might contain tree/master) it will not work and the error message is not helpful in resolving this problem (useless/unspecific error messages are a common problem in Golang).

3. Another thing that is commonly criticized about Go is that it does not support a "class" keyword. I have not found this to be a problem so far because you have struct, type aliases and interfaces. You can also make constructors for your structs, so I currently do not have the impression that anything important is missing. My opinion about this topic might change over time.

4. What I also dislike about the language is that it dictates a certain style, for example:
```go
if true {
	// do sth
} else {
	// do sth
}
```
forces you to put the else on the same line as the "}" closing "if". You are not allowed to put the "else" on a new line, like you might have done for decades in C. The following code throws an error:
 ```go
if true {
	// do sth
} 
else {
	// do sth
}
```
While I do see the appeal of forcing the same coding style on everyone, it might be a counteractive mechanic if you want to convince people to adapt to Go. Long-formed habits are not easily broken, and this might just be one of the many small annoyances that make you decide to use another language instead. Another example is when you want to initialize an instance of a struct:
 ```go
	transaction1 := transaction.Transaction{
		From:   "0xa2e",
		TxTime: 1690545224,
		To:     "0xe2b",
		Value:  1.1,
	}
```
You are forced to put a comma after the last argument! The reasoning is that when adding a newline, you can't possibly forget to add a comma to the previous line because it was already there. But this forced comma is just another one of these small annoyances that you keep discovering as you learn Go. Stop breaking the habits of old-time programmers.

5. No function overloading. Why do we have to end up with five different functions that essentially do the same thing? Note: A new Golang update now allows for generics. Which does not solve this problem but at least makes it bearable.

6. Inconsistency. You are allowed to initialize a struct with named parameters, but if your struct has a constructor, and you use the constructor to initialize an instance of your struct, you are suddenly not allowed to use named parameters anymore. Another example: If you initialize a struct directly, you must put ',' behind every argument (even the last one). But if your struct has a constructor function, then you are not allowed to put ',' behind the last argument.

## Conclusion

Golang is a popular, relatively new language that especially shines when used for backend stuff like networking or peer-to-peer code. It feels like a decent middle ground between easy-to-learn, slow languages like Python and hard-to-master, fast languages like C. While there are lots of small annoyances that can be frustrating, I still would recommend anyone check out Golang if they want something faster than Python but don't have the time to learn C or C++. If I could choose a single thing about the language, I would make unused code a warning, not an error.