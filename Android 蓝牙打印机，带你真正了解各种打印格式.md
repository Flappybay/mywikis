> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/ec0cdcd62595)

> **注意：**本文原创，转载请注明出处。欢迎关注我的 [简书](https://www.jianshu.com/users/17aed505615e/latest_articles) 。

本文主要讲解蓝牙打印机在打印小票的过程中，如何打印各种常见格式。由于之前需要调试打印格式，但是苦于网上没有详细的讲解教程，无奈只能自给自足，自己封装了一个。如果各位盆友正在或者曾经苦恼蓝牙打印机的打印格式，那么恭喜你，本篇博文就是你要寻找的。

什么是蓝牙打印机
--------

是的，你没看错。一开始兔子哥先来介绍一下什么是蓝牙打印机。。。好吧，这个就交给百度了：

> 蓝牙打印机（Bluetooth printer）就是将蓝牙技术应用在打印机上，摆脱[打印机](https://link.jianshu.com?t=http://baike.baidu.com/view/7836.htm)连线所带来的不便，实现无线打印，可以减少桌面上令人不快的电缆，并且可以将打印机远离主机任意搬动，摆放在房间中适合的位置。

常见小票样式
------

![](http://upload-images.jianshu.io/upload_images/1467310-b58bab95db2fda1b.png) Paste_Image.png

这个小票格式基本就是最常见的了。这里面的各种格式，都可以从蓝牙打印机的 API 里面找到。蓝牙打印机有好多 API，我把常用的给封装了一下：`PrintUtils.java`

```
/**
 * 复位打印机
 */
public static final byte[] RESET = {0x1b, 0x40};

/**
 * 左对齐
 */
public static final byte[] ALIGN_LEFT = {0x1b, 0x61, 0x00};

/**
 * 中间对齐
 */
public static final byte[] ALIGN_CENTER = {0x1b, 0x61, 0x01};

/**
 * 右对齐
 */
public static final byte[] ALIGN_RIGHT = {0x1b, 0x61, 0x02};

/**
 * 选择加粗模式
 */
public static final byte[] BOLD = {0x1b, 0x45, 0x01};

/**
 * 取消加粗模式
 */
public static final byte[] BOLD_CANCEL = {0x1b, 0x45, 0x00};

/**
 * 宽高加倍
 */
public static final byte[] DOUBLE_HEIGHT_WIDTH = {0x1d, 0x21, 0x11};

/**
 * 宽加倍
 */
public static final byte[] DOUBLE_WIDTH = {0x1d, 0x21, 0x10};

/**
 * 高加倍
 */
public static final byte[] DOUBLE_HEIGHT = {0x1d, 0x21, 0x01};

/**
 * 字体不放大
 */
public static final byte[] NORMAL = {0x1d, 0x21, 0x00};

/**
 * 设置默认行间距
 */
public static final byte[] LINE_SPACING_DEFAULT = {0x1b, 0x32};


```

打印实现
----

打印小票，当然首先需要连接蓝牙打印机。至于如何扫描打印机，如何连接，这个都是标准的蓝牙方式，网上资料也很多。因为本博文主要关注打印格式，所以这个就不再赘述了。连接打印机后，需要从`BluetoothSocket`中获取`OutputStream`。然后接下来都是通过`OutputStream`来给打印机发送打印指令。

*   **设置打印格式**  
    设置打印格式，就要用到上面封装的那些指令了。

```
    /**
     * 设置打印格式
     *
     * @param command 格式指令
     */
    public static void selectCommand(byte[] command) {
        try {
            outputStream.write(command);
            outputStream.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


```

用法如下：

```
  PrintUtils.selectCommand(PrintUtils.RESET);
  PrintUtils.selectCommand(PrintUtils.LINE_SPACING_DEFAULT);
  PrintUtils.selectCommand(PrintUtils.ALIGN_CENTER);
  PrintUtils.selectCommand(PrintUtils.NORMAL);


```

*   **打印文字**

```
    /**
     * 打印文字
     *
     * @param text 要打印的文字
     */
    public static void printText(String text) {
        try {
            byte[] data = text.getBytes("gbk");
            outputStream.write(data, 0, data.length);
            outputStream.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


```

用法如下：

```
  PrintUtils.printText("好吃的牛肉面" + "\n");


```

打印文字的时候，最后都要手动拼接一个 "\n" 用来换行。

完美吗？
----

根据上面封装的代码，“貌似” 是可以实现所有的打印样式了。是的，没毛病。因为上面既有打印格式的设置，又有打印文字的用法。打印小票是没问题了。but……

这种格式如何实现？

![](http://upload-images.jianshu.io/upload_images/1467310-c7f80d947c121565.png) Paste_Image.png

这种格式呢？

![](http://upload-images.jianshu.io/upload_images/1467310-ce428c4c1b543c66.png) Paste_Image.png

有的盆友可能会说，这有啥问题的？？？ 并且给出了他们认为完美的解释：

```
  PrintUtils.printText("合计                           53.50" + "\n");
  PrintUtils.printText("抹零                            3.50" + "\n");
  PrintUtils.printText("项目            数量            金额" + "\n");


```

可是，完美吗？  
你可能觉得人工加空格是可以 “实现” 需求。but…… 中间的空格，你知道应该添加多少吗？添加多了或者少了，打印出来的结果都会一塌糊涂！并且注意小票上都是要求对齐的！合计、抹零左侧对齐。金额右侧对齐。项目、数量、金额这三列都要中心对齐。。看到这里，这个人工加空格的做法，还完美吗？

给我一个完美的解释！
----------

> “海参炒面，海参呢？给我一个完美的解释！”  
> “我叫海参，面是我炒的。完美不？”

是的，我们需要一个完美的解释。到底如何实现上面说的`打印两列`、`打印三列`的情况。  
首先，讲解之前，先设置几个默认值：

```
    /**
     * 打印纸一行最大的字节
     */
    private static final int LINE_BYTE_SIZE = 32;

    /**
     * 打印三列时，中间一列的中心线距离打印纸左侧的距离
     */
    private static final int LEFT_LENGTH = 16;

    /**
     * 打印三列时，中间一列的中心线距离打印纸右侧的距离
     */
    private static final int RIGHT_LENGTH = 16;

    /**
     * 打印三列时，第一列汉字最多显示几个文字
     */
    private static final int LEFT_TEXT_MAX_LENGTH = 5;


```

我们知道，通用的打印纸都是有固定宽度的。经过大量测试，得出打印纸一行的最大字节数是 32 个字节。那么根据上面的注释，我们可以得到以下结论：

```
LEFT_LENGTH + RIGHT_LENGTH = LINE_BYTE_SIZE


```

这是毋庸置疑的。左侧宽度 + 右侧宽度 必须要等于打印纸总宽度。  
而且因为打印三列的时候，中间一列是要居中显示的，所以`LEFT_LENGTH`和`RIGHT_LENGTH`都必须是总宽度 32 的一半，也就是必须是 16.

那么如何计算某个文字所占的字节数呢？

```
    /**
     * 获取数据长度
     *
     * @param msg
     * @return
     */
    @SuppressLint("NewApi")
    private static int getBytesLength(String msg) {
        return msg.getBytes(Charset.forName("GB2312")).length;
    }


```

OK，准备了这么多，海参终于准备好了。接下来就可以准备炒面了~

#### 打印两列

```
    /**
     * 打印两列
     *
     * @param leftText  左侧文字
     * @param rightText 右侧文字
     * @return
     */
    @SuppressLint("NewApi")
    public static String printTwoData(String leftText, String rightText) {
        StringBuilder sb = new StringBuilder();
        int leftTextLength = getBytesLength(leftText);
        int rightTextLength = getBytesLength(rightText);
        sb.append(leftText);

        // 计算两侧文字中间的空格
        int marginBetweenMiddleAndRight = LINE_BYTE_SIZE - leftTextLength - rightTextLength;

        for (int i = 0; i < marginBetweenMiddleAndRight; i++) {
            sb.append(" ");
        }
        sb.append(rightText);
        return sb.toString();
    }


```

那位说话了：“你这代码明明也是手动拼的空格啊，完美个毛啊！”。大兄弟你消消气，这里是通过逻辑进行拼接的空格，不是无脑的拼接。打印两列的步骤如下：

*   **拼接左侧一列的文字**
*   **拼接两侧文字中间的空格**
*   **拼接右侧一列的文字**

关键步骤是计算两侧文字中间的空格。怎么计算呢？很简单，`总宽度 - 左侧文字长度 - 右侧文字长度` 就是空格的长度。

#### 打印三列

```
    /**
     * 打印三列
     *
     * @param leftText   左侧文字
     * @param middleText 中间文字
     * @param rightText  右侧文字
     * @return
     */
    @SuppressLint("NewApi")
    public static String printThreeData(String leftText, String middleText, String rightText) {
        StringBuilder sb = new StringBuilder();
        // 左边最多显示 LEFT_TEXT_MAX_LENGTH 个汉字 + 两个点
        if (leftText.length() > LEFT_TEXT_MAX_LENGTH) {
            leftText = leftText.substring(0, LEFT_TEXT_MAX_LENGTH) + "..";
        }
        int leftTextLength = getBytesLength(leftText);
        int middleTextLength = getBytesLength(middleText);
        int rightTextLength = getBytesLength(rightText);

        sb.append(leftText);
        // 计算左侧文字和中间文字的空格长度
        int marginBetweenLeftAndMiddle = LEFT_LENGTH - leftTextLength - middleTextLength / 2;

        for (int i = 0; i < marginBetweenLeftAndMiddle; i++) {
            sb.append(" ");
        }
        sb.append(middleText);

        // 计算右侧文字和中间文字的空格长度
        int marginBetweenMiddleAndRight = RIGHT_LENGTH - middleTextLength / 2 - rightTextLength;

        for (int i = 0; i < marginBetweenMiddleAndRight; i++) {
            sb.append(" ");
        }

        // 打印的时候发现，最右边的文字总是偏右一个字符，所以需要删除一个空格
        sb.delete(sb.length() - 1, sb.length()).append(rightText);
        return sb.toString();
    }


```

打印三列的步骤如下：

*   **拼接左侧一列的文字**
*   **拼接左侧文字和中间文字中间的空格**
*   **拼接中间的文字**
*   **拼接右侧文字和中间文字中间的空格**
*   **拼接右侧一列的文字**

在计算空格的时候，为了保证中间一列始终保持中心线对齐，所以在计算中间文字长度时候，都除以 2。

完整打印代码
------

```
PrintUtils.selectCommand(PrintUtils.RESET);
PrintUtils.selectCommand(PrintUtils.LINE_SPACING_DEFAULT);
PrintUtils.selectCommand(PrintUtils.ALIGN_CENTER);
PrintUtils.printText("美食餐厅\n\n");
PrintUtils.selectCommand(PrintUtils.DOUBLE_HEIGHT_WIDTH);
PrintUtils.printText("桌号：1号桌\n\n");
PrintUtils.selectCommand(PrintUtils.NORMAL);
PrintUtils.selectCommand(PrintUtils.ALIGN_LEFT);
PrintUtils.printText(PrintUtils.printTwoData("订单编号", "201507161515\n"));
PrintUtils.printText(PrintUtils.printTwoData("点菜时间", "2016-02-16 10:46\n"));
PrintUtils.printText(PrintUtils.printTwoData("上菜时间", "2016-02-16 11:46\n"));
PrintUtils.printText(PrintUtils.printTwoData("人数：2人", "收银员：张三\n"));

PrintUtils.printText("--------------------------------\n");
PrintUtils.selectCommand(PrintUtils.BOLD);
PrintUtils.printText(PrintUtils.printThreeData("项目", "数量", "金额\n"));
PrintUtils.printText("--------------------------------\n");
PrintUtils.selectCommand(PrintUtils.BOLD_CANCEL);
PrintUtils.printText(PrintUtils.printThreeData("面", "1", "0.00\n"));
PrintUtils.printText(PrintUtils.printThreeData("米饭", "1", "6.00\n"));
PrintUtils.printText(PrintUtils.printThreeData("铁板烧", "1", "26.00\n"));
PrintUtils.printText(PrintUtils.printThreeData("一个测试", "1", "226.00\n"));
PrintUtils.printText(PrintUtils.printThreeData("牛肉面啊啊", "1", "2226.00\n"));
PrintUtils.printText(PrintUtils.printThreeData("牛肉面啊啊啊牛肉面啊啊啊", "888", "98886.00\n"));

PrintUtils.printText("--------------------------------\n");
PrintUtils.printText(PrintUtils.printTwoData("合计", "53.50\n"));
PrintUtils.printText(PrintUtils.printTwoData("抹零", "3.50\n"));
PrintUtils.printText("--------------------------------\n");
PrintUtils.printText(PrintUtils.printTwoData("应收", "50.00\n"));
PrintUtils.printText("--------------------------------\n");

PrintUtils.selectCommand(PrintUtils.ALIGN_LEFT);
PrintUtils.printText("备注：不要辣、不要香菜");
PrintUtils.printText("\n\n\n\n\n");


```

举一反三
----

![](http://upload-images.jianshu.io/upload_images/1467310-d853b14c0325f2f4.png) Paste_Image.png

学习了上面的打印格式，那么这个小票怎么打印？ 区别就是打印三列的时候，中间一列是偏右了。相信大家应该知道答案了。如果有疑问，可以给我留言。

鉴于好多读者给我留言，要 PrintUtils 工具类代码，所以我把代码发布到 github 上了，大家可以自行下载。地址是：[https://github.com/heroxuetao/PrintUtils](https://link.jianshu.com?t=https://github.com/heroxuetao/PrintUtils)

##### 如果有帮助到你，可以顺手来个 star 。万分感谢！

本文止。
