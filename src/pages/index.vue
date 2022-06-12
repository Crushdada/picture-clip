<script setup lang="ts">
const picList = ref([
  {
    url: "https://img1.baidu.com/it/u=638159551,1901728755&fm=253&fmt=auto&app=120&f=JPEG?w=889&h=500",
    trackItemWidth: 1,
  },
  {
    url: "https://img1.baidu.com/it/u=1756691966,3932535320&fm=253&fmt=auto&app=120&f=JPEG?w=889&h=500",
    trackItemWidth: 1,
  },
  {
    url: "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242259192W5-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657449010&t=067a1eb74fff1c705825ad7d7402c072",
    trackItemWidth: 1,
  },
  {
    url: "https://img0.baidu.com/it/u=3096666729,1604160352&fm=253&fmt=auto&app=138&f=PNG?w=862&h=450",
    trackItemWidth: 1,
  },
  {
    url: "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F1114%2F011621113142%2F210116113142-12-1200.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657449010&t=17c8e19b33a2be793421c9286a9a6613",
    trackItemWidth: 1,
  },
  {
    url: "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fgss0.baidu.com%2F9vo3dSag_xI4khGko9WTAnF6hhy%2Fzhidao%2Fpic%2Fitem%2Ff7246b600c338744429a8f1b5d0fd9f9d72aa07e.jpg&refer=http%3A%2F%2Fgss0.baidu.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657447909&t=a7e823b3bbdb8953035630bdb1a3c5a7",
    trackItemWidth: 1,
  },
  {
    url: "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.php.cn%2Fupload%2Fjscode%2F000%2F120%2F096%2F5dbfc68d6436f397.jpg&refer=http%3A%2F%2Fimg.php.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657447928&t=3e6498045babaea26ff37b8963effe46",
    trackItemWidth: 1,
  },
  {
    url: [
      "https://img2.baidu.com/it/u=647493040,2510651914&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=293",
      "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242306111155-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657447978&t=91ce5218836895fc2648931c1538da39",
      "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F1114%2F063021120F9%2F210630120F9-1-1200.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1657447978&t=340134cf527eade6e27c472480dab2f1",
    ],
    trackItemWidth: 3,
  },
]);
const pathWay = ref([]) as unknown as any;
const trackImgWidth = 200;
const trackImgHeight = 120;
const dragIndex = ref(0);
const isExternalDrag = ref(false);
const activeTrackPicIndex = ref(0);
const pointerX = ref(0);

onMounted(() => {
  // Shim: Can not trigger the event by Key Aliases 'delete' or 'backspace'.
  document.body.addEventListener("keyup", (e) => {
    if (e.key === "Backspace" || e.key === "Delete")
      pathWay.value.splice(activeTrackPicIndex.value, 1);
    activeTrackPicIndex.value = -1;
  });
});
// styles & tool function
function isMultiplePics(trackItem: any) {
  return typeof trackItem.url !== "string";
}
function getBK(item: any) {
  return `url(${item.url})`;
}
function getTrackItemClass(trackPicIdx: number) {
  return trackPicIdx === activeTrackPicIndex.value
    ? "border-3 border-emerald"
    : "";
}
function setActivePic(e: any, idx: number) {
  pointerX.value = e.layerX;
  activeTrackPicIndex.value = idx;
}
function mergeImgsRow(list: Array<string>, cwith: number, cheight: number) {
  return new Promise((resolve, reject) => {
    const baseList = [];
    // 创建 canvas 节点并初始化
    const canvas = document.createElement("canvas");
    canvas.width = cwith * list.length;
    canvas.height = cheight;
    const context = canvas.getContext("2d");

    list.map((item, index) => {
      const img = new Image();
      img.src = item;
      // 跨域
      img.crossOrigin = "Anonymous";

      img.onload = () => {
        context.drawImage(img, cwith * index, 0, cwith, cheight);
        const base64 = canvas.toDataURL("image/png");
        baseList.push(base64);
        if (baseList[list.length - 1]) {
          // 返回新的图片
          resolve(baseList[list.length - 1]);
        }
      };
    });
  });
}
// 剪切逻辑
async function cutImage() {
  const source = pathWay.value[activeTrackPicIndex.value];
  if (isMultiplePics(source)) {
    const imgUrl = await mergeImgsRow(source, trackImgWidth, trackImgHeight);
    pathWay.value[activeTrackPicIndex.value] = imgUrl;
  }
}
// 拖拽逻辑
function dragstart(index: number, flag = false) {
  dragIndex.value = index;
  isExternalDrag.value = flag;
}
async function dragenter(e: any, index: number) {
  e.preventDefault();
  // 避免源对象触发自身的dragenter事件
  if (!isExternalDrag.value) {
    // 内部拖拽排序逻辑
    if (dragIndex.value !== index) {
      const source = pathWay.value[dragIndex.value];
      pathWay.value.splice(dragIndex.value, 1);
      pathWay.value.splice(index, 0, source);
      // 排序变化后目标对象的索引变成源对象的索引
      dragIndex.value = index;
      activeTrackPicIndex.value = index;
    }
  } else {
    // 外部拖拽插入轨道
    const source = picList.value[dragIndex.value];
    // 多张图片的插入逻辑
    if (isMultiplePics(source)) {
      // canvas拼接多张图片为一张
      const mergedSource = await mergeImgsRow(
        source.url as Array<string>,
        trackImgWidth,
        trackImgHeight
      );
      pathWay.value.splice(index, 0, {
        url: mergedSource,
        trackItemWidth: source.trackItemWidth * trackImgWidth,
      });
    } else {
      pathWay.value.splice(index, 0, {
        url: source.url,
        trackItemWidth: source.trackItemWidth * trackImgWidth,
      });
    }
    dragIndex.value = index;
    isExternalDrag.value = false;
    activeTrackPicIndex.value = index;
  }
}
function dragover(e: any, index: number) {
  e.preventDefault();
}
</script>

