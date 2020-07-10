## 常用材质介绍

> Threejs提供了一系列的材质，所有材质就是对WebGL着色器代码的封装

### 点材质`PointsMaterial`

只有`PointsMaterial`,通常使用点模型的时候会使用点材质`PointsMaterial`。

点材质`PointsMaterial`的`.size`属性可以每个顶点渲染的方形区域尺寸像素大小。

```
var geometry = new THREE.SphereGeometry(100, 25, 25); //创建一个球体几何对象
var material = new THREE.PointsMaterial({// 创建一个点材质对象
  color: 0x0000ff, //颜色
  size: 3, //点渲染尺寸
});
//点模型对象  参数：几何体  点材质
var point = new THREE.Points(geometry, material);
scene.add(point); //网格模型添加到场景中
```

### 线材质

线材质有基础线材质`LineBasicMaterial`和虚线材质`LineDashedMaterial`两个，通常使用`Line`等线模型才会用到线材质。

**基础线材质`LineBasicMaterial`。**

```
var geometry = new THREE.SphereGeometry(100, 25, 25);//球体
var material = new THREE.LineBasicMaterial({// 直线基础材质对象
  color: 0x0000ff
});
var line = new THREE.Line(geometry, material); //线模型对象
scene.add(line); //点模型添加到场景中
```

**虚线材质`LineDashedMaterial`。**

```
// 虚线材质对象：产生虚线效果
var material = new THREE.LineDashedMaterial({
  color: 0x0000ff,
  dashSize: 10,//显示线段的大小。默认为3。
  gapSize: 5,//间隙的大小。默认为1
});
var line = new THREE.Line(geometry, material); //线模型对象
//  computeLineDistances方法  计算LineDashedMaterial所需的距离数组
line.computeLineDistances();
```

### 网格材质

网格材质是网格类模型才会使用的材质对象。

**基础网格材质对象**

`MeshBasicMaterial`,不受带有方向光源影响，没有棱角感。

```
var material = new THREE.MeshBasicMaterial({
  color: 0x0000ff,
})
```

`MeshLambertMaterial`材质可以实现网格Mesh表面与光源的漫反射光照计算，有了光照计算，物体表面分界的位置才会产生棱角感。

```
var material = new THREE.MeshLambertMaterial({
  color: 0x00ff00,
});
```

高光网格材质`MeshPhongMaterial`除了和`MeshLambertMaterial`一样可以实现光源和网格表面的漫反射光照计算，还可以产生高光效果(镜面反射)。

```
var material = new THREE.MeshPhongMaterial({
  color: 0xff0000,
  specular:0x444444,//高光部分的颜色
  shininess:20,//高光部分的亮度，默认30
});
```

## 材质共有属性、私有属性

点材质`PointsMaterial`、基础线材质`LineBasicMaterial`、基础网格材质`MeshBasicMaterial`、高光网格材质`MeshPhongMaterial`等材质都是父类[Material](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/Material)的子类。

所有子类的材质都会从父类材质[Material](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/Material)继承透明度`opacity`、面`side`等属性，这些来自父类的属性都是子类共有的属性。

### `.side`属性

`.side`属性的属性值定义面的渲染方式前面后面 或 双面. 属性的默认值是`THREE.FrontSide`，表示前面. 也可以设置为后面`THREE.BackSide` 或 双面`THREE.DoubleSide`.

```
var material = new THREE.MeshBasicMaterial({
  color: 0xdd00ff,
  // 前面FrontSide  背面：BackSide 双面：DoubleSide
  side:THREE.DoubleSide,
});
```

## 材质透明度`.opacity`

通过材质的透明度属性`.opacity`可以设置材质的透明程度，`.opacity`属性值的范围是0.0~1.0，`.opacity`默认值1.0（完全不透明）。

当设置`.opacity`属性值的时候，需要设置材质属性`transparent`值为`true`，如果材质的transparent属性没设置为true, 材质会保持完全不透明状态。

在构造函数参数中设置`transparent`和`.opacity`的属性值

```
var material = new THREE.MeshPhongMaterial({
  color: 0x220000,
  transparent: true,// 开启透明
  opacity: 0.4,// 设置材质透明度
});
```

## 点、线、网格模型

点模型`Points`、线模型`Line`、网格网格模型`Mesh`都是由几何体Geometry和材质Material构成，这三种模型的区别在于对几何体顶点数据的渲染方式不同。

### 点模型`Points`

点模型`Points`就是几何体的每一个顶点数据渲染为一个方形区域，方形区域的大小可以设置。

```
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 点渲染模式
var material = new THREE.PointsMaterial({
  color: 0xff0000,
  size: 5.0 //点对象像素尺寸
}); //材质对象
var points = new THREE.Points(geometry, material); //点模型对象
```

### 线模型`Line`

两点确定一条直线，线模型`Line`就是使用线条去连接几何体的顶点数据。

```
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 线条渲染模式
var material=new THREE.LineBasicMaterial({
    color:0xff0000 //线条颜色
});//材质对象
// 创建线模型对象   构造函数：Line、LineLoop、LineSegments
var line=new THREE.Line(geometry,material);//线条模型对象
```

### 网格模型`Mesh`

三个顶点确定一个三角形，网格模型[Mesh](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Mesh)默认的情况下，通过三角形面绘制渲染几何体的所有顶点，通过一系列的三角形拼接出来一个曲面。

```
var geometry = new THREE.BoxGeometry(100, 100, 100);
// 三角形面渲染模式  
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff, //三角面颜色
}); //材质对象
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
```

## 模型对象旋转平移缩放变换

