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

## slots

允许一些部分的自定义slot。

### title

地图上方的名称部分

### point

地图上的点位元素，默认是一个带popover的小红点，如果需要替换的话需要自己给上popover

#### popover

如果使用默认的point，可以用popover这个slot来替换的默认的popover里的内容  