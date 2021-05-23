# 思路
1. 设定鼠标左右滑动为x轴，上下滑动为y轴，鼠标滚动为z轴
   x轴表示色度，y轴表示明暗，z轴表示饱和度
   首先实现一个盒子可以根据鼠标的move和滚轴滚动确定坐标
   > 知识点：鼠标滚轮事件
2. x轴是色度，y轴是明亮，z轴是饱和度
    x轴是一个连续渐变的彩虹条
    实现方法：canvas渐变
       确定x,实现一个连续渐变的彩虹条
       将x轴平分为5份，每份平分为255：
       – 255,0,x (x从0到255)
       – x,0,255 (255-0)
       – 0,x,255 (0-255)
       – 0,255,x (255-0)
       – x,255,0 (0-255)
       – 255,x,0 (255-0)
    y轴是明暗度实现


>最近看到一个颜色选择网站，[网站地址点这里]()。觉得很有趣，想自己实现一下，顺便学习一下canvas。
>没有看源码，全部是自己设计的，预计两天完成吧。
# 思路和实现
## 调色盘的实现
使用canvas画一个渐变色画布，其中x轴是色度，y轴是明暗度，官网其实还有一个z轴，是鼠标滚动选择饱和度，这里先不考虑。
1. 首先，先做x轴的色盘，相当于做一个类似ps里的选色的彩虹条，思路是创建canvas的渐变对象，从左到右分为六份，是红黄绿青蓝紫这六种颜色，最后又过渡回到红色做一个，代码如下：
    ```html 
    <canvas class="box" id="show" width="500px" height="500px"></canvas>
    ```
   ```css
    .box {
        position: relative;
        background-color: pink;
        margin: 100px;
    }   
    ```
    ```javascript
    var show = document.getElementById("show");
    if (show.getContext) {//首先判断getcontext方法是否存在，这是一个良好的习惯
            var context = show.getContext('2d');
            var gradientBar = context.createLinearGradient(0, show.width, show.width, show.width);//创建渐变，前两个参数是渐变开始的横纵坐标，后两个参数是渐变结束的横纵坐标
            gradientBar.addColorStop(0, '#f00');
            gradientBar.addColorStop(1 / 6, '#f0f');
            gradientBar.addColorStop(2 / 6, '#00f');
            gradientBar.addColorStop(3 / 6, '#0ff');
            gradientBar.addColorStop(4 / 6, '#0f0');
            gradientBar.addColorStop(5 / 6, '#ff0');
            gradientBar.addColorStop(1, '#f00');

            context.fillStyle = gradientBar;
            context.fillRect(0, 0, show.width, show.width);
    }
    ```
    效果：

    == 这里有个小问题是，我发现在class里给canvas设置宽高没有用，只能在canvas标签里专门设置一下，大家注意一下这个问题 ==

2. 我利用白色过渡到透明再过渡到黑色来做明度的过渡，有其他更好的方法可以在评论区讨论哦
    在js代码中加入：
    ```javascript
        //白色透明黑色，明暗度实现
        var lightToDark = context.createLinearGradient(0, 0, 0, show.width);
        lightToDark.addColorStop(0, "#fff");
        lightToDark.addColorStop(1 / 2, 'rgba(255,255,255,0)');
        lightToDark.addColorStop(1, '#000');

        context.fillStyle = lightToDark;
        context.fillRect(0, 0, show.width, show.width);
    ```
    效果：

## 实现鼠标滑动到不同的位置，获取对应位置的rgb颜色
- 利用addEventListener监听鼠标滑动，并获得鼠标的位置
- 利用canvas的getImageData() 方法获得当前位置的颜色
- 创建一个小方格来跟随选择颜色

```html 
    <canvas class="box" id="show" width="500px" height="500px"></canvas>
    <em id="cur"></em>
```
```css
    .box {
        position: relative;
        background-color: pink;
        margin: 100px;
    }
    /* 选择颜色的小方格 */
    #cur {
        width: 3px;
        height: 3px;
        outline: 2px solid #535353;
        margin-left: -1px;
        margin-top: -1px;
        position: absolute;
    }   
```

