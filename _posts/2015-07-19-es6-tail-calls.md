---
layout: post
title: ES6 Tail Call Optimization
---

The goal of this post to give the reader some insight into the es6 tail call optimizations. What follow's has been influenced by my current SICP studies and from reading other blog posts.

The first thing to know about the ES6 tail call optimization is that it is an optimization which INTERPRETERS will implement. There is not some new JS symbol which denotes tail-call-optimization or anything like that, so if you are reading below looking for how the code has extra stuff in it, you will be left unsatisfied.

To talk about the tail call optimizations, it'll be useful to discuss the fibonacci sequence and about the difference between a recursive procedure and a recursive process. You may have seen a lot of this before, but I promise - it'll be worth the read.

Fibonacci sequence written as a recursive process:

{% highlight js %}

function fib(n) {
  if (n <= 1){
    return n;
  } else {
    return fib(n-1) + fib(n - 2);
  }
}

{% endhighlight %}

Below is a diagram of how this process will develop - each node in this tree represent a function call to fib.

{% highlight js %}
/*

                           fib(5)
                          /      \
                         /        \
                    fib(4)         fib(3)
                   /     \           /   \
                  /       \         /     \
            fib(3)         fib(2)    .........
            /  \            /  \
           /    \          /    \
        fib(2)  fib(1)  fib(1)  fib(0)
        /   \
       /     \
    fib(1)  fib(0)

*/

{% endhighlight %}

Notice, none of these nodes has any clue that they are part of a huge process and none of the nodes on this tree has enough information to encapsulate the entire state of the process. For example, the calls to fib(2) have no clue that they are part of a process to calculate the value of fib(5). So, what is keeping track of the process's state? The call stack.

The call stack keeps track of all of the ways that these function calls need to be combined after there calculations complete. The call stack is the one remembering to combine the calls of fib(4) and fib(3) to get fib(5). This is a recursive process: think of a recursive process as a series of deferred operations, where there is information hidden to each recursive call - that hidden information is in the call stack.

When executing this in a browser, there are a bunch of stack frames which are created created, with new environments for each call. You can see this in your chrome debugger.

Besides the fact that this algorithm repeats a lot of calculations unnecessarily (fib(1) is calculated many times), this algorithm is a poor one because it requires O(n) memory complexity on the call stack. Looking at the left branch of the tree, whatever n is the call stack will have n deferred stack frames.

Now, here is an iterative way to calculate fibonacci sequence:

{% highlight js %}

function fibIter(n){
  var a = 1, b = 0, temp;

  while (n > 0){
    temp = a;
    a = a + b;
    b = temp;
    n--;
  }

    return b;
}

/* To picture this, imagine a,b moving along the sequence as such:

b,a
0,1,1,2,3,5 ....

  b,a
0,1,1,2,3,5 ....

    b,a
0,1,1,2,3,5 ....

etc...

*/

{% endhighlight %}

This function will only use one stack frame on the call stack since there are no other function calls.
Now, what if we define this same method recursively?

{% highlight js %}

function fibIterRecursive(n, a, b){
  if (n === 0) {
    return b;
  } else {
    return fibIterRecursive(n-1, a + b, b);
  }
};

function fib(n){
  return fibIterRecursive(n, 1, 0);
}

{% endhighlight %}

Calculating fib(5), here is what the calls will look like

{% highlight js %}

fib(5)
fibIterRecursive(5, 1, 0)
fibIterRecursive(4, 1, 1)
fibIterRecursive(3, 2, 1)
fibIterRecursive(2, 3, 2)
fibIterRecursive(1, 5, 3)
fibIterRecursive(0, 8, 5)

{% endhighlight %}

In this implementation, there entire state of the process IS encapsulated in each function call. Notice, each call has enough information to complete the process for calculating fib(5), which is in contrast to the recursive process's nodes from above. If we stopped that process at fibIterRecursive(2, 3, 2) and then resumed it in a different enviroment with a clean call stack, we would still get the correct number.

So, one would hope that the call stack would not bother remembering a bunch of information in stack frames, since that would be unnecessary.  But, it still does in ES5. In ES6, the call stack won't - and that is the optimization.

In ES6, using the same exact code as fibIterRecursive, what will happen during each call is:

The interpreter will notice that the next recursive call does not need access to any of the local variables in the current stack frame, so it will CLEAR the current stack frame and REUSE IT. This is great because, in es5, calling fibIterRecursive(30000) will give you a stack overflow, but in es6 with the optimization, the stack frame will be reused and it won't cause a stack overflow (however, without a bigInt library, fibIterRecursive(30000) would return Infinity).

It's an interpreter thing, so there isn't anywhere to test it yet sadly : /

There is a creative way to get around this in es5. It's not used often. If you're interested check out http://raganwald.com/2013/03/28/trampolines-in-javascript.html .