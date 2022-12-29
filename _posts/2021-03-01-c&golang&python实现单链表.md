---
layout: post
title: c&golang&python实现单链表
author: Fish-pro
tags:
- python
- c
- golang
date: 2021-03-01 13:56 +0800
---
以下为个人在复习c语言中写的一个demo，然后分别用python和golang又实现了一遍，如有错误之处，请批评指出，谢谢！

## C语言

```c
#include <stdio.h>
#include <stdlib.h>

struct Book
{
	char title[128];
	char author[40];
	struct Book *next;
};

void getInput(struct Book *book)
{
	printf("书名：");
	scanf("%s", book->title);
	printf("作者：");
	scanf("%s", book->author);
}

void addBook(struct Book **library)
{
	struct Book *book, *temp;
	book = (struct Book *)malloc(sizeof(struct Book));
	if (book == NULL)
	{
		printf("内存分配失败了\n");
 		exit(1);
	}
	getInput(book);
	if (*library != NULL)
	{
		temp = *library;
		*library = book;
		book->next = temp;
	}
	else
	{
		*library = book;
		book->next = NULL;
	}
}

void printLibrary(struct Book *library)
{
	struct Book *book;
	int count = 1;

	book = library;
	while (book != NULL)
	{
		printf("\nBook%d:",count);
		printf("书名：%s", book->title);
		printf("作者：%s", book->author);
		book = book->next;
		count++;
	}
}

void releaseLibrary(struct Book *library)
{
	while (library != NULL)
	{
		free(library);
		library = library->next;
	}
}

int main(void)
{
	struct Book *library = NULL;
	char ch;
	addBook(&library);
	while (1)
	{
		printf("是否输入书籍信息(Y/N)：");
		do
		{
			ch = getchar();
		} while (ch != 'Y' && ch != 'N');
		if (ch == 'Y')
		{
			addBook(&library);
		}else
		{
			break;
		}
	}
	printf("是否打印书籍信息(Y/N):");
	do
	{
		ch = getchar();
	} while (ch != 'Y' && ch != 'N');
	if (ch == 'Y')
	{
		printLibrary(library);
	}
	releaseLibrary(library);

	return 0;
}
```

## Go语言

```golang
package main

import (
	"fmt"
)

type Book struct {
	title, author string
	next          *Book
}

func getInput(book *Book) {
	fmt.Printf("书名：")
	fmt.Scanln(&book.title)
	fmt.Printf("作者：")
	fmt.Scanln(&book.author)
}

func addBook(library **Book) {
	book := new(Book)
	getInput(book)
	if library != nil {
		var temp *Book
		temp = *library
		*library = book
		book.next = temp
	} else {
		*library = book
		book.next = nil
	}
}

func printLibrary(library *Book) {
	var count = 1
	book := library
	for book != nil {
		fmt.Printf("\nBook:%d", count)
		fmt.Printf("书名:%s", book.title)
		fmt.Printf("作者:%s", book.author)
		book = book.next
		count++
	}
}

func main() {
	var library *Book = nil
	addBook(&library)
	for {
		var in string
		fmt.Println("是否输入书籍信息(Y/N):")
		fmt.Scanln(&in)
		if in == "Y" {
			addBook(&library)
		} else if in == "N" {
			break
		}
	}
	var out string
	fmt.Println("是否打印书籍信息(Y/N):")
	fmt.Scanln(&out)
	if out == "Y" {
		printLibrary(library)
	}
}
```

## Python语言

```python

class Book(object):
    def __init__(self, title, author):
        self.title = title
        self.author = author
        self.next = None


def get_input():
    title = input("书名：")
    author = input("作者：")
    return Book(title,author)


def add_book(library: Book) -> Book:
    book = get_input()
    if library:
        book.next = library
    return book


def print_library(library: Book):
    count = 1
    while True:
        if not library:
            break
        print("Book:", count)
        print("书名：", library.title)
        print("作者：", library.author)
        library = library.next
        count += 1


if __name__ == "__main__":
    library = get_input()
    while True:
        in_put = input("是否输入书籍信息（Y/N）：")
        if in_put == "Y":
            library = add_book(library)
        else:
            break
    out_put = input("是否打印书籍信息（Y/N）:")
    if out_put:
        print_library(library)
```