<template>
  <div>
    <div i-carbon-campsite text-4xl inline-block />
    <!-- 列表 -->
    <div>
      <div
        v-for="(picItem, idx) in picList"
        :key="idx"
        m-4
        cursor-move
        inline-block
        w-50
        h-30
        :style="{
          backgroundImage: getBK(picItem),
          backgroundSize: 'cover',
          backgroundRepeat: 'no-repeat',
          backgroundPosition: 'center',
        }"
        draggable="true"
        @dragstart="dragstart(idx, true)"
      />
    </div>
    <div mt-20 overflow-y-visible relative>
      <!-- 刻度指针 -->
      <div
        btn
        absolute
        :style="{
          top: '-32px',
          left: `${pointerX || 30}px`,
          transform: 'translateX(-50%)',
          transition: 'all ease-out 0.3s',
        }"
        @click="cutImage"
      >
        Cut
      </div>
      <!-- 轨道 -->
      <transition-group
        id="wrap"
        relative
        h-38
        flex
        items-center
        border="1 gray-500/40"
        shadow-md
        flex-nowrap
        overflow-x-auto
        name="drag"
        tag="div"
      >
        <!-- 分割线 -->
        <div
          :key="2"
          absolute
          scale-x-50
          z-1
          w-1px
          h-full
          :style="{
            left: `${pointerX}px`,
            'background-color': '#00fff7',
          }"
        ></div>
        <template v-for="(trackItem, idx) in pathWay" :key="trackItem">
          <div
            border-emerald
            shadow-md
            cursor-move
            inline-block
            flex-grow-0
            flex-shrink-0
            :class="getTrackItemClass(idx)"
            :style="{
              width: `${trackItem.trackItemWidth}px`,
              height: `${trackImgHeight}px`,
              backgroundImage: getBK(trackItem),
              backgroundSize: 'cover',
              backgroundRepeat: 'no-repeat',
              backgroundPosition: 'center',
            }"
            draggable="true"
            @dragenter="dragenter($event, idx)"
            @dragover="dragover($event, idx)"
            @dragstart="dragstart(idx)"
            @click="setActivePic($event, idx)"
          />
        </template>
        <div
          h-full
          flex-grow
          :key="1"
          @dragenter="dragenter($event, pathWay.length)"
        ></div>
      </transition-group>
    </div>
  </div>
</template>

<style>
.drag-move {
  transition: transform ease-out 0.3s;
}

::-webkit-scrollbar {
  width: 12px;
  height: 12px;
  border-radius: 15px;
  -webkit-border-radius: 15px;
}

::-webkit-scrollbar-track-piece {
  background-color: transparent;
  border-radius: 15px;
  -webkit-border-radius: 15px;
}

::-webkit-scrollbar-thumb:vertical {
  height: 10px;
  background-color: rgba(144, 147, 153, 0.3);
  border-radius: 15px;
  -webkit-border-radius: 15px;
  opacity: 0.2;
}

::-webkit-scrollbar-thumb:horizontal {
  width: 12px;
  background-color: rgba(144, 147, 153, 0.3);
  border-radius: 15px;
  -webkit-border-radius: 15px;
  opacity: 0.2;
}
</style>
