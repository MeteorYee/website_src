+++
title = "C++ coroutine"
date = "2024-03-19"
+++

The funtinality of coroutine in C++ is more of a semi-finished product. It doesn’t provide the users a handy way to invoke a coroutine. For instance, in Golang, we can set off a goroutine readily by letting go of any functions. On the contrary, C++’s post-20 standards come with some ‘gadgets’ which are the building blocks for the programmers to build a fully functioning architecture of coroutine.

Most of the content is taking reference to [1].

## Stackful vs Stackless
C++ coroutine is stackless which outperforms the stackful ones but trades off the code readability. Stackful coroutines fall short of the performance in that they share stack frames with each other. The sharing mechanism could be either implemented using a system call [getcontext](https://man7.org/linux/man-pages/man3/getcontext.3.html), or implemented by specific assembly code, and, hence, produces more overhead upon the context switch of coroutines. However, stackless coroutines only store the necessary information such as program counter and local variables without the need to allocate a whole stack, which is a significant performance gain against the stackful alternatives.

On the other hand, coding with stackful coroutines is more straight forward and the original codes don’t need to be revised too much, because the coroutines are still running on a normal stack frame similar to that of a thread, while jugging with stackless ones requires the programmers to be mentally aware of what they are doing and they have to explicitly tell the coroutines when they are going to yield/resume and adjust the funtions to a special return type.

In C++, there are three important objects that help the coroutine magic work: `std::coroutine_handle<>`, `Awaiter` and `promise_type`. What’s more, three operators deserve to be noticed as well: `co_await`, `co_yield`, and `co_return`.

## The Coroutine Handles/Objects
### Coroutine Handle
`std::coroutine_handle<>` is the key to access a coroutine and suspend/resume its execution. The behavior of the handle is analogous to a pointer. Hence, it’s pointing to the address of the state of a coroutine and can be destroyed explicitly.

### Awaiter
A coroutine is created through an “awaiter” object together with `co_await` operator. Awaiter is awaitable in that it’s able to suspend the execution of a coroutine and let programmers design the suspending logic via three specific methods.

_await_ready_: return false if it is going to suspend the coroutine, or true if it doesn’t want to yield its CPU for now.
_awat_suspend_: before the execution of the method, all preparation works of a coroutine state required by `std::coroutine_handle` have been finished. This is why it receives a parameter of a coroutine handle. In addition, it’s up to the programmers to decide what to do with the handle.
_await_resume_: the return type of the method is not necessarily to be void. If it returns a kind of object, that will be the evaluation value of the `co_await` operation.

```c++
struct Awaiter {
  std::coroutine_handle<> *hp_;
  constexpr bool await_ready() const noexcept { return false; }
  void await_suspend(std::coroutine_handle<> h) { *hp_ = h; }
  constexpr void await_resume() const noexcept {}
};
```

C++ provides two built-in awaiters: `std::suspend_never` and `std::suspend_always`. As the names show, one never switches out when calling co_await while another always switches out. Those two are really helpful when working with the `promise_type` illustrated below.

Moreover, `await_transform` can be used to customize the co_await behavior, which depends on the type of the expression evaluated. Basically, c++ will look for `await_transform` overloads when evaluating a `co_await` expression.

```c++
template <typename T>
struct MyAwaitable {
    T value;

    bool await_ready() { return true; }
    T await_resume() { return value; }
    void await_suspend(std::coroutine_handle<>) {}
};

MyAwaitable<int> await_transform(int value) {
    return {value}; // 42, as an int, fits into here
}

MyAwaitable<int> asyncFunction() {
    int result = co_await 42;
    co_return result;
}
```

### promise_type
C++’s standard requires the return type `R` of a coroutine must be an object with a nested type `R::promise_type`. The `promise_type` must define some fixed-name methods in order to fulfill its responsibilities.

- initial_suspend: return an awaiter object to decide whether to suspend the coroutine even before the execution of the user defined function
- final_suspend: decide whether to suspend the coroutine at the very last step. If it’s been suspended, it will be user’s responsibility to destroy the coroutine state. Otherwise, the coroutine state will be reclaimed automatically.
- get_return_object: this method helps the usre to get the return object they want. To get the coroutine handle from within the promise_type, one should call `std::coroutine_handle<promise_type>::from_promise(*this)`.
- unhandled_exception: this method provides an entry point to handle any exceptions thrwon frim within the user-defined function body. To obtain the exception, call `std::current_exception`.
- yield_value (optional): works with `co_yield`.
- return_void/return_value (optional?): works with `co_return`. However, `return_void` is optional __only when__ the coroutine never falls off the end of the function body, otherwise there would be undefined behavior. Hence, it is safe to have it defined in the promise_type object.

```c++
struct ReturnObject {
  struct promise_type {
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    ReturnObject get_return_object() { return {}; }
    void unhandled_exception() {}
    std::suspend_always yield_value(<yield-return-type> value) {
      value_ = value;
      return {};
    }
    void return_void() {}
  };
};
```

Given a user-defined coroutine function, the C++ compiler will compile the code into the snippet below.

```c++
{
    promise-type promise promise-constructor-arguments;
    try {
        co_await promise.initial_suspend();
        user_defined_function_body();
    } catch (...) {
        if (!initial-await-resume-called)
            throw;
        promise.unhandled_exception();
    }
    
    co_await promise.final_suspend();
}
// The coroutine state gets destroyed here.
```

## The Coroutine Operators
### co_await
The behavior of the operator can be customized, but it must go in tandem with an awaitable object.

### co_yield
`co_yield e` is equivalent to `co_await p.yield_value(e)`, where the `p` is the promise_type of the current coroutine.

### co_return
`co_return` => `promise.return_void()`
`co_return e` => `promise.return_value(e)`

## Reference
[1]. My tutorial and take on C++20 coroutines. https://www.scs.stanford.edu/~dm/blog/c++-coroutines.html