+++
title = "Stylesheet"
author = ["Walker Griggs"]
draft = false
creator = "Emacs 29.4 (Org mode 9.6.15 + ox-hugo)"
weight = 2001
+++

**bold**, <span class="underline">underscore</span>, _italic_, `inline code`, [Hyperlink](https://walkergriggs.com)


## h1 {#h1}


### h2 {#h2}


#### h3 {#h3}


## Quote Block {#quote-block}

> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.


## Code Block {#code-block}

```lisp
(defun fizzbuzz ()
  (loop for x from 1 to 100 do
    (princ (cond ((zerop (mod x 15)) "FizzBuzz")
                 ((zerop (mod x 3))  "Fizz")
                 ((zerop (mod x 5))  "Buzz")
                 (t                  x)))
    (terpri)))
```


## Bulleted list {#bulleted-list}

-   foo
-   bar
-   buzz


## Numbered list {#numbered-list}

1.  foo
2.  bar
3.  buzz
