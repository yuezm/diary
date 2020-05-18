# 11 月第 4 周

## Javascript

## Typescript

## Node

### C++扩展

#### ObjectWrap

在 C++中封装 Javascript 类，当然也可以自己实现，也可以继承 ObjectWrap 类

```
#include <node/node.h>
#include <node/node_object_wrap.h>

#include <string>

namespace Entry {

using namespace v8;
using namespace std;

class MyObject : public node::ObjectWrap {
 public:
  static void Init(Local<Object> exports);
  double plusOne();

 protected:
  double value_;

  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const FunctionCallbackInfo<Value> &args);
  static void PlusOne(const FunctionCallbackInfo<Value> &args);
};

Persistent<Function> pers; // 持久句柄，为了让构造函数不使用new调用，必须让c++手动new的对象长时间存在

MyObject::MyObject(double value) : value_(value) {}

MyObject::~MyObject() {}

void MyObject::Init(Local<Object> exports) {
  Isolate *iso = Isolate::GetCurrent();
  Local<FunctionTemplate> ftp = FunctionTemplate::New(iso, MyObject::New);

  ftp->SetClassName(String::NewFromUtf8(iso, "MyObject"));
  ftp->InstanceTemplate()->SetInternalFieldCount(1);

  NODE_SET_PROTOTYPE_METHOD(ftp, "plusOne", PlusOne);

  pers.Reset(iso, ftp->GetFunction());  // 将函数赋予持久句柄
  exports->Set(String::NewFromUtf8(iso, "MyObject"), ftp->GetFunction());
}

void MyObject::New(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();

  if (args.IsConstructCall()) {  // new调用
    double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
    MyObject *obj = new MyObject(value);

    obj->Wrap(args.This());

    args.GetReturnValue().Set(args.This());
  } else {
    // 非new调用，手动生成实例
    const int argc = 1;
    Local<Value> argv[argc] = {args[0]};
    Local<Context> ctx = iso->GetCurrentContext();

    Local<Function> cons = Local<Function>::New(iso, pers);
    Local<Object> result = cons->NewInstance(ctx, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(result);
  }
}

void MyObject::PlusOne(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();
  MyObject *obj = ObjectWrap::Unwrap<MyObject>(args.Holder());

  args.GetReturnValue().Set(Number::New(iso, obj->plusOne()));
}

double MyObject::plusOne() { return ++this->value_; }

void INIT_ALL(Local<Object> exports) { MyObject::Init(exports); }

NODE_MODULE(addon, INIT_ALL);
}  // namespace Entry
```

##### 静态方法包裹，实例包裹，prototype 包裹，及对象传参

