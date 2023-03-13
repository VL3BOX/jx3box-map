# @jx3box/jx3box-map

用于各种需要在地图上展示点位的场合  

## QuickStart

```js
import Jx3boxMap from '@jx3box/jx3box-map/src/components/Map.vue
```

```html
<jx3box-map :map-id="8" :datas="datas"></jx3box-map>
```

```js
export default {
    name: "App",
    components: {
        Jx3boxMap,
    },
    data: () => ({
        datas: [
            {
                x: 9655,
                y: 13231,
                title: "测试",
                content: "测试内容11<br />22",
            },
        ],
    }),
};
```

## props

组件props作用。

### mapId

当前所展示的地图

### datas

在地图上的点位，是一个对象数组。其中的对象的属性如表所示

| 属性名  | 描述             | 必填 | 栗子    | 备注                                           |
| ------- | ---------------- | ---- | ------- | ---------------------------------------------- |
| x       | 坐标x点          | 是   | 114514  |                                                |
| y       | 坐标y点          | 是   | 1919810 |                                                |
| title   | 悬浮框标题       | 否   |         |
| content | 悬浮框显示的内容 | 否   |         |
| focus   | 是否展示这个点   | 否   | true    | Boolean，多个数据focus都为true时只会展示第一个 |

### overview

默认情况下，overview为true。  
组件会尝试展示所有的点位，并且把整个地图缩放以适应容器大小。  
如果希望不缩放地图，仅展示一部分点位可以设置overview为false。  
组件将会只展示特定的点位，可以在datas内的对应对象设置focus为true或者给组件传入focus属性，该属性应该是要展示的点位在datas内的索引。  
如果不通过任何方式指定要展示的点位会直接取第一个。  

## slots

允许一些部分的自定义slot。

### title

地图上方的名称部分

### point

地图上的点位元素，默认是一个带popover的小红点，如果需要替换的话需要自己给上popover

#### popover

如果使用默认的point，可以用popover这个slot来替换的默认的popover里的内容  