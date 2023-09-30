<template>
    <div ref="component" class="c-map">
        <div class="u-map__wrapper" :style="wrapperSize">
            <div class="u-map__inner" :style="innerStyle">
                <img class="u-map-img" :src="mapImg" draggable="false" />
                <div class="u-map-title__wrapper" v-if="overview">
                    <slot name="title" v-bind:title="mapName">
                        <div class="u-map-title">{{ mapName }}</div>
                    </slot>
                </div>
                <template v-for="(i, k) in datas">
                    <div class="u-map-point__wrapper" :style="pointStyle(i)" :key="k">
                        <slot name="point" v-bind:data="i">
                            <el-popover popper-class="u-map-point__popover" placement="top" width="200" trigger="hover">
                                <slot name="popover" v-bind:data="i">
                                    <div>
                                        <div v-if="!overview" class="u-map-title">{{ mapName }}</div>
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
    </div>
</template>

<script>
import jx3boxData from "@jx3box/jx3box-common/data/jx3box.json";
import { getMapScales } from "../service/data";

export default {
    name: "Jx3boxMap",
    props: {
        // 地图ID
        mapId: {
            type: Number,
            default: 1,
        },
        // 点位数据
        datas: {
            type: Array,
            default: () => [],
        },
        overview: {
            type: Boolean,
            default: true,
        },
        focus: {
            type: Number,
            default: undefined,
        },
        hiddenBoarder: {
            type: Boolean,
            default: false,
        },
        draggable: {
            type: Boolean,
            default: true,
        },
    },
    data: () => ({
        outerWidth: 0,
        outerHeight: 0,
        mapScales: {},
    }),
    computed: {
        // 内层容器宽高
        innerWidth() {
            return this.overview ? this.outerWidth : 1024;
        },
        innerHeight() {
            return this.overview ? this.outerHeight : 896;
        },
        // 容器尺寸
        wrapperSize() {
            return {
                width: this.outerWidth + "px",
                height: this.outerHeight + "px",
            };
        },
        // 中心点
        focusPoint() {
            if (this.focus != undefined) {
                return this.datas[this.focus];
            } else {
                return this.datas.find((d) => d.focus) ?? this.datas[0];
            }
        },
        // 内层容器相对外层容器偏移
        innerOffset() {
            if (this.overview) return { x: 0, y: 0 };
            // 外层容器的中心点
            const outerCenter = {
                x: this.outerWidth / 2,
                y: this.outerHeight / 2,
            };
            // 要展示的点相对内层容器的偏移
            const positionOffset = this.pointPosition(this.focusPoint);
            const innerOffset = {
                left: outerCenter.x - positionOffset.left,
                bottom: outerCenter.y - positionOffset.bottom,
            };
            return innerOffset;
        },
        innerStyle() {
            const style = {
                width: this.innerWidth + "px",
                height: this.innerHeight + "px",
            };
            if (this.overview) return style;
            let { left, bottom } = this.innerOffset;
            // 边界条件处理
            const maxLeft = 0;
            const minLeft = this.outerWidth - this.innerWidth;
            left = Math.max(Math.min(left, maxLeft), minLeft);
            const maxBottom = 0;
            const minBottom = this.outerHeight - this.innerHeight;
            bottom = Math.max(Math.min(bottom, maxBottom), minBottom);
            return {
                ...style,
                left: left + "px",
                bottom: bottom + "px",
            };
        },
        // 地图ID、名称、尺寸、图片等
        subId() {
            let scales = this.mapScales[this.mapId];
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
            return this.mapScales[this.mapId]?.[this.subId]?.Name;
        },
        mapScale() {
            return this.mapScales[this.mapId]?.[this.subId];
        },
        mapImg() {
            return `${jx3boxData.__imgPath}map/maps/map_${this.mapId}_${this.subId}.png`;
        },
    },
    mounted() {
        getMapScales().then((data) => {
            this.mapScales = data;
        });
        this.$nextTick(function () {
            this.updateSize();
            window.addEventListener("resize", this.updateSize);
        });
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.updateSize);
    },
    methods: {
        pointPosition(item) {
            const scale = this.mapScale;
            if (!scale) return { left: 0, bottom: 0 };
            const Width = scale.Width / scale.Scale;
            const Height = scale.Height / scale.Scale;
            const left = ((item.x - scale.StartX) / Width) * this.innerWidth;
            const bottom = ((item.y - scale.StartY) / Height) * this.innerHeight;
            return {
                left,
                bottom,
            };
        },
        pointStyle(item) {
            const { left, bottom } = this.pointPosition(item);
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
            this.outerWidth = this.$refs["component"]?.clientWidth;
            if (!this.outerWidth) return;
            if (this.overview) {
                this.outerHeight = this.outerWidth / (1024 / 896);
            } else {
                this.outerHeight = this.$refs["component"]?.clientHeight;
            }
            this.$emit("resize", [this.outerWidth, this.outerHeight]);
        },
    },
};
</script>

<style lang="less" scoped>
.c-map {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    .u-map__wrapper {
        overflow: hidden;
        position: relative;
    }

    .u-map__inner {
        display: inline-block;
        position: absolute;
        border: 1px solid rgb(238, 238, 238);
        box-shadow: rgb(0 0 0 / 10%) 0px 0px 3px;
        box-sizing: border-box;
        user-select: none;
        height: 100%;
    }

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
        width: 10px;
        height: 10px;
        position: absolute;

        .u-map-point {
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
    }
}

.u-map-point__popover {
    .u-map-title {
        font-weight: bold;
        color: #aaa;
    }
}
</style>