```javascript
    show.addEventListener('mouseover', function (e) {
            var ePos = {
                x: e.layerX || e.offsetX,
                y: e.layerY || e.offsetY
            }
            //前两个参数为鼠标的位置，后娘个参数为canvas的宽高
            var imgData = context.getImageData(ePos.x, ePos.y, show.width, show.height).data;
            //可以通过下面的方法获得当前的rgb值
            var red = imgData[0];
            var green = imgData[1];
            var blue = imgData[2]; 
            var rgbStr = "rgb(" + red + "," + green + ',' + blue + ")";
            console.log(rgbStr);
            cover.style.backgroundColor = rgbStr;

            document.onmousemove = function (e) {
                var pos = {
                    x: e.clientX,
                    y: e.clientY
                }
                cur.style.left = pos.x + 'px';
                cur.style.top = pos.y + 'px';
            };
            document.onmouseup = function () {
                document.onmouseup = document.onmousemove = null;
            }

        });
```
效果图：


## 设置一个遮罩，随鼠标改变而改变自己的颜色，并且把对应的十六进制显示在中央 
这部分思路很简单，就是准备一个定位为绝对定位的div作为遮罩，问题在于，当鼠标放到遮罩上时，鼠标作用在遮罩上而不是我们的canvas上，这个问题的解决办法有几种：
1. 可以在鼠标移动到遮罩上时，触发下边元素的事件
```javascript
   (".a").on("mouseover",function(){(".b").on("mouseover",function(){});
```
2. 把元素遮罩设置为子元素，把你想点击的作为父元素。这样使用DOM的冒泡时间,当点击遮罩的时候,就能捕获到事件了。
3. 添加pointer-events: none，你的遮罩盖住了下边的元素，所以你点击的事件其实是作用在了遮罩上面。而pointer-events: none可以穿透遮罩，点击到下面的元素。我刚做的项目用过这个属性，亲测有效。支持ie11及其他浏览器。
4. 增加你要点击页面的z-index值。
   
这里使用最简单的第三种方法：
```html 
    <canvas class="box" id="show" width="500px" height="500px"></canvas>
     <div class="cover" id='cover'></div>
    <em id="cur"></em>
```
```css
    #cover{
        position: absolute;
        left: 100px;
        top: 100px;
        width: 500px;
        height: 500px;
        background-color: antiquewhite;
        /* cover作为遮罩挡住了canvas，使用这个可以穿透遮罩，点击到下面的元素 */
        pointer-events: none;
    } 
```

到现在为止已实现我们能在官网看到的基本的样子啦：

