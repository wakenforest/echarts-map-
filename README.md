## echarts地图可以使用geoJson的数据：

### 1 下面是一个典型数据格式：
```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands"
  }
}
```

### 2 geoJson数据

这份数据有点老，崇文宣武还在，凑合用。


### 3 实现上下钻的代码如下：

```html
<html>

<head>
    <script src="js/echarts.js"></script>
    <script src="js/jquery.min.js"></script>
</head>

<body>
    <div id="map" style="width:1000px;height:700px;"></div>
<script>


(function(echarts) {
    option_map = {
            title: {
				text: '中国',
				left: 'center',
				top: '15',
				textStyle: {
					color: '#B81820',
					fontSize: 22
				}
			},
            toolbox: {
                show : true,
                itemSize : 30,
                itemGap: 30,
                iconStyle: {
                    // shadowColor: 'rgba(0, 0, 0, 0.5)',
                    // shadowBlur: 10
                    // borderColor: '#666',
                    // color: '#665',
                },
                feature : {
                    // mark : {show: true},
                    // dataView : {show: true, readOnly: false},
                    // magicType : {show: true, type: ['line', 'bar']},
                    // restore : {show: true},
                    myBtn:{//自定义按钮 danielinbiti,这里增加，selfbuttons可以随便取名字  
                        show:true,//是否显示  
                        title:'自定义', //鼠标移动上去显示的文字  
                        icon:'image://images/test.png', //图标  
                        option:{},  
                        onclick:function() {//点击事件,这里的option1是chart的option信息  
                                // alert('1');//这里可以加入自己的处理代码，切换不同的图形 
                                if(level==2){
                                    option_map.title.text = "中国";
                                    country();
                                }
                                else if(level==3){
                                    console.log(option_map);
                                    option_map.title.text = parentName;
                                    province(parentId,'');
                                }
                            }  
                        },
                    saveAsImage : {show: true}
                }
            },
            geo: {
                map: 'mapCity',
                label: {
                    emphasis: {
                        show: true,
                        color: '#fff',
                    }
                },
                roam: true, //允许拖动
                itemStyle: {
                    normal: {
                        areaColor: 'rgba(255, 255, 255, 0.9)',
                        borderColor: '#02334a',
                        borderWidth: 1,
                        shadowColor: 'rgba(0, 189, 249, 0.5)',
                        shadowBlur: 10,
                        shadowOffsetY: 0,
                        shadowOffsetX: 0,
                    },
                    emphasis: {
                        areaColor: '#006387'
                    }
                }
            }
        }

    var map = echarts.init(document.getElementById('map'))
    var reset = function() {
        map = echarts.init(document.getElementById('map'));
        map.setOption(option_map);
    }

    var level = 1
    var parentId = '';
    var parentName = '';
    var country = function(){
        level = 1
        $.ajax({
            url: 'mapdata/china.json',
            type: 'get',
            success: function(res) {
               echarts.registerMap("mapCity", res);
                map.dispose();
                reset();
                map.on('click', (params) => {
                    const provinceCode=JSON.parse(res).features.filter((val) => {
                        if(val.properties.name.indexOf(params.name)!==-1){
                            return true
                        }
                    });
                    parentId = '';
                    parentName = '';
                    province(provinceCode[0].properties.id,params.name);
                    option_map.title.text = params.name;
                });
            }
        })
    }

    var province = function(id,name){
        level = 2
        $.ajax({
            url: 'mapdata/geometryprovince/'+id+'.json',
            type: 'get',
            success: function(res) {
               echarts.registerMap("mapCity", res);
               map.dispose()
                reset() 
                map.on('click', (params) => {
                    const provinceCode=JSON.parse(res).features.filter((val) => {
                        if(val.properties.name.indexOf(params.name)!==-1){
                            return true;
                        }
                    });

                    parentId = id;
                    parentName = name;
                    city(provinceCode[0].properties.id,params.name);
                    option_map.title.text = params.name;
                });
            }
        })
    }

    var city = function(id,name){
        level = 3
        $.ajax({
            url: 'mapdata/geometryCouties/'+id+'00.json',
            type: 'get',
            success: function(res) {
                echarts.registerMap("mapCity", res);
                map.dispose()
                reset() 
                map.on('click', (params) => {
                    alert(params.name)
                });
            }
        })
    }

    country()

})(echarts)

</script>

</body>

</html>
```