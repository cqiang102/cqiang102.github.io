---
title: flutter loading 动画插件（flutter_spinkit 4.1.2）
date: 2020-3-29 22:24:16
categories: 笔记
tags: [Flutter,dart]
type: posts
description: flutter插件使用demo
---
### [flutter_spinkit 4.1.2](https://github.com/jogboms/flutter_spinkit)

- pubspec.yaml 中添加依赖
```yaml
dependencies:
  flutter_spinkit: "^4.1.2"
```
- 阅读文档，看看案例。照着实现一个最简单的例子

![image](https://note.youdao.com/yws/public/resource/ecee5d952864b60d9fb997e61e338e01/xmlnote/1B43F3151BE44158A6E0E106DF890604/3056)

```
void main() => runApp(App());
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'loading 动画',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
      ),
      home: Scaffold(
        // SafeArea 异形屏适配
        body: SafeArea(
        // 层叠布局
          child: Stack(
            children: <Widget>[
              Positioned.fill(
                child: Container(
                  decoration: BoxDecoration(color: Colors.white),
                ),
              ),
              Positioned.fill(
                child: Container(
//                  color: Colors.blueAccent,
                  color: Colors.transparent,
                  child: SpinKitFadingCircle(
                    // 可以画一个圈，分为 0 1 2 3 4 5 6 7 8 9 份
                    itemBuilder: (_, int index) {
                      return DecoratedBox(
                        decoration: BoxDecoration(
                          // isEven 表示是否整数
                          color: index.isEven ? Colors.red : Colors.green,
                        ),
                        child: Text("$index"),
                      );
                    },
                    size: 120.0,
                  ),
                ),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```
- 模拟一个简单的场景，点击登录出现 loading 动画，一段时间后取消


![image](https://note.youdao.com/yws/public/resource/ecee5d952864b60d9fb997e61e338e01/xmlnote/FB15025133284553926676A1F11808D0/3054)


```
void main() => runApp(App());
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'loading 动画',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
      ),
      home: Scaffold(
        // SafeArea 异形屏适配
        body: BodyWidget(),
      ),
    );
  }
}
class BodyWidget extends StatefulWidget {
  @override
  _BodyWidgetState createState() => _BodyWidgetState();
}
class _BodyWidgetState extends State<BodyWidget> {
  bool ifLoading = false;
  Timer timer;
  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Stack(
        children: <Widget>[
          Positioned.fill(
            child: Container(
              decoration: BoxDecoration(color: Colors.white),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.end,
                children: <Widget>[
                  RaisedButton(
                    child: Text("登录"),
                    onPressed: () {
                      print("登录啊");
                      setState(() {
                        ifLoading = true;
                      });
                      // 三秒后取消动画
                      timer = new Timer(Duration(seconds: 3), () {
                        setState(() {
                          ifLoading = false;
                        });
                      });
                    },
                  ),
                ],
              ),
            ),
          ),
          ifLoading
              ? Positioned.fill(
                  child: Container(
                    color: Colors.blueAccent,
                    child: SpinKitFadingCircle(
                      // 可以画一个圈，分为 0 1 2 3 4 5 6 7 8 9 份
                      itemBuilder: (_, int index) {
                        return DecoratedBox(
                          decoration: BoxDecoration(
                            // isEven 表示是否整数
                            color: index.isEven ? Colors.red : Colors.green,
                          ),
                          child: Text("$index"),
                        );
                      },
                      size: 80.0,
                    ),
                  ),
                )
              : Container(),
        ],
      ),
    );
  }
}
```
- 引入 provider 模拟更加真实的例子
```yaml
dependencies:
  flutter_spinkit: "^4.1.2"
  provider: ^4.0.4
```
- 看到的效果是一样的
```
void main() => runApp(App());
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'loading 动画',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
      ),
      home: Scaffold(
        // SafeArea 异形屏适配
        body: MultiProvider(
          providers: [
            ChangeNotifierProvider(
              create: (_) => LoginProvider(),
            )
          ],
          child: Consumer<LoginProvider>(builder: (context,loginProvider,_){
            return BodyWidget(ifLoading: loginProvider.ifLoading,onPressed: loginProvider.onPressed,);
          },),
        ),
      ),
    );
  }
}
class LoginProvider with ChangeNotifier {
  bool ifLoading = false;
  VoidCallback onPressed;
  Timer timer;
  LoginProvider() {
    onPressed = () {
      ifLoading = true;
      // notifyListeners 我的理解就是把它当做 setState 用
      notifyListeners();
      // 模拟发送网络请求异步获得请求结果
      timer = new Timer(Duration(seconds: 3), () {
        ifLoading = false;
        notifyListeners();
      });
    };
  }
}
class BodyWidget extends StatelessWidget {
  bool ifLoading = false;
  VoidCallback onPressed;

  BodyWidget({@required this.ifLoading, @required this.onPressed});

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      child: Stack(
        children: <Widget>[
          Container(
            width: MediaQuery.of(context).size.width,
            height: MediaQuery.of(context).size.height,
            decoration: BoxDecoration(color: Colors.amberAccent),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.end,
              children: <Widget>[
                RaisedButton(
                  child: Text("登录"),
                  onPressed: onPressed,
                ),
              ],
            ),
          ),
          ifLoading
              ? Positioned.fill(
                  child: Container(
                    color: Colors.blueAccent,
                    child: SpinKitFadingCircle(
                      // 可以画一个圈，分为 0 1 2 3 4 5 6 7 8 9 份
                      itemBuilder: (_, int index) {
                        return DecoratedBox(
                          decoration: BoxDecoration(
                            // isEven 表示是否整数
                            color: index.isEven ? Colors.red : Colors.green,
                          ),
                          child: Text("$index"),
                        );
                      },
                      size: 120.0,
                    ),
                  ),
                )
              : Container(),
        ],
      ),
    );
  }
}

```