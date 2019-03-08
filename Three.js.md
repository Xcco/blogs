# Three.js!

标签（空格分隔）： 未分类

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

 
  [1]: http://pnz8aty4g.bkt.clouddn.com/orthographic-camera.jpg
  [2]: http://pnz8aty4g.bkt.clouddn.com/perspective-camera.jpg