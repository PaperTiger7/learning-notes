## Cesium学习笔记

#### 一、Cesium简介

- 展示三维地球和地图的javascript库
- 使用时不需要任何插件支持，但是浏览器必须支持WebGL
- 学习路线

![图片](https://img-blog.csdnimg.cn/img_convert/025b46abc4494997648a23cfb34e9b17.png)

#### 二、使用说明

- 源码下载https://cesium.com/downloads/

- ```
  npm install cesium
  ```

#### 三、源码目录结构

##### 1、根路径

**CHANGES.md**： Cesium每个版本的变更记录以及每个版本修复了哪些功能

##### 2、Apps文件夹

**CesiumViewer**：一个简单的Cesium初始化示例。

**SampleData**：所有示例代码所用到的数据，包括json、geoJson、topojson、kml、czml、gltf、3dtiles以及图片等。

**Sandcastle**：Ceisum的示例程序代码。

**TimelineDemo**：时间轴示例代码。

#### 四、Viewer界面

任何[Cesium](https://so.csdn.net/so/search?q=Cesium&spm=1001.2101.3001.7020)应用程序的基础都是Viewer，Viewer是一个带有多种功能的可交互的三位数字地球的容器。以下代码初始化了一个视图窗口，看到了一个基本的数字地球

```
var viewer = new Cesium.Viewer("cesiumContainer");
```

![image-20220907162337768](C:\Users\wb\AppData\Roaming\Typora\typora-user-images\image-20220907162337768.png)

1. Geocoder：查找位置工具，查找到之后会将镜头对准找到的地址，默认使用微软的Bing地图
2. HomeButton：首页位置，点击之后将视图跳转到默认全球视角
3. SceneModePicker：选择视角的模式，3D，2D，哥伦布视图(CV)
4. BaseLayerPicker：图层选择器，选择要显示的地图服务和地形服务
5. NavigationHelpButton：导航帮助按钮，显示默认的地图控制帮助
6. Animation：动画器件，控制视图动画的播放速度
7. CreditsDisplay：展示商标版权和数据归属
8. Timeline：时间轴，指示当前时间，并允许用户跳到特定的时间
9. FullscreenButton：全屏按钮

- js控制组件隐藏

```javascript
var viewer = new Cesium.Viewer("cesiumContainer", {
      animation: false, // 动画小组件
      baseLayerPicker: false, // 底图组件，选择三维数字地球的底图（imagery and terrain）。
      fullscreenButton: false, // 全屏组件
      vrButton: false, // VR模式
      geocoder: false, // 地理编码（搜索）组件
      homeButton: false, // 首页，点击之后将视图跳转到默认视角
      infoBox: false, // 信息框
      sceneModePicker: false, // 场景模式，切换2D、3D 和 Columbus View (CV) 模式。
      selectionIndicator: false, //是否显示选取指示器组件
      timeline: false, // 时间轴
      navigationHelpButton: false, // 帮助提示，如何操作数字地球。
      // 如果最初应该看到导航说明，则为true；如果直到用户明确单击该按钮，则该提示不显示，否则为false。
      navigationInstructionsInitiallyVisible: false,
    });

    // 隐藏logo
    viewer._cesiumWidget._creditContainer.style.display = "none";
```

- css控制组件隐藏

```css
/* 通过CSS控制组件显隐及位置 */
    .cesium-viewer-toolbar,             /* 右上角按钮组 */
    .cesium-viewer-animationContainer,  /* 左下角动画控件 */
    .cesium-viewer-timelineContainer,   /* 时间线 */
    .cesium-viewer-bottom               /* logo信息 */ {
      display: none !important;
    }

    .cesium-widget-credits  /* 隐藏logo图片 */ {
      display: none !important;
    }

    .cesium-viewer-fullscreenContainer  /* 全屏按钮 */ {
      display: none !important;
      position: absolute;
      top: 0;
    }
```

#### 五、API简介

- Viewer类属性

imageryLayers 影像数据
terrainProvider 地形数据
dataSources 矢量数据
entities 几何实体集合（用于空间数据可视化）
Widgets 组件，即Viewer初始化界面上的组件
Camera 相机
Event 事件，鼠标事件、实体选中事件等

- Scene类属性

primitives 图元集合（几何体和外观）
postProcessStages 场景后期处理
环境对象，大气圈、天空盒、太阳、月亮等
Event事件，更新、渲染事件等
Camera类属性
位置、方位角、俯仰角、翻滚角

#### 六、坐标系与坐标变换

##### 1、屏幕坐标

二维笛卡尔平面坐标：鼠标点击直接获取的坐标就是屏幕坐标，单位是像素值

```
new Cesium.Cartesian2(x, y)
```

笛卡尔空间直角坐标又称为世界坐标：用来做空间位置的变化如平移、旋转和缩放

```
new Cesium.Cartesian3(x, y, z)
```

##### 2、地理坐标

Cesium中的地理坐标单位默认是弧度制，用Cartographic变量表示，参数是用弧度表示的经纬度，即经度和纬度

```
new Cesium.Cartographic(longitude, latitude, height)
```

##### 3、经纬度坐标

默认是WGS84坐标系，坐标原点在椭球的质心

Cesuim中没有具体的经纬度对象，要得到经纬度首先需要计算为弧度，再进行转换。Cesium提供了如下对应的转换方法：

```javascript
// 经纬度转弧度
Cesium.Math.toRadians(degrees)
// 弧度转经纬度
Cesium.Math.toDegrees(radians)
```

##### 4、坐标变换

###### 4.1 经纬度坐标转世界坐标

```javascript
// 方法1：直接转换
var cartesian3 = Cesium.Cartesian3.fromDegrees(lng, lat, height);

// 方法2：借助ellipsoid对象，先转换成弧度再转换
var cartographic = Cesium.Cartographic.fromDegrees(lng, lat, height); 
//单位：度，度，米
var cartesian3 = ellipsoid.cartographicToCartesian(cartographic);
```

###### 4.2 世界坐标转经纬度

```javascript
// 3.笛卡尔空间直角坐标系转为地理坐标（弧度制）
// var cartographic = Cesium.Cartographic.fromCartesian(cartesian3); // 方法1
var cartographic = ellipsoid.cartesianToCartographic(cartesian3); // 方法2
// 4.地理坐标（弧度制）转为经纬度坐标
var lat = Cesium.Math.toDegrees(cartographic.latitude);
var lng = Cesium.Math.toDegrees(cartographic.longitude);
var height = cartographic.height;
```

###### 4.3 屏幕坐标和世界坐标互转

```javascript
// 二维屏幕坐标转为三维笛卡尔空间直角坐标（世界坐标）
var cartesian3 = scene.globe.pick(
	viewer.camera.getPickRay(windowPostion),
	scene
);
 // 三维笛卡尔空间直角坐标（世界坐标）转为二维屏幕坐标
 // 结果是Cartesian2对象，取出X,Y即为屏幕坐标。
 windowPostion = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
	scene,
	cartesian3
 );
```

#### 七、影像数据加载

- ImageryLayer类

Cesium.ImageryLayer类用于表示Cesium中的影像图层，它需要数据源（imageryProvider）为其提供内在丰富的地理空间信息和属性信息。同时，通过该类还能设置影像图层相关属性，比如透明度、亮度、对比度、色调等。

- ImageryProvider类

Cesium.ImageryProvider类及其子类封装了加载各种影像图层的方法，其中Cesium.ImageryProvider类是抽象类、基类或者可将其理解为接口，它不能被直接实例化。我们可以把ImageryProvider看作是影像图层的数据源（包裹在ImageryLayer类内部），我们想使用哪种影像图层数据或服务就用对应的ImageryProvider子类去加载，目前Cesium提供了以下14种ImageryProvider。

- ImageryLayerCollection类

Cesium.ImageryLayerCollection类是ImageryLayer类对象的容器，它可以装载、放置多个ImageryLayer或ImageryProvider类对象，而且它内部放置的ImageryLayer或ImageryProvider类对象是有序的。
Cesium.Viewer类对象中包含的imageryLayers属性就是ImageryLayerCollection类的实例，它包含了当前Cesium应用程序所有的ImageryLayer类对象，即所有影像图层，所以Cesium种的影像图层可以添加多个。

- Cesium加载不同类型的影像图层

1. ArcGisMapServerImageryProvider——支持ArcGIS Online和Server的相关服务

   ```javascript
   var arcgisProvider = new Cesium.ArcGisMapServerImageryProvider({  		   url:"https://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer",
   });
   imageryLayers.addImageryProvider(arcgisProvider);
   ```

2. SingleTileImageryProvider——单张图片的影像服务，适合离线数据或对影像数据要求并不高的场景下

   ```javascript
    var imagelayer = new Cesium.SingleTileImageryProvider({
         url: "./images/worldimage.jpg",
       });
       imageryLayers.addImageryProvider(imagelayer);
   ```

3. UrlTemplateImageryProvider——指定url的format模版，方便用户实现自己的Provider.比如国内的高德，腾讯等影像服务，url都是一个固定的规范，都可以通过该Provider轻松实现。而OSM也是通过该类实现的，以下是使用XYZ方式加载上面加载过的OSM影像服务

   ```javascript
   const osmImageryProvider = new Cesium.UrlTemplateImageryProvider({
         url: "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
         subdomains: ["a", "b", "c"],
       });
       imageryLayers.addImageryProvider(osmImageryProvider);
   ```

4. WebMapServiceImageryProvider——符合WMS规范的影像服务都可以通过该类封装，指定具体参数实现

   ```javascript
   var provider = new Cesium.WebMapServiceImageryProvider({
         url: "https://nationalmap.gov.au/proxy/http://geoserver.nationalmap.nicta.com.au/geotopo_250k/ows",
         layers: "Hydrography:bores",
         parameters: {
           transparent: true,
           format: "image/png",
         },
       });
       imageryLayers.addImageryProvider(provider);
   ```

5. WebMapTileServiceImageryProvider——服务WMTS1.0.0规范的影像服务，都可以通过该类实现，比如国内的天地图

   ```javascript
   var shadedRelief1 = new Cesium.WebMapTileServiceImageryProvider({  url:"https://services.arcgisonline.com/arcgis/rest/services/World_Imagery/MapServer/WMTS",
         layer: "World_Imagery",
         style: "default",
         format: "image/jpeg",
         tileMatrixSetID: "default028mm",
         maximumLevel: 23,
       });
       imageryLayers.addImageryProvider(shadedRelief1);
   ```

#### 八、矢量数据加载

矢量数据（Vector Data）是用X、Y、Z坐标表示地图图形或地理实体位置的数据，一般是通过记录坐标的方式来尽可能将地理实体的空间位置表现的准确无误，常见的矢量数据有：点、线、面等格式。

目前最常见的矢量数据格式就是shapfile（简称shp）

Cesium直接支持的矢量数据格式包括：geojson、topojson、kml以及具有时间特性的czml，并以DataSouce后缀去命名相关的类

##### 1、geojson加载

GeoJSON的最外层是一个单独的对象（object）。这个对象可表示：

- 几何体（Geometry）
- 特征（Feature）
- 特征集合（FeatureCollection）

加载示例：

```javascript
var viewer = new Cesium.Viewer("cesiumContainer");                   
viewer.dataSources.add(
  Cesium.GeoJsonDataSource.load(
    "../../SampleData/ne_10m_us_states.topojson",
    {
      stroke: Cesium.Color.HOTPINK,
      fill: Cesium.Color.PINK.withAlpha(0.5),
      strokeWidth: 3,
    }
  )
);
```

#### 九、空间数据可视化

两种API：

1、低级（原始）API，通过Primitive类实现 

2、数据驱动的高级（实体）API，通过Entity类实现

![image-20220908165127148](C:\Users\wb\AppData\Roaming\Typora\typora-user-images\image-20220908165127148.png)

广告牌billboard使用示例：

```javascript
var entity = viewer.entities.add({
      name: "billboard",
      position: Cesium.Cartesian3.fromDegrees(-75.59777, 40.03883),
      billboard: {
        show: true, 
        image: "./images/Cesium_Logo_overlay.png", 
        scale: 2.0, 
        pixelOffset: new Cesium.Cartesian2(0, -50),
        eyeOffset: new Cesium.Cartesian3(0.0, 0.0, 0.0),
        horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM, 
        heightReference: Cesium.HeightReference.CLAMP_TO_GROUND,
        color: Cesium.Color.LIME,
        rotation: Cesium.Math.PI_OVER_FOUR, 
        alignedAxis: Cesium.Cartesian3.ZERO,
        width: 100, 
        height: 25, 
        scaleByDistance: new Cesium.NearFarScalar(1.0e3, 2.0, 2.0e3, 1.0), 
        translucencyByDistance: new Cesium.NearFarScalar(
          1.0e3,
          1.0,
          1.5e6,
          0.5
        ),
        pixelOffsetScaleByDistance: new Cesium.NearFarScalar(
          1.0e3,
          1.0,
          1.5e6,
          0.0
        ),
        disableDepthTestDistance: Number.POSITIVE_INFINITY,
      },
    });
```

其余示例见：https://blog.csdn.net/ls870061011/article/details/122724225

#### 十、组件重写

- **homeButton组件**

点击homeButton时，相机不是定位到Cesium自带的默认位置，而是定位到我们想要的位置

1、修改相机默认矩形范围

```javascript
Cesium.Camera.DEFAULT_VIEW_RECTANGLE = Cesium.Rectangle.fromDegrees(
      110.15,
      34.54,
      110.25,
      34.56
    ); //Rectangle(west, south, east, north)
```

2、homeButton 的 viewModel 中添加监听事件

```javascript
if (viewer.homeButton) {
      viewer.homeButton.viewModel.command.beforeExecute.addEventListener(
        function (e) {
          e.cancel = true;
          //你要飞的位置
          viewer.camera.flyTo({
            destination: Cesium.Cartesian3.fromDegrees(117.16, 32.71, 15000.0),
          });
        }
      );
    }
```

#### 十一、事件应用

##### 1、鼠标操作ScreenSpaceEventHandler 类

（1）viewer.canvas参数实例化ScreenSpaceEventHandler类

```javascript
var handler = new Cesium.ScreenSpaceEventHandler(viewer.canvas);
	// 事件类型：单击鼠标左键
    let eventType = Cesium.ScreenSpaceEventType.LEFT_CLICK;
    // 注册事件
    handler.setInputAction((event) => {
      console.log(event);
    }, eventType);

    // 注销事件
    handler.removeInputAction(eventType);
```

（2）为 handler 注册鼠标事件的监听

（3）在监听事件的回调方法中获取 event.position，并将其作为参数执行scene.pick 方法获取对应的选中对象

```javascript
var picked = viewer.scene.pick(event.position);
if (Cesium.defined(picked)) {
        if (picked.id && picked.id instanceof Cesium.Entity) {
          console.log("选中了Entity");
        }
        if (picked.primitive instanceof Cesium.Primitive) {
          console.log("选中了Primitive");
        }
        if (picked.primitive instanceof Cesium.Model) {
          console.log("选中了模型");
        }
        if (picked instanceof Cesium.Cesium3DTileFeature) {
          console.log("选中了3DTile");
        }
}
```

简洁的Entity选择，selectedEntityChanged（viewer类事件类型的属性）来帮助我们获取选中的Entity，通过这个属性，无需再写注册鼠标事件

```javascript
viewer.selectedEntityChanged.addEventListener(function (entity) {
      console.log(entity.id);
    });
```

##### 2、通用事件

##### 3、相机控制screenSpaceCameraController类

Cesium在Viewer类实例化过程中，也实例化其他很多类，其中包括ScreenSpaceCameraController类，并把实例化结果赋值给了viewer.scene.screenSpaceCameraController。所以，直接操作viewer.scene.screenSpaceCameraController就可以

#### 十二、相机控制

相机控制主要是用于相机的飞行定位，例如系统初始化位置定位、视点切换、设备定位、报警事件定位等，这些都是通过对相机进行操作实现的。Cesium虽然提供了很多种方法用于实现相机的飞行定位，但这些方法都是基于Viewer、Camera这两个类实现的。