```
#include "node/node.h"
#include "node/node_object_wrap.h"

namespace Entry {
using namespace std;
using namespace v8;

Persistent<Function> pers; // 持久句柄，此处主要目的是当函数并非new调用时报错

class TestClass : public node::ObjectWrap {
 public:
  static void Init(Local<Object> exports);
  static void New(const FunctionCallbackInfo<Value> &args);
  static void CreateObject(const FunctionCallbackInfo<Value> &args);
  static void Add(const FunctionCallbackInfo<Value> &args);
  static void PlusOne(const FunctionCallbackInfo<Value> &args);

  double showValue();

 protected:
  TestClass(double _value);
  double value;

  double plusOne();
};

TestClass::TestClass(double _value) { value = _value; }

double TestClass::plusOne() { return ++value; }

double TestClass::showValue() { return value; }

void TestClass::New(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();

  double val = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();

  TestClass *testClass = new TestClass(val);
  testClass->Wrap(args.This());
  args.GetReturnValue().Set(args.This());
}

void TestClass::CreateObject(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();
  Local<Context> ctx = iso->GetCurrentContext();
  Local<Function> fn = Local<Function>::New(iso, pers);

  const int argc = 1;
  Local<Value> argv[argc] = {args[0]};
  Local<Object> result = fn->NewInstance(ctx, argc, argv).ToLocalChecked();

  args.GetReturnValue().Set(result);
}

void TestClass::PlusOne(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();
  TestClass *testClass = node::ObjectWrap::Unwrap<TestClass>(args.This());

  double val = testClass->plusOne();

  args.GetReturnValue().Set(Number::New(iso, val));
}

void TestClass::Init(Local<Object> exports) {
  Isolate *iso = Isolate::GetCurrent();

  Local<FunctionTemplate> ftp = FunctionTemplate::New(iso, New);
  ftp->SetClassName(String::NewFromUtf8(iso, "TestClass"));
  ftp->InstanceTemplate()->SetInternalFieldCount(1);

  NODE_SET_PROTOTYPE_METHOD(ftp, "plusOne", PlusOne);

  pers.Reset(iso, ftp->GetFunction());
  NODE_SET_METHOD(exports, "createObj", CreateObject);
  NODE_SET_METHOD(exports, "add", Add);
  // exports->Set(String::NewFromUtf8(iso, "TestClass"), ftp->GetFunction());
}

void TestClass::Add(const FunctionCallbackInfo<Value> &args) {
  Isolate *iso = args.GetIsolate();
  TestClass *t1 = node::ObjectWrap::Unwrap<TestClass>(args[0]->ToObject());
  TestClass *t2 = node::ObjectWrap::Unwrap<TestClass>(args[1]->ToObject());

  double val = t1->showValue() + t2->showValue();
  args.GetReturnValue().Set(Number::New(iso, val));
}

void Init(Local<Object> exports) { TestClass::Init(exports); }

NODE_MODULE(addon, Init);
}  // namespace Entry
```

#### 进程退出钩子

```
void Init(Local<Object> exports) {
  node::AtExit(exit); // 进程退出钩子
}
```

### NAN

node 安装包，提供 C++头文件，无序担心各个版本之间 APi 差异

```
npm i nan -D

// binding.gyp
{
  "targets": [
      {
        "target_name": "addon",
        "sources": ["main.cpp"],
        "include_dirs": [
            "<!(node -e \"require('nan')\")"
        ]
      }
  ]
}

```

```
NAN_METHOD(Echo){
 // 提供方法
}
// 相当于是 void Echo(const v8::FunctionCallbackInfo<v8::Value> &args) {}


NAN_MODULE_INIT(Init){
  // 提供target 相当于是export

  Nan::New<v8::String>("echo").ToLocalChecked(); // 取得字符串
  Nan::New<v8::FunctionTemplate>(Echo).ToLocalChecked(); // 取得函数
  Nan::Export(target,"echo",Echo); // 相当于 exports->Set(v8::String::NewFromUtf8(iso, "echo"), Echo->GetFunction());
  NAN_EXPORT(target,Echo)

  node::MakeCallback() // Function执行
  Nan::MakeCallback() // Function执行
}
// 相当于是 void Init(Local<Object> exports){}
```

```
Nan Map Demo

#include <nan.h>
#include <node.h>

namespace __NAN__ADD__ {
using namespace v8;

NAN_METHOD(Map) {
  Local<Array> array = info[0].As<Array>();
  Local<Function> fn = info[1].As<Function>();
  int len = array->Length();

  Local<Array> res = Nan::New<Array>(len);
  Local<Value> args[3] = {};
  args[2] = array;

  for (int i = 0; i < len; i++) {
    args[0] = array->Get(i);
    args[1] = Number::New(info.GetIsolate(), i);

    res->Set(i, Nan::MakeCallback(info.This(), fn, 3, args));
  }
  info.GetReturnValue().Set(res);
}

NAN_MODULE_INIT(Init) { Nan::Export(target, "map", Map); }
NODE_MODULE(addon, Init);
}  // namespace __NAN__ADD__
```

### NPM

## C++

## 网络

## 算法
