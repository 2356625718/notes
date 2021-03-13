# CSS回顾（一些不常用属性快忘完了都）

## animation

```javascript
animation: move 5s linear 2s infinite alternate Forwards paused;
      /**
      * 1、animation-name
      * 2、-duration
      * 3、-timing-function:速度,ease、ease-in、ease-out、ease-in-out、linear
      * 4、-delay:延迟时间
      * 5、iteration-count:执行次数,infinite为无限次
      * 6、-direction:方向，normal、reverse、alternate、alternate-reverse
      * 7、-fill-mode:保持状态,none,Forwards(保持最后一帧),backwards,both
      * 8、-play-state:paused,running
      **/
```

## 盒子模型

```javascript
w3c标准盒子模型：
盒子 = border + padding + content
盒子高度 = content.height
盒子宽度 = content.weight

IE盒子模型：
盒子 = border + padding + content
盒子高度 = border上下高度 + padding上下高度 + content高度
盒子宽度 = border左右宽度 + padding左右宽度 + content宽度

默认使用的是w3c标准盒模型，如果需要使用IE盒模型，可以设置属性：box-sizing:border-box;
```

## BFC

常见定位方案：1、普通流 2、浮动 3、绝对定位

BFC是普通流定位的一种方案，可以认为BFC是元素节点的一种属性

如何触发BFC：

```javascript
触发BFC（块级格式化上下文）：
    1、根元素（<html>)
    2、浮动元素（float不是none）
    3、绝对定位元素（其position为absolute或fixed）
    4、display为inline-block、table、inline-table、flex、grid（仅列出常见的几种）
    5、overflow不为visible的块元素（常用）
    6、contain值为layout、content、paint的元素
    7、多列容器（元素的column-count和column-width不为auto）
```

BFC作用：

```javascript
1、避免外边距重叠
2、清除浮动
3、避免覆盖浮动元素
```

## IFC

触发IFC：

```javascript
IFC出现（行内格式化上下文）：当一个块级元素内部没有块级元素时，就形成了IFC
```



特性：

```javascript
特性：
    1、宽度不够就换行
    2、在水平方向上，margin、border、padding都会计算，但在垂直方向上border、padding和margin都不会撑开行盒的高度。
    3、垂直方向上，这些盒可能以不同的方式来对齐，可以通过vertical-align来设置，默认对齐为baseline
    4、每一行将生成一个行盒（line box），包括该行所有盒子
    5、当所有盒的总宽度小于行盒的宽度，那么行盒中的水平方向排版由text-align属性来决定。
    6、当所有盒的宽度超过行盒的宽度时，就会换行形成多个行盒。
    7、当一个行内盒超出行盒的宽度时，它会被分割成多个盒放在多个行盒中，如果行内盒不能被分割，那么这个行内盒将溢出行盒
    8、行盒的高度由内部元素高度最高的元素计算出来。
    */
```

## CSS居中

### 行内元素

#### 水平居中

1、text-align

2、父元素width：fit-content适配子元素宽度，然后margin：auto进行居中

#### 垂直居中

1、父元素中line-height=父元素height

### 块级元素

#### 水平居中

1、margin：0 auto；

### 水平垂直居中

1、定位+transform：translate

2、定位+margin：auto（left、right、top、bottom都设置为0，使子元素占满父元素空间）

3、父元素高宽等于子元素高宽，使用padding

4、flex布局：align-items和justify-center都设置为center

5、将子元素转换为行内块元素，先使用text-align：center水平居中，再创建父元素的伪元素，使其转换为行内块元素，令其高=父元素高，设置vertical-align：middle使行盒的居中线对其，然后给子元素设置vertical-align：center