当多个线程访问同一区域的内存时，有可能会发生「竞态条件」。

## 互斥锁（mutex）

为了避免「竞争条件」的出现，我们一般会使用「互斥锁」，使得在同一时间内，进入「关键区」的线程最多只有一个。

例如，有多个线程同时访问一个计数器。我们使用「互斥锁」，“保护”关键区：

```c++
int main() {
    std::mutex mu;
    std::vector<std::thread> threads;
    int counter = 0;
    for (int i = 0; i < 5; i++) {
        threads.emplace_back([&counter, &mu]() {
            for (int i = 0; i < 100000; i++) {
                mu.lock();
                counter++; // 关键区
                mu.unlock();
            }
        });
    }
    for (auto& t : threads) {
        t.join();
    }
    printf("%d\n", counter);
    return 0;
}
```

之所以`counter++`要用 mutex 保护，是因为在一般情况下，`counter++`并非一步完成，而是会被拆成多步：

```c++
register = counter // 将内存中的数据读取到寄存器
register += 1      // 寄存器自增1
counter = register // 将数据从寄存器写回内存
```

然而，「互斥锁」是操作系统提供的接口。如果获取锁失败，线程则会暂时放弃执行权限，操作系统会将该线程挂起，直到该锁被其他线程释放。**由于涉及到上下文的切换，因此时间代价较高。**

## 原子变量

幸运的是，对于多线程计数这种场景，我们可以使用原子变量，从而避免 mutex 引起的上下文切换开销。

原子变量提供了一系列接口，包括了`fetch_add` 、`fetch_and`、`fetch_or`等等。与上文中的`counter++`不同，**原子操作一步即可完成，没有被拆分成多步。**因此，很大程度上避免了竞态条件。用原子变量实现的多线程计数如下：

```c++
int main() {
    std::vector<std::thread> threads;
    std::atomic<int> counter(0);
    for (int i = 0; i < 5; i++) {
        threads.emplace_back([&counter]() {
            for (int i = 0; i < 100000; i++) {
                counter.fetch_add(1);
            }
        });
    }
    for (auto& t : threads) {
        t.join();
    }
    printf("%d\n", counter.load());
    return 0;
}
```

### Compare And Swap（CAS）操作

原子变量的另一大利器是 CAS 操作。

先看一下 C++ 中 CAS 操作的接口：

```c++
bool compare_exchange_weak(
  T& expected,
  T desired,
  std::memory_order order = std::memory_order_seq_cst
);

bool compare_exchange_strong(
  T& expected,
  T desired,
  std::memory_order order = std::memory_order_seq_cst
);
```

`compare_exchage_weak`与`compare_exchange_strong`类似，先讲`comapre_exchange_strong`。这里还出现了`memory_order`，这个也等下再说。

假设我们定义了一个`std::atomic<int>`，然后执行以下操作：

```c++
int main() {
    std::atomic<int> a(0);
    int expected = 0;
    bool rv = a.compare_exchange_strong(expected, 1);
    printf("rv=%d a=%d expected=%d\n", int(rv), a.load(), expected);
    return 0;
}
```

程序会输出：

```
rv=1 a=1 expected=0
```

`compare_exchange_strong`做的事情其实就是：判断`a`的值是否与`expected`相等。若相等，将`desired`赋值给`a`，并且操作返回`true`；若不相等，则不改变`a`的值，而是将`a`的值赋值给`expected`，并且操作返回`false`。

#### 使用 CAS 操作的例子

