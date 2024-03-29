### 渐变色

#### 线性渐变

`background: liner-gradient(方向, 颜色1, 颜色2)`
默认方向为从上到下

```css
.box {
  display: flex;
  width: 400px;
  height: 400px;
  background: linear-gradient(orange, red);
}
```

![css1-1](..\assets\css1-1.png)

##### 方向设置

to + 方位
`linear-gradient(to top,orange,red);` 从上到下
`linear-gradient(to right,orange,red);` 从左到右
`linear-gradient(to top right,orange,red);` 从右上到左下

```css
.box {
  display: flex;
  width: 400px;
  height: 400px;
  background: linear-gradient(to top right, orange, red);
}
```

![css1-2](..\assets\css1-2.png)

角度表示
默认是从上往下，把中间分界线当成 0deg, 逆时针旋转 90deg 就是向右（从左向右）

- `0deg = to top`
- `45deg = to top right`
- `90deg = to right`
- `145deg = to bottom right`
  ...

##### 颜色设置

三种方式：英文单词，16 进制 (hex)，rgba

#### 径向渐变

`radital-gradient(形状, 大小, 颜色1, 颜色2)`
默认从中间均匀地渐变（椭圆 ellipse）

```css
.box {
  display: flex;
  width: 400px;
  height: 400px;
  background: radial-gradient(ellipse, orange, red);
}
```

![css1-3](..\assets\css1-3.png)

##### 形状设置

`ellipse, circle`

##### 大小设置

- closest-side: 用来设置从中心到最近的边的渐变大小
- closest-corner: 用来设置从中心到最近的角的渐变大小
- farthest-side: 用来设置从中心到最远的边的渐变大小
- farthest-corner:用来设置从中心到最远的角的渐变大小

具体数值用百分比，ex: closest-side at 10% 10%

#### 重复渐变

`background: repeating-linear-gradient(to right,orange, red 100px);`

![css1-4](..\assets\css1-4.png)