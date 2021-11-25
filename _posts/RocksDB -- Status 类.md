---
title: RocksDB -- Status 类
date: 2021-11-25 15:20:35
updated: 2021-11-25 18:51:19
categories: [数据库]
tags: [RocksDB,数据库]
---





## 1. 什么是 Status 类

Status 是 RocksDB 中大部分操作函数的返回值类型。

Status 类封装了 RocksDB 操作的结果。它可以指示成功，也可以指示带有相关错误信息的错误。

> 多个线程可以在没有外部同步的情况下调用 Status 类中的 const 方法，但是如果任何一个线程可以调用非 const 方法，那么访问相同 Status 的所有线程都必须使用外部同步。



Status 类定义在头文件 `status.h` 中，[点此查看](https://github.com/facebook/rocksdb/blob/v6.25.3/include/rocksdb/status.h)。



<!--more-->

## 2. Status 类内成员

Status 类内成员定义的部分代码如下：

```cpp
 protected:
  // A nullptr state_ (which is always the case for OK) means the message
  // is empty, else state_ points to message.

  Code code_;
  SubCode subcode_;
  Severity sev_;
  const char* state_;
#ifdef ROCKSDB_ASSERT_STATUS_CHECKED
  mutable bool checked_ = false;
#endif  // ROCKSDB_ASSERT_STATUS_CHECKED
```



* `code_`: 类型为 Code 的错误代码，这是最主要的代码，例如两个 Status 比较时，只要 `code_` 相等即认为两个 Status 相等。Code 是枚举类型，定义在下面写；
* `subcode_`: 类型为 SubCode 的错误代码，这是一个次级的错误代码，SubCode 是枚举类型，定义在下面写；
* `sev`: 类型为 Severity 的错误代码，Severity 是枚举类型，定义在下面写，另外从名字中可以看到，这个类型的错误是严峻的；
* `state_`: 这个代码中的注释有写。如果其是空指针 `nullptr`，那说明 Status 的状态是 OK，也就是没有错误，如果不是空指针，那么其指向一条错误信息字符串。
* `checked_`: 这个需要定义了宏 `ROCKSDB_ASSERT_STATUS_CHECKED` 才有效。这个值用来标记这个 Status 状态信息有没有被检查过，默认是 false，当 Status 被检查过后，会被修改为 true。



下面分别给出 Code，SubCode 和 Severity 的定义：

```cpp
enum Code : unsigned char {
  kOk = 0,
  kNotFound = 1,
  kCorruption = 2,
  kNotSupported = 3,
  kInvalidArgument = 4,
  kIOError = 5,
  kMergeInProgress = 6,
  kIncomplete = 7,
  kShutdownInProgress = 8,
  kTimedOut = 9,
  kAborted = 10,
  kBusy = 11,
  kExpired = 12,
  kTryAgain = 13,
  kCompactionTooLarge = 14,
  kColumnFamilyDropped = 15,
  kMaxCode
};
```

```cpp
enum SubCode : unsigned char {
  kNone = 0,
  kMutexTimeout = 1,
  kLockTimeout = 2,
  kLockLimit = 3,
  kNoSpace = 4,
  kDeadlock = 5,
  kStaleFile = 6,
  kMemoryLimit = 7,
  kSpaceLimit = 8,
  kPathNotFound = 9,
  KMergeOperandsInsufficientCapacity = 10,
  kManualCompactionPaused = 11,
  kOverwritten = 12,
  kTxnNotPrepared = 13,
  kIOFenced = 14,
  kMaxSubCode
};
```

```cpp
enum Severity : unsigned char {
  kNoError = 0,
  kSoftError = 1,
  kHardError = 2,
  kFatalError = 3,
  kUnrecoverableError = 4,
  kMaxSeverity
};
```



Code，SubCode 和 Severity 中的每个枚举值都可以从名字看出作用，就不解释了。

注：在 RocksDB 中，所有的常量，都以 `k` 开头。

-

另外，在 `status.cc` 中，还为 SubCode 中的每一个错误代码，都定义了一段对应的文字信息，用来在后面方便调用的，如下：

```cpp
static const char* msgs[static_cast<int>(Status::kMaxSubCode)] = {
    "",                                                   // kNone
    "Timeout Acquiring Mutex",                            // kMutexTimeout
    "Timeout waiting to lock key",                        // kLockTimeout
    "Failed to acquire lock due to max_num_locks limit",  // kLockLimit
    "No space left on device",                            // kNoSpace
    "Deadlock",                                           // kDeadlock
    "Stale file handle",                                  // kStaleFile
    "Memory limit reached",                               // kMemoryLimit
    "Space limit reached",                                // kSpaceLimit
    "No such file or directory",                          // kPathNotFound
    // KMergeOperandsInsufficientCapacity
    "Insufficient capacity for merge operands",
    // kManualCompactionPaused
    "Manual compaction paused",
    " (overwritten)",    // kOverwritten, subcode of OK
    "Txn not prepared",  // kTxnNotPrepared
    "IO fenced off",     // kIOFenced
};
```



## 3. Status 类内方法



### 3.1. Status 检查标记

Status 是一个 RocksDB 大部分操作的返回值类型，用来说明操作的执行结果。

RocksDB 的代码中，认为 Status 中的内容是应该被检查的（即你应该检查 Status 来判断你执行的操作是成功了还是失败了），所以这里有几个函数，来标记、修改 Status 的检查状态（对应修改的就是 Status 的成员 `checked_`）。

首先要知道，这里有一个宏 `ROCKSDB_ASSERT_STATUS_CHECKED`，当这个宏被设定时，RocksDB 就认为这个 Status 已经被检查过了（`checked_` 应当为 true）。

然后我们再看那几个函数。

#### 3.1.1. 将 Status 标记为已检查（protected）

```cpp
  inline void MarkChecked() const {
#ifdef ROCKSDB_ASSERT_STATUS_CHECKED
    checked_ = true;
#endif  // ROCKSDB_ASSERT_STATUS_CHECKED
  }
```

这个函数是 protected 的。

**Status 中查看过错误代码、信息的大部分方法，都会调用此函数，这个后面就不重复说了。**

#### 3.1.2. 将 Status 标记为未检查

```cpp
  inline void MustCheck() const {
#ifdef ROCKSDB_ASSERT_STATUS_CHECKED
    checked_ = false;
#endif  // ROCKSDB_ASSERT_STATUS_CHECKED
  }
```

#### 3.1.3. 接收 Status 中的错误

```cpp
inline void PermitUncheckedError() const { MarkChecked(); }
```

如果用户有意地接收错误，则必须显式地调用此函数。

通过这种方式，我们可以很容易地搜索代码，找到发生错误的地方。



### 3.2. 构造与析构

Status 对象通常不是由用户主动创建的，而是在执行一些操作的时候由操作函数创建，所以我们一般只查看 Status 中的信息（操作结果），而不会去手动创建一个 Status 对象。

不过既然是看源码，构造和析构函数还是要说的。

Public 的构造函数有 7 个重载，还有 2 个 Protected 的构造。

#### 3.2.1. 默认构造

```cpp
Status() : code_(kOk), subcode_(kNone), sev_(kNoError), state_(nullptr) {}
```

默认构造就是上面三个错误代码都初始化为没有错误，然后错误信息指针初始化为空指针。

#### 3.2.2. 拷贝和赋值构造

拷贝和赋值构造的声明和定义是分开的，但是都在 `status.h` 中，  
共有四个，写在一起吧，声明和实现分别如下：

```cpp
// Copy the specified status.
Status(const Status& s);

Status& operator=(const Status& s);

Status(Status&& s)
#if !(defined _MSC_VER) || ((defined _MSC_VER) && (_MSC_VER >= 1900))
      noexcept
#endif
      ;

Status& operator=(Status&& s)
#if !(defined _MSC_VER) || ((defined _MSC_VER) && (_MSC_VER >= 1900))
      noexcept
#endif
      ;
```

```cpp
inline Status::Status(const Status& s)
    : code_(s.code_), subcode_(s.subcode_), sev_(s.sev_) {
  s.MarkChecked();
  state_ = (s.state_ == nullptr) ? nullptr : CopyState(s.state_);
}

inline Status& Status::operator=(const Status& s) {
  if (this != &s) {
    s.MarkChecked();
    MustCheck();
    code_ = s.code_;
    subcode_ = s.subcode_;
    sev_ = s.sev_;
    delete[] state_;
    state_ = (s.state_ == nullptr) ? nullptr : CopyState(s.state_);
  }
  return *this;
}

inline Status::Status(Status&& s)
#if !(defined _MSC_VER) || ((defined _MSC_VER) && (_MSC_VER >= 1900))
    noexcept
#endif
    : Status() {
  s.MarkChecked();
  *this = std::move(s);
}

inline Status& Status::operator=(Status&& s)
#if !(defined _MSC_VER) || ((defined _MSC_VER) && (_MSC_VER >= 1900))
    noexcept
#endif
{
  if (this != &s) {
    s.MarkChecked();
    MustCheck();
    code_ = std::move(s.code_);
    s.code_ = kOk;
    subcode_ = std::move(s.subcode_);
    s.subcode_ = kNone;
    sev_ = std::move(s.sev_);
    s.sev_ = kNoError;
    delete[] state_;
    state_ = nullptr;
    std::swap(state_, s.state_);
  }
  return *this;
}
```

拷贝和赋值构造，还是比较常规的，但是有几个地方还是要说一下。

第 1 个和第 3个，参数是 `&`，这两个拷贝（赋值）构造，会将原来的 Status 设定为已检查过，然后将新的 Status 设定为未检查状态。

第 2 个和第 4 个，参数是 `&&` 右值引用，也就是说，当拷贝（赋值）构造执行完后，原来的 Status 将失效。

-

上面的代码中，有一个函数 `CopyState()`，定义在 `status.cc` 中，就是一个深拷贝字符串的函数，实现如下：

```cpp
const char* Status::CopyState(const char* state) {
#ifdef OS_WIN
  const size_t cch = std::strlen(state) + 1;  // +1 for the null terminator
  char* result = new char[cch];
  errno_t ret
#if defined(_MSC_VER)
    ;
#else
    __attribute__((__unused__));
#endif
  ret = strncpy_s(result, cch, state, cch - 1);
  result[cch - 1] = '\0';
  assert(ret == 0);
  return result;
#else
  const size_t cch = std::strlen(state) + 1;  // +1 for the null terminator
  return std::strncpy(new char[cch], state, cch);
#endif
}
```



#### 3.2.4. 拷贝构造 + 单独设定 `sev_`

```cpp
inline Status::Status(const Status& s, Severity sev)
    : code_(s.code_), subcode_(s.subcode_), sev_(sev) {
  s.MarkChecked();
  state_ = (s.state_ == nullptr) ? nullptr : CopyState(s.state_);
}
```

代码很简单，就是就普通的，拷贝另一个 Status 中的内容，但是重设了 `sev_`。

同样的，这个构造会把参数里的那个 Statu 标记为已检查。

#### 3.2.5. 显式指定所有成员值的构造

```cpp
Status(Code _code, SubCode _subcode, Severity _sev, const Slice& msg)
  : Status(_code, _subcode, msg, "", _sev) {}
```

这个构造函数调用了另一个 protected 的构造函数，结果就是指定了所有成员值（与参数对应）。

关于其调用的另一个 protected 构造，在下面 3.2.7 说。

#### 3.2.6. 指定部分成员值的构造（protected）

```cpp
explicit Status(Code _code, SubCode _subcode = kNone)
    : code_(_code), subcode_(_subcode), sev_(kNoError), state_(nullptr) {}
```

这个实在是太简单了，不说了。

#### 3.2.7. 指定成员值的构造（可拼接信息）（protected）

```cpp
Status(Code _code, SubCode _subcode, const Slice& msg, const Slice& msg2,
       Severity sev = kNoError);
```

这个函数在 `status.h` 中只有声明，其实现在 `status.cc` 中，[点此查看源文件](https://github.com/facebook/rocksdb/blob/v6.25.3/util/status.cc#L60)。

其实现如下：

```cpp
Status::Status(Code _code, SubCode _subcode, const Slice& msg,
               const Slice& msg2, Severity sev)
    : code_(_code), subcode_(_subcode), sev_(sev) {
  assert(subcode_ != kMaxSubCode);
  const size_t len1 = msg.size();
  const size_t len2 = msg2.size();
  const size_t size = len1 + (len2 ? (2 + len2) : 0);
  char* const result = new char[size + 1];  // +1 for null terminator
  memcpy(result, msg.data(), len1);
  if (len2) {
    result[len1] = ':';
    result[len1 + 1] = ' ';
    memcpy(result + len1 + 2, msg2.data(), len2);
  }
  result[size] = '\0';  // null terminator for C style string
  state_ = result;
}
```

这个实现还是非常好理解的，除了 `state_`，其他成员都是直接用对应形参赋值的。

关于 `state_`，代码也是比较好理解的，结果是将 `msg` 和 `msg2` 这两个参数按格式 `msg: msg2` 拼接起来（中间加了一个冒号一个空格），然后赋值给 `state_`。

#### 3.2.8. 指定部分成员值的构造（可拼接信息）（protected）

```cpp
Status(Code _code, const Slice& msg, const Slice& msg2)
  : Status(_code, kNone, msg, msg2) {}
```

这个构造函数调用了 3.2.7 中的那个构造函数，指定了 `_code` 值，再拼接下字符串，没什么其他的。



#### 3.2.9. 析构

```cpp
  ~Status() { 
#ifdef ROCKSDB_ASSERT_STATUS_CHECKED
    if (!checked_) {
      fprintf(stderr, "Failed to check Status %p\n", this);
      port::PrintStack();
      abort();
    }
#endif  // ROCKSDB_ASSERT_STATUS_CHECKED
    delete[] state_;
  }
```

析构函数还是很简单的，只需要释放 `state_` 指向的空间。但析构时，如果 Status 其中的内容未检查，RocksDB 会打印出一条错误信息，给出警告。



### 3.3. 运算符重载

除了赋值构造重载的 `=` 运算符外，Status 类中还重载了 `==` 和 `!=` 两个运算符。

这两个重载的定义和声明均在 `status.h` 中，但没有写在一起，如下：

```cpp
bool operator==(const Status& rhs) const;
bool operator!=(const Status& rhs) const;
```

```cpp
inline bool Status::operator==(const Status& rhs) const {
  MarkChecked();
  rhs.MarkChecked();
  return (code_ == rhs.code_);
}

inline bool Status::operator!=(const Status& rhs) const {
  MarkChecked();
  rhs.MarkChecked();
  return !(*this == rhs);
}
```

通过代码可以注意到，两个 Status 对象，只要其中的 `code_` 一样，就认为相等，而无关 `subcode_`、`sev_`、`state_` 和 `checked_`。



### 3.4. 获取类内成员的值

Status 类中提供了几个方法用来获取 `code_`、`subcode_`、`sev_` 和`state_` 的值。

同样的，只要查看过 Status 内的信息，RocksDB 就会会把 Status 标记为已检查（调用 `MarkChecked()` 函数）。

下面列出查看上述几个成员值的函数，比较简单，就不解释了。

```cpp
// 获取 code_
Code code() const {
  MarkChecked();
  return code_;
}
```

```cpp
// 获取 subcode_
SubCode subcode() const {
  MarkChecked();
  return subcode_;
}
```

```cpp
// 获取 sev_
Severity severity() const {
  MarkChecked();
  return sev_;
}
```

```cpp
// 获取 state_
const char* getState() const {
  MarkChecked();
  return state_;
}
```

### 3.5. 创建 Status（程序操作）

之前说过，Status 是大多 RocksDB 操作的返回值类型，其中存储的是操作的结果反馈（成功或者失败信息）。

所以说，我们几乎不会去手动创建一个 Status 对象，Status 对象一般有操作函数来创建。

操作函数创建一个 Status 对象，也是调用 Status 类中提供的方法，这些方法都是 `static` 的，然后这些方法再去调用合适的构造函数，传递参数，创建并返回一个 Status。

下面给出这些方法的代码，由于我们前面已经讲过了 Status 的构造函数相关内容，所以这些方法都很简单，都可以从名字看出其作用，注释也有简单解释，就不多说了。

```cpp
// Return a success status.
static Status OK() { return Status(); }

// Successful, though an existing something was overwritten
// Note: using variants of OK status for program logic is discouraged,
// but it can be useful for communicating statistical information without
// changing public APIs.
static Status OkOverwritten() { return Status(kOk, kOverwritten); }

// Return error status of an appropriate type.
static Status NotFound(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kNotFound, msg, msg2);
}

// Fast path for not found without malloc;
static Status NotFound(SubCode msg = kNone) { return Status(kNotFound, msg); }

static Status NotFound(SubCode sc, const Slice& msg,
                       const Slice& msg2 = Slice()) {
  return Status(kNotFound, sc, msg, msg2);
}

static Status Corruption(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kCorruption, msg, msg2);
}
static Status Corruption(SubCode msg = kNone) {
  return Status(kCorruption, msg);
}

static Status NotSupported(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kNotSupported, msg, msg2);
}
static Status NotSupported(SubCode msg = kNone) {
  return Status(kNotSupported, msg);
}

static Status InvalidArgument(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kInvalidArgument, msg, msg2);
}
static Status InvalidArgument(SubCode msg = kNone) {
  return Status(kInvalidArgument, msg);
}

static Status IOError(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kIOError, msg, msg2);
}
static Status IOError(SubCode msg = kNone) { return Status(kIOError, msg); }

static Status MergeInProgress(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kMergeInProgress, msg, msg2);
}
static Status MergeInProgress(SubCode msg = kNone) {
  return Status(kMergeInProgress, msg);
}

static Status Incomplete(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kIncomplete, msg, msg2);
}
static Status Incomplete(SubCode msg = kNone) {
  return Status(kIncomplete, msg);
}

static Status ShutdownInProgress(SubCode msg = kNone) {
  return Status(kShutdownInProgress, msg);
}
static Status ShutdownInProgress(const Slice& msg,
                                 const Slice& msg2 = Slice()) {
  return Status(kShutdownInProgress, msg, msg2);
}
static Status Aborted(SubCode msg = kNone) { return Status(kAborted, msg); }
static Status Aborted(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kAborted, msg, msg2);
}

static Status Busy(SubCode msg = kNone) { return Status(kBusy, msg); }
static Status Busy(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kBusy, msg, msg2);
}

static Status TimedOut(SubCode msg = kNone) { return Status(kTimedOut, msg); }
static Status TimedOut(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kTimedOut, msg, msg2);
}

static Status Expired(SubCode msg = kNone) { return Status(kExpired, msg); }
static Status Expired(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kExpired, msg, msg2);
}

static Status TryAgain(SubCode msg = kNone) { return Status(kTryAgain, msg); }
static Status TryAgain(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kTryAgain, msg, msg2);
}

static Status CompactionTooLarge(SubCode msg = kNone) {
  return Status(kCompactionTooLarge, msg);
}
static Status CompactionTooLarge(const Slice& msg,
                                 const Slice& msg2 = Slice()) {
  return Status(kCompactionTooLarge, msg, msg2);
}

static Status ColumnFamilyDropped(SubCode msg = kNone) {
  return Status(kColumnFamilyDropped, msg);
}

static Status ColumnFamilyDropped(const Slice& msg,
                                  const Slice& msg2 = Slice()) {
  return Status(kColumnFamilyDropped, msg, msg2);
}

static Status NoSpace() { return Status(kIOError, kNoSpace); }
static Status NoSpace(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kIOError, kNoSpace, msg, msg2);
}

static Status MemoryLimit() { return Status(kAborted, kMemoryLimit); }
static Status MemoryLimit(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kAborted, kMemoryLimit, msg, msg2);
}

static Status SpaceLimit() { return Status(kIOError, kSpaceLimit); }
static Status SpaceLimit(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kIOError, kSpaceLimit, msg, msg2);
}

static Status PathNotFound() { return Status(kIOError, kPathNotFound); }
static Status PathNotFound(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kIOError, kPathNotFound, msg, msg2);
}

static Status TxnNotPrepared() {
  return Status(kInvalidArgument, kTxnNotPrepared);
}
static Status TxnNotPrepared(const Slice& msg, const Slice& msg2 = Slice()) {
  return Status(kInvalidArgument, kTxnNotPrepared, msg, msg2);
}
```





### 3.6. 查看 Status（用户操作）（重要）

前面说过，创建 Status 对象一般都是程序自己完成的，我们在使用 RocksDB 时，通常不需要关注 Status 的创建过程，我们主要还是查看操作返回 Status 中的结果。



#### 3.6.1. 成功



一般我们最常用的，是判断操作结果是否成功，函数如下：

```cpp
// Returns true iff the status indicates success.
bool ok() const {
  MarkChecked();
  return code() == kOk;
}
```

当我们调用 Status 对象的这个成员函数 `ok()` 时，如果返回结果为 true，就说明操作成功了。

还有一个是，操作成功了，但覆盖掉了一些原有的内容，如下：

```cpp
// Returns true iff the status indicates success *with* something
// overwritten
bool IsOkOverwritten() const {
  MarkChecked();
  return code() == kOk && subcode() == kOverwritten;
}
```





#### 3.6.2. 错误

判断错误类型的函数比较多，下面看源码，都比较简单，可以从名字和注释中看出作用。

```cpp
// Returns true iff the status indicates a NotFound error.
bool IsNotFound() const {
  MarkChecked();
  return code() == kNotFound;
}

// Returns true iff the status indicates a Corruption error.
bool IsCorruption() const {
  MarkChecked();
  return code() == kCorruption;
}

// Returns true iff the status indicates a NotSupported error.
bool IsNotSupported() const {
  MarkChecked();
  return code() == kNotSupported;
}

// Returns true iff the status indicates an InvalidArgument error.
bool IsInvalidArgument() const {
  MarkChecked();
  return code() == kInvalidArgument;
}

// Returns true iff the status indicates an IOError.
bool IsIOError() const {
  MarkChecked();
  return code() == kIOError;
}

// Returns true iff the status indicates an MergeInProgress.
bool IsMergeInProgress() const {
  MarkChecked();
  return code() == kMergeInProgress;
}

// Returns true iff the status indicates Incomplete
bool IsIncomplete() const {
  MarkChecked();
  return code() == kIncomplete;
}

// Returns true iff the status indicates Shutdown In progress
bool IsShutdownInProgress() const {
  MarkChecked();
  return code() == kShutdownInProgress;
}

bool IsTimedOut() const {
  MarkChecked();
  return code() == kTimedOut;
}

bool IsAborted() const {
  MarkChecked();
  return code() == kAborted;
}

bool IsLockLimit() const {
  MarkChecked();
  return code() == kAborted && subcode() == kLockLimit;
}

// Returns true iff the status indicates that a resource is Busy and
// temporarily could not be acquired.
bool IsBusy() const {
  MarkChecked();
  return code() == kBusy;
}

bool IsDeadlock() const {
  MarkChecked();
  return code() == kBusy && subcode() == kDeadlock;
}

// Returns true iff the status indicated that the operation has Expired.
bool IsExpired() const {
  MarkChecked();
  return code() == kExpired;
}

// Returns true iff the status indicates a TryAgain error.
// This usually means that the operation failed, but may succeed if
// re-attempted.
bool IsTryAgain() const {
  MarkChecked();
  return code() == kTryAgain;
}

// Returns true iff the status indicates the proposed compaction is too large
bool IsCompactionTooLarge() const {
  MarkChecked();
  return code() == kCompactionTooLarge;
}

// Returns true iff the status indicates Column Family Dropped
bool IsColumnFamilyDropped() const {
  MarkChecked();
  return code() == kColumnFamilyDropped;
}

// Returns true iff the status indicates a NoSpace error
// This is caused by an I/O error returning the specific "out of space"
// error condition. Stricto sensu, an NoSpace error is an I/O error
// with a specific subcode, enabling users to take the appropriate action
// if needed
bool IsNoSpace() const {
  MarkChecked();
  return (code() == kIOError) && (subcode() == kNoSpace);
}

// Returns true iff the status indicates a memory limit error.  There may be
// cases where we limit the memory used in certain operations (eg. the size
// of a write batch) in order to avoid out of memory exceptions.
bool IsMemoryLimit() const {
  MarkChecked();
  return (code() == kAborted) && (subcode() == kMemoryLimit);
}

// Returns true iff the status indicates a PathNotFound error
// This is caused by an I/O error returning the specific "no such file or
// directory" error condition. A PathNotFound error is an I/O error with
// a specific subcode, enabling users to take appropriate action if necessary
bool IsPathNotFound() const {
  MarkChecked();
  return (code() == kIOError || code() == kNotFound) &&
    (subcode() == kPathNotFound);
}

// Returns true iff the status indicates manual compaction paused. This
// is caused by a call to PauseManualCompaction
bool IsManualCompactionPaused() const {
  MarkChecked();
  return (code() == kIncomplete) && (subcode() == kManualCompactionPaused);
}

// Returns true iff the status indicates a TxnNotPrepared error.
bool IsTxnNotPrepared() const {
  MarkChecked();
  return (code() == kInvalidArgument) && (subcode() == kTxnNotPrepared);
}

// Returns true iff the status indicates a IOFenced error.
bool IsIOFenced() const {
  MarkChecked();
  return (code() == kIOError) && (subcode() == kIOFenced);
}
```

#### 3.6.2. `ToString()` 函数

除了通过上面的方法，判断成功或某些错误 true 或 false 以外，我们可能也关注具体点的信息，或者不想逐个错误去判断，只想直接看结果，就要用到 `ToString()` 函数。

```cpp
// Return a string representation of this status suitable for printing.
// Returns the string "OK" for success.
std::string ToString() const;
```

这个函数，当操作成功的时候，返回 "OK"，当操作出错的时候，会返回具体的错误信息。

其函数实现在 `status.cc` 文件中，比较简单而且不是很重要，这里就不说了。给出其实现代码如下：

```cpp
std::string Status::ToString() const {
#ifdef ROCKSDB_ASSERT_STATUS_CHECKED
  checked_ = true;
#endif  // ROCKSDB_ASSERT_STATUS_CHECKED
  const char* type = nullptr;
  switch (code_) {
    case kOk:
      return "OK";
    case kNotFound:
      type = "NotFound: ";
      break;
    case kCorruption:
      type = "Corruption: ";
      break;
    case kNotSupported:
      type = "Not implemented: ";
      break;
    case kInvalidArgument:
      type = "Invalid argument: ";
      break;
    case kIOError:
      type = "IO error: ";
      break;
    case kMergeInProgress:
      type = "Merge in progress: ";
      break;
    case kIncomplete:
      type = "Result incomplete: ";
      break;
    case kShutdownInProgress:
      type = "Shutdown in progress: ";
      break;
    case kTimedOut:
      type = "Operation timed out: ";
      break;
    case kAborted:
      type = "Operation aborted: ";
      break;
    case kBusy:
      type = "Resource busy: ";
      break;
    case kExpired:
      type = "Operation expired: ";
      break;
    case kTryAgain:
      type = "Operation failed. Try again.: ";
      break;
    case kCompactionTooLarge:
      type = "Compaction too large: ";
      break;
    case kColumnFamilyDropped:
      type = "Column family dropped: ";
      break;
    case kMaxCode:
      assert(false);
      break;
  }
  char tmp[30];
  if (type == nullptr) {
    // This should not happen since `code_` should be a valid non-`kMaxCode`
    // member of the `Code` enum. The above switch-statement should have had a
    // case assigning `type` to a corresponding string.
    assert(false);
    snprintf(tmp, sizeof(tmp), "Unknown code(%d): ", static_cast<int>(code()));
    type = tmp;
  }
  std::string result(type);
  if (subcode_ != kNone) {
    uint32_t index = static_cast<int32_t>(subcode_);
    assert(sizeof(msgs) / sizeof(msgs[0]) > index);
    result.append(msgs[index]);
  }

  if (state_ != nullptr) {
    if (subcode_ != kNone) {
      result.append(": ");
    }
    result.append(state_);
  }
  return result;
}
```

