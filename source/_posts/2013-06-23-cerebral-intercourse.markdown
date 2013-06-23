---
layout: post
title: "cerebral intercourse"
date: 2013-06-23 12:49
comments: false
categories: [python, pl, interpreter, brainfuck]
---

One of my first interviews post-Hacker School was with HuffPo Labs, I
particularly enjoyed it because of the off-the-wall programing challenge they
gave me. Many of you will have heard of
[brainfuck](http://en.wikipedia.org/wiki/Brainfuck),
though I doubt as many of you will have actually programed in it. The HuffPo
Labs guys asked me to write a few simple programs in the language and afterwards
I wrote a simple [interpreter](https://github.com/ncollins/cerebral_intercourse).

The rather terse syntax makes reading brainfuck programs difficult, but the
other challenges of programing in the language can easily be explained in a
more familiar format. The Wikipedia article provides a direct translation of
brainfuck to C, but I'm going to use a C-like pseudo code in this article.

<h3>Basic operations</h3>

Brainfuck has a "tape" of data, which we can think of as an array or
linked-list that's infinite in both directions. Like a linked-list we can only
move left or right one cell at a time, we can't "jump"
to an arbitrary point in the tape. We also have the ability to increment or
or decrement the current byte by 1. Treating the tape as an
array, `data[]`, indexed by `i`, the following operations are permited:

```c
i++; // move the data pointer one to the right
i--; // move the data pointer one to the left
data[i]++; // increment the value at the data pointer
data[i]++; // decrement the value at the data pointer
```

As a result even simple changes to the data require multiple instructions.
It is assumed that we start at `i = 0`, so to add three to the byte stored at
`data[2]` we need to do the following:

```c
i++;
i++;
data[i]++;
data[i]++;
data[i]++;
```

In addition brainfuck allows us to read a single byte into the current place
in memory, and output the value of the current byte in memory.

```c
data[i] = input()
output(data[i])
```

Finally, brainfuck has a single control structure, equivalent to a while loop that runs
while the current data value is non-zero.

```c
while (data[i] > 0) {
    ...
}
```

<h3>Count down</h3>

The "hello world" of brainfuck takes a single byte as input and counts down
to zero printing each number in turn. This only requires a single data cell
to be used, so we don't need to move the data pointer at all.

```c Count Down
data[i] = input();
while (data[i] > 0) {
    print(data[i]);
    data[i]--;
}
print(data[i]);
```

<h3>Counting up</h3>

Since these loops only terminate when `data[i]` equals zero, other types of
loops have to be shoehorned into this format. To loop upwards we have to use a
second data cell: `data[0]` holds the loop variable which we decrement and
`data[1]` holds a byte which we increment and output.

```c Count Up
data[i] = input();
while (data[i] > 0) {
    data[i]--;
    i++;
    print(data[i]);
    data[i]++;
    i--;
}
i++;
print(data[i]);
```

<h3>If-Else</h3>

This didn't come up in the interview but I felt I should try it on my own as
if-else is a fundamental control structure of most programing languages.
Producing an if-else with arbirary predicates seemed a little too ambitious
so I set myself the following target: Take one number as input, print `1` if
it's non-zero and `0` otherwise.

The construction I came up with uses nested while loops and two data bytes.
The "equals zero" case is simple, if the input is zero then the first while
loop doesn't execute and the default value of zero is return from the second
byte. If the input is non-zero then both while loops
execute, the inner loop reduces `data[0]` to zero, ensuring that both while
loops terminate, and the outer while loop increments `data[1]` before terminating.

I've written it here with named variables, `in = data[0]` and `out = data[1]`,
for easy reading:

```c If-Else (with named variables)
in = input();
out = 0;
while (in > 0) {
    while (in > 0) {
        in--;
    }
    out++;
}
print(out);
```

It's then easy to replace `in`, `out` with `data[i]` and intersperse these
with `i++`s and `i--`s to move the data pointer back and forth.

```c If-Else
data[i] = input()
while (data[i] > 0) {
    while (data[i] > 0) {
        data[i]--;
    }
    i++;
    data[i]++;
    i--;
}
i++;
print(data[i]);
```
