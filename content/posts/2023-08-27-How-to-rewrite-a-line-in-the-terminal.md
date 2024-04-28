---
title: "How to Rewrite a Line in the Terminal"
date: 2023-08-27T09:45:19-03:00
tags:
  - terminal
  - c
draft: false
---

I have always liked when you run a command on the terminal and the output is updated in the same line, not having lots of prints in the terminal is something that I always wanted to do but never searched.

Example:
```bash
# Not this
Hello world!
1
2
3
4
Good Bye!

# This
Hello world!
4
Good Bye!
```

I always thought that this was fully done by your code with some special dependency, something like this:
```C
#include <stdio.h>
#include <magic.h>

int main() {
	printf("Hello world!\n");
	for (int i = 1; i < 5; ++i) {
		printf("%d", i);
		// Here something special that is done in code
		clearline();
	}
	printf("\nGood Bye!\n");
}
```

So I finally did a quick search and discovered the [`ANSI escape codes`](https://en.wikipedia.org/wiki/ANSI_escape_code) which should solve everything. The codes that I want are `\033[G` and `\033[K`, the first one for moving the terminal cursor to the beginning of the line and the second one to clear the line.

Modifying the previous code:
```C
#include <stdio.h>
// No more #include <magic.h>

int main() {
	printf("Hello world!\n");
	for (int i = 1; i < 5; ++i) {
		printf("%d\033[G\033[K", i);
	}
	printf("\nGood Bye!\n");
}
```

And with this I can get the expected result but doing this is boring, I want to see the numbers changing. This is why I decided to generate a simple timer in C.

And why not other languages?, it should work the same way!. Well then I will also do it on Go, Rust, Python and maybe Typescript or... Bash. With this simple exercise I will learn a few things from these languages.

---
You can get the code here: [github](https://github.com/Leonardo-de-Requesens/Timer)

