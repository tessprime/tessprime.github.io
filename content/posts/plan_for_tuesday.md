---
title: "Plan for tuesday"
date: 2023-05-01T21:23:58-07:00
---

What's the plan for Tuesday? We're going to consider the C function(s):

```c
int f(int b) {
    printf("%d\n", b);
}

int main() {
    f(7);
    f(8);
}
```

And ask/answer: "What are the values passed into printf?".

Might cheap out and do this in javascript instead. May or may not attempt to use CodeQL to do this.
