---
part: pixi/examples
title: Opencv example
description: How to run opencv using pixi
---

`opencv` 示例位于 `pixi` 仓库中。

```shell
git clone https://github.com/prefix-dev/pixi.git
```

进入示例文件夹

```shell
cd pixi/examples/opencv
```

## 人脸识别

运行 `start` 命令以启动人脸识别算法。

```shell
pixi run start
```

启动后的屏幕应该如下所示：
![](https://storage.googleapis.com/prefix-cms-images/docs/opencv_face_recognition.png)

查看 `webcame_capture.py` 了解我们是如何进行人脸识别的。

## 相机标定

除了人脸识别外，还包括了相机标定的示例。

你需要一个棋盘格来使这个功能生效。
打印这个：[![chessboard](https://github.com/opencv/opencv/blob/4.x/doc/pattern.png?raw=true)](https://github.com/opencv/opencv/blob/4.x/doc/pattern.png)

然后运行

```shell
pixi run calibrate
```

按 `SPACE` 键拍摄标定图片
大约拍摄 10 次，确保棋盘格出现在相机视野中

之后按 `ESC` 键开始标定。

标定完成后，相机将再次使用来计算棋盘格的距离。

![](https://storage.googleapis.com/prefix-cms-images/docs/calibration_board_detected.png)
