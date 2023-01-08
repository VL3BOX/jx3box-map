<template>
    <div ref="component" class="c-map">
        <div class="u-map__wrapper" :style="wrapperSize">
            <img class="u-map-img" :src="mapImg" draggable="false" />
            <div class="u-map-title__wrapper">
                <slot name="title" v-bind:title="mapName">
                    <div class="u-map-title">{{ mapName }}</div>
                </slot>
            </div>
            <template v-for="(i, k) in datas">
                <div class="u-map-point__wrapper" :style="pointStyle(i)" :key="k">
                    <slot name="point" v-bind:data="i">
                        <el-popover class="u-map-point__popover" placement="top" width="200" trigger="hover">
                            <slot name="popover" v-bind:data="i">
                                <div>
                                    <div>{{ i.title }}</div>
                                    <div v-html="i.content"></div>
                                </div>
                            </slot>
                            <span slot="reference" class="u-map-point"> </span>
                        </el-popover>
                    </slot>
                </div>
            </template>
        </div>
    </div>
</template>

<script>
import jx3boxData from "@jx3box/jx3box-common/data/jx3box.json";
import mapScales from "../data/map_scales.json";

export default {
    name: "Jx3boxMap",
    props: {
        mapId: {
            type: Number,
            default: 1,
        },
        datas: {
            type: Array,
            default: () => [],
        },
    },
    data() {
        return {
            popoverContent: "",
            width: 1024,
            height: 896,
        };
    },
    computed: {
        subId() {
            let scales = mapScales[this.mapId];
            if (!scales || Object.keys(scales) <= 1) return 0;
            let _sub = 0;
            let _subScale = 0;
            for (let sub in scales) {
                let rect = {
                    x: scales[sub].StartX,
                    y: scales[sub].StartY,
                    width: scales[sub].Width / scales[sub].Scale,
                    height: scales[sub].Height / scales[sub].Scale,
                };
                if (this.isPointsInRect(rect) && scales[sub].Scale > _subScale) {
                    _sub = sub;
                    _subScale = scales[sub].Scale;
                }
            }
            return _sub;
        },
        mapName() {
            return mapScales[this.mapId][this.subId].Name;
        },
        mapScale() {
            return mapScales[this.mapId][this.subId];
        },
        mapImg() {
            return `${jx3boxData.__imgPath}map/maps/map_${this.mapId}_${this.subId}.png`;
        },
        wrapperSize() {
            return {
                width: this.width + "px",
                height: this.height + "px",
            };
        },
    },
    mounted() {
        this.$nextTick(function () {
            this.updateSize();
            window.addEventListener("resize", this.updateSize);
        });
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.updateSize);
    },
    methods: {
        pointStyle(item) {
            const scale = this.mapScale;
            const Width = scale.Width / scale.Scale;
            const Height = scale.Height / scale.Scale;
            const left = ((item.x - scale.StartX) / Width) * this.width;
            const bottom = ((item.y - scale.StartY) / Height) * this.height;
            return {
                left: left + "px",
                bottom: bottom + "px",
            };
        },
        isPointsInRect(rect) {
            let points = this.datas.map((p) => {
                return {
                    x: p.x,
                    y: p.y,
                };
            });
            return points.every((p) => {
                return p.x >= rect.x && p.x <= rect.x + rect.width && p.y >= rect.y && p.y <= rect.y + rect.height;
            });
        },
        updateSize() {
            this.width = this.$refs["component"]?.clientWidth;
            if (!this.width) return;
            this.height = this.width / (1024 / 896);
        },
    },
};
</script>

<style lang="less" scoped>
.c-map {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    .u-map__wrapper {
        display: inline-block;
        position: relative;
        border: 1px solid rgb(238, 238, 238);
        box-shadow: rgb(0 0 0 / 10%) 0px 0px 3px;
        box-sizing: border-box;
        user-select: none;
        height: 100%;

        .u-map-img {
            display: block;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        .u-map-title__wrapper {
            z-index: 2;
            position: absolute;

            top: 16px;
            width: 100%;
            display: flex;
            justify-content: center;

            .u-map-title {
                user-select: text;
                color: #efe;
                font-weight: bold;
                padding: 6px;
                background: rgba(0, 0, 0, 0.5);
            }
        }

        .u-map-point__wrapper {
            z-index: 3;
            position: absolute;

            .u-map-point__popover {
                display: block;
                width: 10px;
                height: 10px;
                background: #f94343;
                border: #efe solid 1px;
                cursor: pointer;
                transition: all 0.1s ease-in-out;
                &:hover {
                    transform: scale(1.6);
                }
            }

            .u-map-point {
                display: block;
                width: 10px;
                height: 10px;
            }
        }
    }
}
</style>
