# Echarts - 基础实例

[toc]

### 基础配置

#### 全球地图轮廓 - 无数据

**演示效果**

![image-20200611200459791](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611200459791.png)

**Echarts 配置项**

```js
var option = {
    title: {
        text: '全球轮廓演示',
        left: 'center',
    },
    toolbox: {
        show: true,
        feature: {
            saveAsImage: {
                pixelRatio: 4
            }
        }
    },
    series: [{
        type: 'map',
        mapType: ‘world’,
        selectedMode: 'false', //是否允许选中多个区域
        label: {
            normal: {
                show: true
            },
            emphasis: {
                show: true
            }
        },
        data: []
    }]
};
```

**特别说明：**

1. 注册地图的方式：可以采取 JSON 文件引入，也可以采用相对应地图的js文件



#### 全国地图轮廓 - 无数据

**演示效果**

![image-20200611200818504](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611200818504.png)

**Echarts配置项**

```js
var option = {
    title: {
        text: name,
        left: 'center',
    },
    toolbox: {
        show: true,
        feature: {
            saveAsImage: {
                pixelRatio: 4
            }
        }
    },
    series: [{
        type: 'map',
        mapType: ‘china’,
        selectedMode: 'false', //是否允许选中多个区域
        label: {
            normal: {
                show: true
            },
            emphasis: {
                show: true
            }
        },
        data: []
    }]
};
```



#### 各个省份地图轮廓 - 无数据版

**演示效果**

![image-20200611200949110](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611200949110.png)

**Echarts配置项**

```js
loadMap('./lib/map/json/china.json', 'china');

function loadMap(mapCode, name) {
    $.get(mapCode, function(data) {
        if (data) {
            echarts.registerMap(name, data);
            var option = {
                title: {
                    text: name,
                    left: 'center',
                },
                toolbox: {
                    show: true,
                    feature: {
                        saveAsImage: {
                            pixelRatio: 4
                        }
                    }
                },
                series: [{
                    type: 'map',
                    mapType: name,
                    selectedMode: 'false', //是否允许选中多个区域
                    label: {
                        normal: {
                            show: true
                        },
                        emphasis: {
                            show: true
                        }
                    },
                    data: []
                }]
            };
            myChart.setOption(option);
        } else {
            alert('无法加载该地图');
        }
    });
}

myChart.on('click', function(params) {
    let name = params.name;         //地区name
    let mapCode = provinces[name];  //地区的json数据
    loadMap(mapCode, name);
});
```

**特别说明**

各个省份的地图效果，即Echarts的地图下钻功能。通过点击每个省份从而达到展开当前省份地图的效果。



### 测试数据效果演示

#### 全球地图 - 演示数据

**添加数据后演示效果**

![image-20200611201214724](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611201214724.png)

**配置项**

```
let option = {
    title: {
        text: '含有数据的世界地图',
        subtext: '数据来源：测试数据',
        left: 'center',
        top: 10
    },
    visualMap: {
        name:'',
        max: 1e+11,
        min:-1e+11,
        calculable: true,
        inRange: {
            color: ['#8dc1a9','#73b9bc','#759aa0','#e69d87','#ea7e53','#dd6b66']
        }
    },
    series: [{
        type: 'map',
        map: 'world',
        roam: true,
        data:demoData,
    }]
};
```

**说明**

1 - demoData数据的数据格式

```js
demoData = [
	{name:"Aruba",value:1.4107e+08},
    {name:"Austria",value:8.29106e+09},
    ...
]
```



#### 全国地图 - 演示数据

![image-20200611201321917](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611201321917.png)

**配置项**

```js
var option = {
    title: {
        text: name,
        left: 'center',
    },
    toolbox: {
        show: true,
        feature: {
            saveAsImage: {
                pixelRatio: 4
            }
        }
    },
    visualMap: {//左边的图标
        min: 0,
        max: 100000,
        left: 26,
        bottom: 30,
        showLabel: !0,
        text: ["高", "低"],
        pieces: [{
            gt: 10000,
            label: "> 10000人",
            color: "#7f1100"
        }, {
            gte: 1000,
            lte: 10000,
            label: "1000 - 10000人",
            color: "#ff5428"
        }, {
            gte: 100,
            lt: 1000,
            label: "100 - 1000人",
            color: "#ff8c71"
        }, {
            gt: 1,
            lt: 100,
            label: "1 - 100人",
            color: "#ffd768"
        }, {
            value: 0,
            color: "#ffffff"
        }],
        show: !0
    },
    series: [{
        type: 'map',
        mapType: name,
        selectedMode: 'false', //是否允许选中多个区域
        label: {
            normal: {
                show: true
            },
            emphasis: {
                show: true
            }
        },
        data: demoData
    }]
};
```

