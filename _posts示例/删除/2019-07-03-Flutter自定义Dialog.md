---
layout:     post
title:      自定义Flutter Dialog的问题
subtitle:	  自定义Dialog方便人使用
date:       2019-06-22
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - Flutter
---


啥话不说上代码

```
class GenderChooseDialog extends Dialog {
  var title;
  var content;
  Function onBoyChooseEvent;
  Function onGirlChooseEvent;
  Function cancel;

  GenderChooseDialog({
    Key key,
    @required this.title,
    @required this.content,
    @required this.onBoyChooseEvent,
    @required this.onGirlChooseEvent,
    @required this.cancel,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Padding(
        padding: const EdgeInsets.all(12.0),
        child: new Material(
            type: MaterialType.transparency,
            child: new Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  new Container(
                      decoration: ShapeDecoration(
                          color: Color(0xFFFFFFFF),
                          shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.all(
                            Radius.circular(20.0),
                          ))),
//                    margin: EdgeInsets.only(left: (MediaQuery.of(context).size.height-300)/2,right: (MediaQuery.of(context).size.height-300)/2),
                      margin: EdgeInsets.only(left: (MediaQuery.of(context).size.width-300)/2,right: (MediaQuery.of(context).size.width-300)/2),
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          new Column(
                              mainAxisAlignment: MainAxisAlignment.center,
                              crossAxisAlignment: CrossAxisAlignment.center,
                              children: <Widget>[
                                new Padding(
                                    padding: const EdgeInsets.only(top: 30.0),
                                    child: Center(
                                        child: new Text(this.title,
                                            style: new TextStyle(
                                              color: Color(0xFF000000),
                                              fontSize: 18.0,
                                            )))),
                                new Padding(
                                  padding: const EdgeInsets.only(
                                      top: 10, bottom: 40,left: 20),
                                  child: new Row(
                                    children: <Widget>[
                                      Image(
                                          image: AssetImage(
                                              'assets/images/base/bulb.png'),
                                          width: 18,
                                          height: 17),
                                      new Text(this.content,
                                          style: new TextStyle(
                                            color: Color(0xFF999999),
                                            fontSize: 16.0,
                                          ))
                                    ],
                                  ),
                                ),
                                new Row(
                                    mainAxisAlignment:
                                        MainAxisAlignment.spaceEvenly,
                                    mainAxisSize: MainAxisSize.max,
                                    children: <Widget>[
                                      _genderChooseItemWid(1),
                                      _genderChooseItemWid(2)
                                    ])
                              ]),
                          new Padding(
                              padding: const EdgeInsets.only(top: 10),
                              child: GestureDetector(
                                  onTap: this.cancel,
                                  child: Image(
                                    image: AssetImage(
                                        'assets/images/base/cancel.png'),
                                    width: 28,
                                    height: 29,
                                  ))),
                        ],
                      ))
                ])));
  }

  Widget _genderChooseItemWid(var gender) {
    return GestureDetector(
        onTap: gender == 1 ? this.onBoyChooseEvent : this.onGirlChooseEvent,
        child: Container(
            height: 36,
            width: 90,
            margin:
                new EdgeInsets.only(top: 10, bottom: 30, left: 20, right: 5),
            decoration: BoxDecoration(
                // color: Colors.blueAccent,
                color: Color(gender == 1 ? 0xff7ED321 : 0xff7357FF),
                borderRadius: BorderRadius.all(Radius.circular(10.0))),
            child: Center(
                child: Text(gender == 1 ? "监课" : "上课",
                    textAlign: TextAlign.center,
                    style: TextStyle(
                        color: Colors.white,
                        decorationColor: Colors.red,
                        fontSize: 15.0)))));
  }
}
```

使用 

```
showDialog(
   		context: context,
		barrierDismissible: false,
		builder: (BuildContext context) {
  		return GenderChooseDialog(
		title: '小哥哥小姐姐请选择',
		onBoyChooseEvent: () {
			Navigator.pop(context);
		},
		onGirlChooseEvent: () {
			Navigator.pop(context);
		});
	});
```

参考

[Flutter 23: 图解自定义 Dialog 对话框](https://www.jianshu.com/p/6266dd5636f5)

