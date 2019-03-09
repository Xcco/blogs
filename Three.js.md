# Three.js!

---

## 基础概念

### 渲染器（Renderer）
```
const renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );
```
### 场景（Scene）
```
const scene = new THREE.Scene();
```
### 照相机（Camera）
```
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 );
camera.position.z = 5;
...
renderer.render( scene, camera );
```

#### 正交投影（orthographic projection）


```
THREE.OrthographicCamera(left, right, top, bottom, near, far)
```
 ![OrthographicCamera][1]
 
设置相机
```
var camera = new THREE.OrthographicCamera(-2, 2, 1.5, -1.5, 1, 10);
camera.position.set(0, 0, 5);
scene.add(camera);
```
 - 为了保持照相机的横竖比例，需要保证(right - left)与(top - bottom)的比例与Canvas宽度与高度的比例一致。

设置相机看向原点

```
// 注意输入vector3 而非坐标
camera.lookAt(new THREE.Vector3(0, 0, 0));
```

#### 透视投影照相机（Perspective Camera）

```
THREE.PerspectiveCamera(fov, aspect, near, far)
```

 ![PerspectiveCamera][2]
 

 - fov：视景体竖直方向上的张角
 - aspect：width / height
 - near/far: 照相机到视景体最近、最远的距离

## 几何形状

### 基本几何形状

 -  立方体（CubeGeometry）
```
THREE.CubeGeometry(width, height, depth, widthSegments, heightSegments, depthSegments)
```
 -  平面（PlaneGeometry）
```
THREE.PlaneGeometry(width, height, widthSegments, heightSegments)
```
 -  球体（SphereGeometry）
```
THREE.SphereGeometry(radius, segmentsWidth, segmentsHeight, phiStart, phiLength, thetaStart, thetaLength)
```
 -  圆形（CircleGeometry）
```
THREE.CircleGeometry(radius, segments, thetaStart, thetaLength)
```


## 材质

### 基本材质（BasicMaterial）
```
THREE.MeshBasicMaterial(opt)
```
opt：

 - visible：是否可见，默认为true
 - side：渲染面片正面或是反面，默认为正面THREE.FrontSide，可设置为反面THREE.BackSide，或双面THREE.DoubleSide
 - wireframe：是否渲染线而非面，默认为false color：十六进制RGB颜色，如红色表示为0xff0000
 - map：使用纹理贴图

### Lambert材质（MeshLambertMaterial）

```
Idiffuse = Kd * Id * cos(theta)
```

 - Idiffuse：漫反射光强
 - Kd：物体表面的漫反射属性
 - Id：光强
 - theta：光的入射角弧度


opt：

 - color：用来表现材质对散射光的反射能力，也是最常用来设置材质颜色的属性
 - ambient：对环境光的反射能力
 - emissive：材质的自发光颜色

 
  [1]: http://pnz8aty4g.bkt.clouddn.com/orthographic-camera.jpg
  [2]: http://pnz8aty4g.bkt.clouddn.com/perspective-camera.jpg