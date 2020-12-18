---
title: Flutter开发Linux程序中文乱码解决
date: 2020-12-08 21:44:00
categories: 笔记
tags: [Flutter]
type: posts
description: Flutter linux 中文乱码（Ubuntu）
---

# Flutter开发Linux程序中文乱码解决
> 盲猜由于字体缺失导致乱码，于是我把字体加过去

## 下载[微软雅黑字体](https://blog.lacia.cn/other/msyh.ttc "从我服务器下")，也可以直接从 Windows 里复制一个

![字体文件放在项目根目录 fonts 中对应 pubspec.yaml 配置](https://blog.lacia.cn/image/flutter_font_config.png "目录结构")

## pubspec.yaml 配置
```yaml
flutter:
  fonts:
    - family: cn
      fonts:
        - asset: fonts/msyh.ttc
```
## main.dart 配置
```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
        // 配置字体
        fontFamily: 'cn'
      ),
      home: HomePage(),
    );
  }
}
```