## 每次点击一个配色来生成一个配色表
- 每次点击动态生成DOM节点
   - 遇到一个问题，我在同一个元素（canvas）上绑定了两个事件，一个mouseover事件，一个click事件，发现两个事件发生了冲突，只执行了mouseover事件，点击事件没有被执行。
   - 一开始以为是因为上面的pointer-events: none;的原因，后来发现并不是，以及发现了不需要设置遮罩的pointer-events: none就能获取到canvas的颜色。想了一下原因是，getImageData本来就是根据canvas元素来获取到颜色的，和遮罩无关，因此直接在遮罩上监听mouseover事件即可。
   - 回到正题，查了网上好多解决办法，大多数基于Jquery进行解决的，比如这篇博文[解决click与hover(mouseover)的冲突问题](https://www.cnblogs.com/smedas/p/12614263.html)；其他还有添加事件延迟的方法进行解决[JavaScript 技巧篇-js增加延迟时间解决单击双击事件冲突，双击事件触发单击事件](https://cloud.tencent.com/developer/article/1703302?from=information.detail.js%E4%BA%8B%E4%BB%B6%E5%8F%91%E7%94%9F%E5%86%B2%E7%AA%81)
   - 最后我的解决办法简单粗暴，为遮罩和canvas元素加上一个父级container，在container上监听点击事件，把从canvas上监听到的颜色rgbStr改成全局变量，每次鼠标移动监听到颜色的改变作用到rgbStr上，然后通过点击事件获得当前rgbStr的值。
   - 这样就完美解决了在同一个DOM上两个事件冲突的问题啦！
   - 顺便解释一下为什么不在遮罩和canvas上分别监听mouseover事件和点击事件？
       
       首先，这两个元素是兄弟关系，遮罩在canvase上面，如果我设置遮罩pointer-events: none，那么遮罩无法监听到事件；而如果我在canvas上监听mouseover，那就必须得让遮罩无法监听到事件。因此，如果他们俩是兄弟元素，那么有一个无法监听到事件。解决办法可以是上面提到的把这俩兄弟变成父子，并考虑事件冒泡的关系；或者是监听遮罩上的事件的时候在回调函数里监听canvas元素上的事件，我感觉有点麻烦。。。如果有更好的方法欢迎在评论区讨论。

代码：
```html 
    <div class="row">
        <div class="container">
            <canvas class="box" id="show" width="900px" height="1000px"></canvas>
            <div class="cover" id='cover'></div>
            <em id="cur"></em>
        </div>
    </div>
```

```javascript
           //监听点击事件
        container.addEventListener('click', function (e) {
            console.log("click:" + rgbStr);
            //var div = createDiv(id,rgbStr);
            var div = document.createElement('div');
            var text = document.createElement('h5');
            text.style.marginTop = "200px";
            text.style.textAlign = 'center';
            var hexcolor = colorHex(rgbStr);
            var textNode = document.createTextNode(hexcolor);
            text.appendChild(textNode);
            div.appendChild(text);
            div.style.backgroundColor = rgbStr;
            div.style.width = "100px";
            row.appendChild(div);
            div.style.backgroundColor = rgbStr;
        }, false);

        //十六进制颜色值的正则表达式
        var reg = /^#([0-9a-fA-f]{3}|[0-9a-fA-f]{6})$/;
        //颜色转换
        function colorHex(string) {
            if (/^(rgb|RGB)/.test(string)) {
                var aColor = string.replace(/(?:\(|\)|rgb|RGB)*/g, "").split(",");
                var strHex = "#";
                for (var i = 0; i < aColor.length; i++) {
                    var hex = Number(aColor[i]).toString(16);
                    if (hex === "0") {
                        hex += hex;
                    }
                    strHex += hex;
                }
                if (strHex.length !== 7) {
                    strHex = string;
                }
                return strHex;
            } else if (reg.test(string)) {
                var aNum = string.replace(/#/, "").split("");
                if (aNum.length === 6) {
                    return string;
                } else if (aNum.length === 3) {
                    var numHex = "#";
                    for (var i = 0; i < aNum.length; i += 1) {
                        numHex += (aNum[i] + aNum[i]);
                    }
                    return numHex;
                }
            } else {
                return string;

            }
        }
```

效果图：

## 为每个配色条增加删除按钮
这里考虑了一个问题是，我想让删除按钮的颜色随颜色变化而变化，主要是深色的话按钮和字体为白色，否则相反。
```javascript
    //判断颜色深浅
    //rgb(245,232,11)
    var temp = rgbStr.slice(4, -1).split(',')
    var r = Number(temp[0])
    var g = Number(temp[1])
    var b = Number(temp[2])
    if (r * 0.299 + g * 0.578 + b * 0.114 >= 192) { //浅色
        var textcolor = "#000";
    } else { //深色
        var textcolor = "#fff";
    }
```

点击按钮删除对应的配色条，代码如下：
```javascript
//添加删除按钮
    var delBtn = document.createElement('a');
    delBtn.href = "#";
    delBtn.id = "delBtn";
    delBtn.style.color = textcolor;
    var deltext = document.createTextNode("del");
    delBtn.onclick = function (e) {
        console.log(e.target.parentNode);
        row.removeChild(e.target.parentNode)
    }
    delBtn.appendChild(deltext);
    div.appendChild(delBtn);
```

截止目前效果如图所示，就是一个小小的demo啦，完整代码见github
