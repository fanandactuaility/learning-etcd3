# Txn 方法

Txn 方法在单个事务中处理多个请求。

```java
rpc Txn(TxnRequest) returns (TxnResponse) {}
```

## 消息体

请求的消息体是 TxnRequest：

```java

// From google paxosdb paper:
// Our implementation hinges around a powerful primitive which we call MultiOp. All other database
// operations except for iteration are implemented as a single call to MultiOp. A MultiOp is applied atomically
// and consists of three components:
// 1. A list of tests called guard. Each test in guard checks a single entry in the database. It may check
// for the absence or presence of a value, or compare with a given value. Two different tests in the guard
// may apply to the same or different entries in the database. All tests in the guard are applied and
// MultiOp returns the results. If all tests are true, MultiOp executes t op (see item 2 below), otherwise
// it executes f op (see item 3 below).
// 2. A list of database operations called t op. Each operation in the list is either an insert, delete, or
// lookup operation, and applies to a single database entry. Two different operations in the list may apply
// to the same or different entries in the database. These operations are executed
// if guard evaluates to
// true.
// 3. A list of database operations called f op. Like t op, but executed if guard evaluates to false.
message TxnRequest {
  // compare is a list of predicates representing a conjunction of terms.
  // If the comparisons succeed, then the success requests will be processed in order,
  // and the response will contain their respective responses in order.
  // If the comparisons fail, then the failure requests will be processed in order,
  // and the response will contain their respective responses in order.
  repeated Compare compare = 1;
  // success is a list of requests which will be applied when compare evaluates to true.
  repeated RequestOp success = 2;
  // failure is a list of requests which will be applied when compare evaluates to false.
  repeated RequestOp failure = 3;
}


```

应答的消息体是 TxnResponse：

```java
message TxnResponse {
  ResponseHeader header = 1;
  // succeeded is set to true if the compare evaluated to true or false otherwise.
  bool succeeded = 2;
  // responses is a list of responses corresponding to the results from applying
  // success if succeeded is true or failure if succeeded is false.
  repeated ResponseOp responses = 3;
}
```