**说明**

1 - demoData的数据格式

```js
let demoData = [
 	{name: "澳门", value: 10},
    {name: "香港", value: 114},
    {name: "台湾", value: 45},
]
```

2 - 各省份的数据格式同全国格式一致

### Geo 地理坐标及覆盖物

#### Geo 配合 散点图使用

**展示效果**

![image-20200611201607153](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611201607153.png)

**Echarts配置项**

```js
var option = {
    backgroundColor: {
        type: 'linear',
        x: 0,
        y: 0,
        x2: 1,
        y2: 1,
        colorStops: [{
            offset: 0, color: '#0f378f' // 0% 处的颜色
        }, {
            offset: 1, color: '#00091a' // 100% 处的颜色
        }],
        globalCoord: false // 缺省为 false
    },
    title: {
        top:20,
        text: 'geo 和 map 共存',
        subtext: '测试案例',
        x: 'center',
        textStyle: {
            color: '#ccc'
        }
    },
    tooltip: {
        trigger: 'item',
        formatter: function (params) {
            if(typeof(params.value)[2] == "undefined"){
                return params.name + ' : ' + params.value;
            }else{
                return params.name + ' : ' + params.value[2];
            }
        }
    },
    legend: {
        orient: 'vertical',
        y: 'bottom',
        x:'right',
        data:['pm2.5'],
        textStyle: {
            color: '#fff'
        }
    },
    geo: [{
        map: 'china',
        show: true,
        roam: true,     // 是否开启鼠标缩放和平移漫游
        itemStyle: {
            normal: {
                areaColor: '#3a7fd5',
                borderColor: '#0a53e9',//线
                shadowColor: '#092f8f',//外发光
                shadowBlur: 20
            },
            emphasis: {
                areaColor: '#0a2dae',//悬浮区背景
            }
        }
    }],
    series : [
        {   name: 'light',
            type: 'scatter',
            coordinateSystem: 'geo',    // 使用的坐标系
            symbolSize: 5,
            label: {
                normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: true
                },
                emphasis: {
                    show: true
                }
            },
            itemStyle: {
                normal: {
                    color: '#fff'
                }
            },
            data: convertData(data),
        },
        {
            type: 'map',
            map: 'china',
            geoIndex: 0,
            aspectScale: 0.75, //长宽比
            showLegendSymbol: false, // 存在legend时显示
            label: {
                normal: {
                    show: false
                },
                emphasis: {
                    show: false,
                    textStyle: {
                        color: '#fff'
                    }
                }
            },
            roam: true,
            itemStyle: {
                normal: {
                    areaColor: '#031525',
                    borderColor: '#FFFFFF',
                },
                emphasis: {
                    areaColor: '#2B91B7'
                }
            },
            animation: false,
            data: data
        },
        {
            name: 'Top 5',
            type: 'scatter',
            coordinateSystem: 'geo',
            symbol: 'pin',
            symbolSize: [50,50],
            label: {
                normal: {
                    show: true,
                    textStyle: {
                        color: '#fff',
                        fontSize: 9,
                    },
                    formatter (value){
                        return value.data.value[2]
                    }
                }
            },
            itemStyle: {
                normal: {
                    color: '#D8BC37', //标志颜色
                }
            },
            data: convertData(data),
            showEffectOn: 'render',
            rippleEffect: {
                brushType: 'stroke'
            },
            hoverAnimation: true,
            zlevel: 1
        },
    ]
};
```

**说明**

Geo 覆盖物的原理是 基于 Geo 地理坐标系添加散点图，但是在series的map配置中添加GeoIndex的目的是禁止map本身渲染地理坐标系，而是使用Geo组件的地理坐标，否则就会产生两个地图。