点模型`Points`、线模型`Line`、网格网格模型`Mesh`等模型对象的基类都是[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)

### 缩放

网格模型`Mesh`的属性`.scale`表示模型对象的缩放比例，默认值是`THREE.Vector3(1.0,1.0,1.0)`

网格模型xyz方向分别缩放0.5, 1.5, 2倍

```
mesh.scale.set(0.5, 1.5, 2)
```

x轴方向放大2倍

```
mesh.scale.x = 2.0;
```

### 位置属性`.position`

模型通过`.position`可以设置在场景Scene中的位置。`.position`的默认值是`THREE.Vector3(0.0,0.0,0.0)`

设置网格模型y坐标

```
mesh.position.y = 80;
```

设置模型xyz坐标

```
mesh.position.set(80,2,10);
```

### 平移

网格模型沿着x轴正方向平移100，可以多次执行该语句，每次执行都是相对上一次的位置进行平移变换。

```
// 等价于mesh.position = mesh.position + 100;
mesh.translateX(100); //沿着x轴正方向平移距离100
```

沿着Z轴负方向平移距离50。

```
mesh.translateZ(-50);
```

### 旋转

立方体网格模型绕立方体的x轴旋转π/4，可以多次执行该语句，每次执行都是相对上一次的角度进行旋转变化

```
mesh.rotateX(Math.PI/4);//绕x轴旋转π/4
```

网格模型绕(0,1,0)向量表示的轴旋转π/8

```
var axis = new THREE.Vector3(0,1,0);//向量axis
mesh.rotateOnAxis(axis,Math.PI/8);//绕axis轴旋转π/8
```

执行旋转`.rotateX()`等方法和执行平移`.translateY()`等方法一样都是对模型状态属性的改变，区别在于执行平移方法改变的是模型的位置属性`.position`

```
// 绕着Y轴旋转90度
mesh.rotateY(Math.PI / 2);
//控制台查看：旋转方法，改变了rotation属性
console.log(mesh.rotation);
```

## 对象克隆`.clone()`和复制`.copy()`

Threejs大多数对象都有克隆`.clone()`和复制`.copy()`两个方法

### 复制方法`.copy()`

`A.copy(B)`表示B属性的值赋值给A对应属性。

```
var p1 = new THREE.Vector3(1.2,2.6,3.2);
var p2 = new THREE.Vector3(0.0,0.0,0.0);
p2.copy(p1)
// p2向量的xyz变为p1的xyz值
console.log(p2);
```

### 克隆方法`.clone()`

`N = M.copy()`表示返回一个和M相同的对象赋值给N。

```
var p1 = new THREE.Vector3(1.2,2.6,3.2);
var p2 = p1.clone();
// p2对象和p1对象xyz属性相同
console.log(p2);
```

### 网格模型复制和克隆

```
var box=new THREE.BoxGeometry(10,10,10);//创建一个立方体几何对象
var material=new THREE.MeshLambertMaterial({color:0x0000ff});//材质对象

var mesh=new THREE.Mesh(box,material);//网格模型对象
var mesh2 = mesh.clone();//克隆网格模型
mesh.translateX(20);//网格模型mesh平移

scene.add(mesh,mesh2);//网格模型添加到场景中
```

## Threejs光源对象

[![threejs32light](https://camo.githubusercontent.com/1bb7f534ac21fa1b144e3581b5c869ac83025f5d/687474703a2f2f7777772e776562676c33642e636e2f75706c6f61642f74687265656a7333326c696768742e706e67)](https://camo.githubusercontent.com/1bb7f534ac21fa1b144e3581b5c869ac83025f5d/687474703a2f2f7777772e776562676c33642e636e2f75706c6f61642f74687265656a7333326c696768742e706e67)

### 环境光AmbientLight

环境光是没有特定方向的光源，主要是均匀整体改变Threejs物体表面的明暗效果，这一点和具有方向的光源不同，比如点光源可以让物体表面不同区域明暗程度不同。

```
var ambient = new THREE.AmbientLight(0x444444);
scene.add(ambient);//环境光对象添加到scene场景中
```

### 点光源PointLight

点光源需要设置位置属性`.position`，光源位置不同，物体表面被照亮的面不同，远近不同因为衰减明暗程度不同。

```
//点光源
var point = new THREE.PointLight(0xffffff);
//设置点光源位置，改变光源的位置
point.position.set(400, 200, 300);
scene.add(point);
```

### 平行光DirectionalLight

```
// 平行光
var directionalLight = new THREE.DirectionalLight(0xffffff, 1);
// 设置光源的方向：通过光源position属性和目标指向对象的position属性计算
directionalLight.position.set(80, 100, 50);
// 方向光指向对象网格模型mesh2，可以不设置，默认的位置是0,0,0
directionalLight.target = mesh2;
scene.add(directionalLight);
```

> 平行光如果不设置`.position`和`.target`属性，光线默认从上往下照射，也就是可以认为`(0,1,0)`和`(0,0,0)`两个坐标确定的光线方向。

### 聚光源SpotLight

可以认为是一个沿着特定方会逐渐发散的光源，照射范围在三维空间中构成一个圆锥体。通过属性`.angle`可以设置聚光源发散角度，聚光源照射方向设置和平行光光源一样是通过位置`.position`和目标`.target`两个属性来实现。

```
// 聚光光源
var spotLight = new THREE.SpotLight(0xffffff);
// 设置聚光光源位置
spotLight.position.set(200, 200, 200);
// 聚光灯光源指向网格模型mesh2
spotLight.target = mesh2;
// 设置聚光光源发散角度
spotLight.angle = Math.PI / 6
scene.add(spotLight);//光对象添加到scene场景中
```