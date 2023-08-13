<template>
<div class="pagination-wrap">
    <div
        v-for="info in displayInfos" :key="info.val"
        class="cube" :class="{active: nowPage === info.val}"
        @click="handleCubeClick(info)"
    >
        <div
            v-if="info.type === 'pageIndicator'"
            class="page-indicator"
        >{{info.val}}</div>
<!--      operationBtn  -->
        <div
            v-else class="operation-btn"
        >
            {{btnCnMap[info.val]}}
        </div>
    </div>
</div>
</template>

<script lang="ts">
import {computed, defineComponent, toRef} from "vue";
type OperationBtnVals = 'prev'|'next'|'fastPrev'|'fastNext';
type DisplayInfo = {
    /**
     * 元素类型
     * pageIndicator: 页数指示器   [1] [2] ....
     * operationBtn: 功能按钮，看 val
     * 不实现 // --ellipsis: 省略号--
     * */
    type: 'pageIndicator'|'operationBtn',
    /**
     * 展示内容
     * 当type 为 operationBtn时：
     * prev: 上N页功能按钮，， N根据 maxPageIndicatorNum 算
     * next: 下N页
     * fastPrev: 快速上调到 第一页
     * fastNext: 快速下调到 最后一页
     * */
    val: number|string|OperationBtnVals,
};

export default defineComponent({
    name: 'Pagination',
    emits: ['update:nowPage'],
    props: {
        nowPage: {type: Number, required: true}, // 当前页数，，第一页 -> 1
        total: {type: Number, required: true}, // 总数
        pageSize: {type: Number, required: true}, // 一页多少条
        maxPageIndicatorNum: {type: Number, required: true}, // 最多展示多少个 页数的 指示器
    },
    setup: function (props, ctx) {
        const total = toRef(props, 'total');
        const pageSize = toRef(props, 'pageSize');
        const maxPageIndicatorNum = toRef(props, 'maxPageIndicatorNum');
        const nowPage = toRef(props, 'nowPage');
        const displayInfos = computed<DisplayInfo[]>(() => {
            const totalPage = Math.ceil(total.value / pageSize.value); // 总页数
            const maxPageIndicatorNumV = maxPageIndicatorNum.value;
            const tmp: DisplayInfo[] = [];
            if (totalPage <= maxPageIndicatorNum.value) { // 总页数 <= 最多展示页数，
                for (let i = 0; i < maxPageIndicatorNumV; i++) {
                    tmp.push({type: 'pageIndicator', val: i + 1});
                }
                return tmp;
            }
            /** 总页数 > 最多展示页数 */
            const nowPageV = nowPage.value;
            /** 在最前面的几页 */
            if (nowPageV <= maxPageIndicatorNumV) {
                for (let i = 0; i < maxPageIndicatorNumV; i++) {
                    tmp.push({type: 'pageIndicator', val: i + 1});
                }
                // 添加 next fastNext
                tmp.push({type: 'operationBtn', val: 'next'});
                tmp.push({type: 'operationBtn', val: 'fastNext'});
                return tmp;
            }
            /** 当前页数 在最后几页 */
            if (nowPageV >= (totalPage - maxPageIndicatorNumV + 1)) {
                tmp.push({type: 'operationBtn', val: 'fastPrev'});
                tmp.push({type: 'operationBtn', val: 'prev'});
                for (let i = (totalPage - maxPageIndicatorNumV); i < totalPage; i++) {
                    tmp.push({type: 'pageIndicator', val: i + 1});
                }
                return tmp;
            }
            /** 页数在中间 */
            // 根据 最多展示页数，，计算 当前页 位置
            // 如： 当前页数 6， 最多展示4页，则余数 -> (6 - 1) % 4 -> 1
            // 则[0, 1(here), 2, 3]
            const indicatorBeginPage = nowPageV - ((nowPageV - 1) % maxPageIndicatorNumV);
            tmp.push({type: 'operationBtn', val: 'fastPrev'});
            tmp.push({type: 'operationBtn', val: 'prev'});
            for (let i = 0; i < maxPageIndicatorNumV; i++) {
                tmp.push({type: 'pageIndicator', val: indicatorBeginPage + i});
            }
            tmp.push({type: 'operationBtn', val: 'next'});
            tmp.push({type: 'operationBtn', val: 'fastNext'});
            return tmp;
        });
        const btnCnMap = {prev: 'p', next: 'n', fastPrev: 'FP', fastNext: 'FN'};
        function changePage(n: number) {
            if (n === nowPage.value) {
                return;
            }
            ctx.emit('update:nowPage', n);
        }
        function handleOperationBtn(operationBtnVal: OperationBtnVals) {
            if (operationBtnVal === 'fastPrev') {
                ctx.emit('update:nowPage', 1);
                return;
            }
            if (operationBtnVal === 'fastNext') {
                ctx.emit('update:nowPage', Math.ceil(total.value / pageSize.value));
                return;
            }
            const nowPageV = nowPage.value;
            const maxPageIndicatorNumV = maxPageIndicatorNum.value;
            const indicatorBeginPage = nowPageV - ((nowPageV - 1) % maxPageIndicatorNumV);
            console.log(indicatorBeginPage);
            if (operationBtnVal === 'prev') {
                ctx.emit('update:nowPage', Math.max(indicatorBeginPage - maxPageIndicatorNumV, 1));
                return;
            }
            if (operationBtnVal === 'next') {
                const totalPage = Math.ceil(total.value / pageSize.value); // 总页数
                ctx.emit('update:nowPage', Math.min(indicatorBeginPage + maxPageIndicatorNumV, totalPage));
            }
        }
        function handleCubeClick(info: DisplayInfo) {
            if (info.type === 'pageIndicator') {
                changePage(info.val as number);
                return;
            }
            // if (info.type === 'operationBtn')
            handleOperationBtn(info.val as OperationBtnVals);
        }
        return {
            nowPage,
            displayInfos,
            btnCnMap,
            handleCubeClick,
        };
    }
});



</script>

<style scoped>
.pagination-wrap {
    display: flex;
    align-items: center;
}

.pagination-wrap .cube {
    width: 30px;
    height: 30px;
    background: cyan;
    margin-right: 8px;
    border-radius: 3px;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
}
.pagination-wrap .cube.active {
    background: darkred;
    color: #fff;
}
.page-indicator {
    font-size: 15px;
    line-height: 20px;
}

.operation-btn {
    font-size: 15px;
    line-height: 20px;
}
</style>
