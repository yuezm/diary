# June-4

## Flutter

### NetWork I/O

#### HTTP 库 dio

```dart
Dio dio = new Dio();
Response res = await dio.get(&#39;http://www.baidu.com&#39;);
print(res.data.toString());

// 并发
Future.wait([ dio.get(xx), dio.post(xx) ]);

// 下载
dio.download(xx, savePath);

// 发送数据
FormData formData = new FormData.from({
});
response = await dio.post(&quot;/info&quot;, data: formData)
```

#### HTTP 分块下载

分块下载一定会加快下载速度吗：分块下载的最终速度受设备所在网络带宽、源出口速度、每个块大小、以及分块的数量等诸多因素影响，实际过程中很难保证速度最优

### WebSocket

web_socket_channel 包

```dart
final channel = IOWebSocketChannel.connect(url);

// StreamBuilder 监听客户端消息
new StreamBuilder(
  stream: channel.stream,
  builder: (context, snapshot){
    return new Text(snapshot.data.toString());
  }
);

// 发送
channel.sink.add(&#39;Hello!&#39;);

// 关闭
channel.sink.close();
```

## JSON 转换

```dart
import &#39;dart:convert&#39;;

json.decode()、json.encode();
```

#### json_serializable

开发阶段，自动话 JSON 源代码生成器

## 包和插件

1. dart 包，其中包含 flutter 代码，对 flutter 框架具有依赖性
2. 插件包：专用于 dart 包，包含 dart 编写的 api，其中包含对各个平台的兼容

由于 Flutter 本身只是一个 UI 系统，它本身是无法提供一些系统能力，比如使用蓝牙、相机、GPS 等，因此要在 Flutter APP 中调用这些能力就必须和原生平台进行通信。为此，Flutter 中提供了一个平台通道（platform channel），用于 Flutter 和原生平台的通信。平台通道正是 Flutter 和原生之间通信的桥梁，它也是 Flutter 插件的底层基础设施

### 包开发

```shell
flutter create --template=package $NAME
```

### Texture

当调用摄像头拍摄时，我们需要将图像显示到 flutter 中，如果此时按照消息机制来实现的话，则性能消耗巨大。flutter 利用 Texture 来实现

Texture 可以理解为 GPU 内保存将要绘制的图像数据的一个对象，Flutter engine 会将 Texture 的数据在内存中直接进行映射（而无需在原生和 Flutter 之间再进行数据传递），Flutter 会给每一个 Texture 分配一个 id，同时 Flutter 中提供了一个 Texture 组件

[Texture](Texture 可以理解为 GPU 内保存将要绘制的图像数据的一个对象，Flutter engine 会将 Texture 的数据在内存中直接进行映射（而无需在原生和 Flutter 之间再进行数据传递），Flutter 会给每一个 Texture 分配一个 id，同时 Flutter 中提供了一个 Texture 组件)

### PlatformView

Flutter SDK 中新增了 AndroidView 和 UIKitView 两个组件，这两个组件的主要功能就是将原生的 Android 组件和 iOS 组件嵌入到 Flutter 的组件树中。_使用 PlatformView 的开销是非常大的，因此，如果一个原生组件用 Flutter 实现的难度不大时，我们应该首选 Flutter 实现_

## 国际化

flutter_localizations 包

intl、intl_translation 包

[国际化常见问题](https://book.flutterchina.club/chapter13/faq.html)

## LeetCode

### 面试题 16.18. 模式匹配

思路：pattern 出现了 ca 个 a，和 cb = pattern - n 个 b，则得出公式  ca * la + cb * lb = value.length。由于 la 和  lb 必须为常数，则可以通过整数解。得知la，lb 再去 根据pattern反推value是否匹配

```typescript
function patternMatching(pattern: string, value: string): boolean {
  let ca = 0;
  let cb = 0;

  for (let i = 0; i < pattern.length; i++) {
    if (pattern[i] === 'a') ca++;
    else cb++;
  }

  if (pattern === '' && value === '') {
    return true;
  } else if (pattern === '') {
    return false;
  } else if (value === '') {
    return ca === 0 || cb === 0;
  } else {
    if (ca === 0) {
      const lb = value.length / cb;
      return Number.isInteger(lb) ? isMatch(0, lb) : false;
    }

    if (cb === 0) {
      const la = value.length / ca;
      return Number.isInteger(la) ? isMatch(la, 0) : false;
    }

    // 开始求解 ca * la + cb * lb = value.length;
    const maxLa = Math.trunc(value.length / ca);
    let la = 0;
    let lb = 0;
    while (la <= maxLa) {
      lb = (value.length - la * ca) / cb;
      if (!Number.isInteger(lb)) {
        la++;
        continue;
      }

      // 此时请求整数解 ca la    cb lb 开始结算是否匹配

      if (isMatch(la, lb)) {
        return true;
      }

      la++;
    }
  }

  return false;

  function isMatch(la: number, lb: number): boolean {
    // 确定长度后，开始取计算 sa，sb的值
    let sa = '';
    let sb = '';

    let valueIndex = 0;
    for (let i = 0; i < pattern.length; i++) {
      if (pattern[i] === 'a') {
        const t = value.slice(valueIndex, valueIndex + la);
        if (sa === '') sa = t;
        if (sa !== t) return false;
        valueIndex += la;
      } else {
        const t = value.slice(valueIndex, valueIndex + lb);
        if (sb === '') sb = t;
        if (sb !== t) return false;
        valueIndex += lb;
      }
    }
    return true;
  }
}
```

[面试题 16.18. 模式匹配](https://leetcode-cn.com/problems/pattern-matching-lcci/)

### 最接近的三数之和

思路：三数之和，看成（1数+2数）之和，确定一个数后，则按照双指针确定另外两数

```typescript
function threeSumClosest(nums: number[], target: number): number {
  nums.sort((a, b) => a - b);
  let result = nums[ 0 ] + nums[ 1 ] + nums[ 2 ];
  let i = 0;

  while (i < nums.length) {
    let t = target - nums[ i ];

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const s = nums[ left ] + nums[ right ];
      if (s === t) {
        return target; // 如果直接相等了，就不用再寻找了
      } else if (s < t) {
        left++;
      } else {
        right++;
      }
      result = Math.abs(s - t) < Math.abs(result - target) ? s + nums[ i ] : result;
    }

    i++;
  }

  return result;
}
```
