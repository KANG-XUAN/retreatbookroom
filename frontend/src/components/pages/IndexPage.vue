<!-- prettier-ignore -->
<template>
  <!-- 首頁主體 -->
  <main>
    <!-- :key="carouselIndex" 和 :key="broadcastIndex"：這是關鍵部分。透過將一個唯一的 key 綁定到 <img> 元素上，並且這個 key 會隨著 carouselIndex 或 broadcastIndex 的更新而改變，你告訴 Vue 這是一個不同的元素。當 key 改變時，Vue 會將舊元素視為被移除，新元素視為被插入，從而觸發 transition 動畫。 -->
    <div class="carousel" @mouseenter="isChange(false)" @mouseleave="isChange(true)">
      <!-- mode="out-in"：<transition> 組件上的這個屬性確保了離開的元素會先完成動畫，然後進入的元素才開始動畫。這可以防止元素在動畫過程中重疊。 -->
      <transition v-bind:name="slide" mode="out-in">
        <img :key="carouselIndex" v-bind:src="carouselUrl[carouselIndex]" v-bind:alt="carouselAlt[carouselIndex]"
          loading="lazy" />
      </transition>

      <!-- 左右按鈕 -->
      <div class="nextImg" @click="increaseCarouselIndex(1)">▶︎</div>
      <div class="lastImg" @click="increaseCarouselIndex(-1)">◀︎</div>

      <!-- 輪播圖片下方中間半透明的圓球，選擇特定圖片 -->
      <div class="changeCarouselIndex">
        <div class="setCarouselIndex" v-bind:class="{ 'active-red': carouselIndex === 0 }" @click="setCarouselIndex(0)">
        </div>
        <div class="setCarouselIndex" v-bind:class="{ 'active-red': carouselIndex === 1 }" @click="setCarouselIndex(1)">
        </div>
        <div class="setCarouselIndex" v-bind:class="{ 'active-red': carouselIndex === 2 }" @click="setCarouselIndex(2)">
        </div>
        <div class="setCarouselIndex" v-bind:class="{ 'active-red': carouselIndex === 3 }" @click="setCarouselIndex(3)">
        </div>
        <div class="setCarouselIndex" v-bind:class="{ 'active-red': carouselIndex === 4 }" @click="setCarouselIndex(4)">
        </div>
      </div>
    </div>

    <sub_block @select_isbn="getIsbn" />

    <!-- 促銷訊息 -->
    <div class="broadcast" @mouseenter="isChange(false)" @mouseleave="isChange(true)">
      <transition name="slide" mode="out-in">
        <img :key="broadcastIndex" :src="broadcastImagesUrl[broadcastIndex]" :alt="broadcastImagesAlt[broadcastIndex]"
          loading="lazy" />
      </transition>

      <div class="changeBroadcastIndex">
        <div class="setBroadcastIndex" v-bind:class="{ 'active-red': broadcastIndex === 0 }"
          @click="setBroadcastIndex(0)"></div>
        <div class="setBroadcastIndex" v-bind:class="{ 'active-red': broadcastIndex === 1 }"
          @click="setBroadcastIndex(1)"></div>
      </div>
    </div>
  </main>
</template>

<!-- <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script> -->
<!-- prettier-ignore -->
<script>
import sub_block from "@/components/pages/sub_block.vue";

export default {
  name: "TitleNav.vue",

  data() {
    return {
      originBooks: null, //所有書籍
      currentBooks: [], //現在顯示書籍
      currentIndex: -1, //書籍詳細內容需要的id
      showPopup: false, //是否顯示彈出視窗
      interval: null, //輪播計時器
      carouselIndex: 0, //輪播圖片id
      carouselUrl: [
        "https://lh3.googleusercontent.com/pw/AP1GczOD0O5aJr_3w2Hg0mEYeC7BO20PMazSANcR24Pk2ODmMWSA9e2cnwOKVNs1H7iqnpr5Ml2tlhU6I8MomNn4x_BNuznQ-q1o7HyfmZV8AzavB9MlPvwTnHzVUFFXEwK6P5Fl7QGzvwYops_dr2iHHrmc=w1024-h528-s-no-gm?authuser=0",
        "https://lh3.googleusercontent.com/pw/AP1GczOcgbypEEZMW5nKDjl178V3MvVKQk1Z3TtDrEL6iPzEf20rn5cPYXVQ2pvB1sIdTA5TDwYjWwzV_0IUZ-VnqPNSd2GbP28d49xK0HhcS3Y-iO33esHSCFdFpfYmsWvtZjYxUoU4jtM85Szr-zryq13Z=w1013-h754-s-no-gm?authuser=0",
        "https://lh3.googleusercontent.com/pw/AP1GczObW6R8ISPk64-E_yQU1WE8mxfR48Y-gXrXxdJLxNjdsRRjdgfI9mkVQxFzx7jEhegubH6z3W6CG_ONIpCvDQAyz0LK00sXG0vKkkn5J0KkqTDOR2kb81NaFDIUuY0EPTIlocZlxEvHTwYvz826Lb4M=w1334-h890-s-no-gm?authuser=0",
        "https://lh3.googleusercontent.com/pw/AP1GczNzDAevB84GaSI0bRBIPvp0R89Dp-Bx0hGAsy87c74E8x54EqjUFL8M_MvtQRk0MtK6AdC4PkbKyKuRmMsHkqFWA2zAVd_tZJO3Y0Jhbnr327wVNXGPtaJXccoruain-JRVH0ssC609JsdkeJK01qzX=w664-h666-s-no-gm?authuser=0",
        "https://lh3.googleusercontent.com/pw/AP1GczO4U6XUxknnXgg-SvguLKJhuj6Pg6lwmucu-PI5cDSKc3N0gM7bE1BI-qOrP0QvA5JjymOiSVQ3wwScsToeJHf_kuwLHdKjIuFPfz1L1iV2Yhu_Wf0dhzPkre_568h6KOWM1zpHLRFMLScTCUjLvriG=w890-h890-s-no-gm?authuser=0",
      ],
      carouselAlt: ["紅樓夢推廣", "三國演義推廣", "AI淘汰人類推廣", "經典文學特展", "少年隊防詐騙"],
      slide: "slideLeft", //動畫方向
      broadcastIndex: 0,
      broadcastImagesUrl: [
        "https://lh3.googleusercontent.com/pw/AP1GczOaiGsYBM0zrSoiqgWZX-FsslrsHg6mNiB42OImZlQJzX91DbZLtNXQK0-YYctsdU1Kx1V095rf3N2jLDOZSUktk8jts-JnXxJJBtKihGY89UI-HnZ0EpT0Eojw2I-TbSfuMgW9GuXJVMbPNoXt-_s_=w647-h649-s-no-gm?authuser=0",
        "https://lh3.googleusercontent.com/pw/AP1GczNrkAjLCL7S9Jgvc7QS1NiGjCGU_Aq89od3qMwCgFbOWS5G6PcyksFULhn2gwb_-n0dyqIbSSweu7F5KG_l4Ddp1bcOOH6c8YTIIhKLaItxzCbi0SYoVcEspB0MCDw1byYx3kEEI1bT4pUUg0lnGDxQ=w641-h642-s-no-gm?authuser=0",
      ],
      broadcastImagesAlt: ["浴佛節推廣活動", "植樹節推廣活動"],
    };
  },
  components: { sub_block },

  mounted() {
    this.fetchRecommendBooks();

    //是否啟用圖片輪播
    this.isChange(true);
  },

  methods: {
    async fetchRecommendBooks() {
      try {
        // 改成這樣能直接從main.js統一控制來源
        const response = await this.$axios.get("/api/railwayDB/products");

        const data = response.data; // 直接取用陣列
        if (!Array.isArray(data)) {
          alert("資料格式錯誤，應該是陣列");
          return;
        }

        this.currentBooks = data;
      } catch (error) {
        alert("讀取資料失敗，請看 console");
        console.error("讀取資料失敗:", error);
      }
    },

    getIsbn(isbn) {
      //取到這個isbn了我要跳轉到isbn頁! -->現在的路由要改成...
      console.log('選擇的ISBN:', isbn); // 這裡是接在前端路由後
      this.$router.push(`/book/${encodeURIComponent(isbn)}`);
    },

    // 這裡丟前端網址
    performSearch(searchText, searchScope) {
      // 透過路由傳遞搜尋內容和範圍
      this.$router.push({
        name: 'MyProduct', // 商品頁面的路由名稱
        query: {
          q: searchText,
          scope: searchScope
        }
      });
    },

    isChange(changeOrNot) {
      if (changeOrNot) {
        this.interval = setInterval(() => {
          this.carouselIndex++;
          if (this.carouselIndex === 5) {
            this.carouselIndex = 0;
          }

          this.broadcastIndex++;
          if (this.broadcastIndex === 2) {
            this.broadcastIndex = 0;
          }
          // 對於自動輪播，通常希望有統一的滑動方向
          this.slide = "slideLeft";
        }, 5000);
      } else {
        clearInterval(this.interval);
      }
    },

    increaseCarouselIndex(num) {
      if (num === 1) {
        this.slide = "slideLeft";
      } else {
        this.slide = "slideRight";
      }

      this.carouselIndex += num;
      if (this.carouselIndex >= 5) {
        this.carouselIndex = 0;
      } else if (this.carouselIndex < 0) {
        this.carouselIndex = 4;
      }
    },

    setCarouselIndex(num) {
      // 根據目前索引和新索引判斷滑動方向
      if (num > this.carouselIndex) {
        this.slide = "slideLeft";
      } else if (num < this.carouselIndex) {
        this.slide = "slideRight";
      }
      this.carouselIndex = num;
    },

    setBroadcastIndex(num) {
      this.broadcastIndex = num;
    },
  },
};
</script>

<style scoped>
/* 格子 */
/* 設定寬度跟置中 */
.container {
  margin: 0 auto;
  width: 1500px;
}

/* 分類頁的作者跟價格 */
.col3 {
  /* border: 1px black solid; */
  display: flex;
  flex-direction: column;
  width: 200px;
  height: 390px;
  margin: 10px auto;
  padding: 0 auto;
}

.authorColor {
  color: hsl(36, 50.7%, 50%);
  transition: all 0.55s;
}

.authorColor:hover {
  color: hsl(38, 74%, 24%);
}

.PandChartBtn {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  align-items: end;
  /* 垂直置中 */
}

/* 分類頁的價格 */
.PandChartBtn h3 {
  font-size: 1.5rem;
  font-weight: bold;
  color: hsl(353, 100%, 29.2%);
  transition: all 1s;
}

/* 分類頁的加入購物車按鈕 */
.PandChartBtn button {
  color: white;
  background-color: hsl(36, 50.7%, 71.4%);
  border: none;
  padding: 10px 12px;
  border-radius: 20px;
  transition: all 0.7s;
}

.PandChartBtn button:hover {
  background-color: hsl(353, 100%, 29.2%);
}

/* 商品小圖 */
.smProduct img {
  /* width: 150px; */
  width: 100%;
  margin: 15px auto;
  transition: all 0.5s;
}

.smProduct {
  margin-bottom: 50px;
  padding: 30px;
}

/* 書名超連結 */
.smProduct span {
  color: rgb(34, 34, 34);
  padding-bottom: 10px;
  display: inline-block;
  transition: all 0.55s;
}

/* 動態產生線 */
.smProduct h4::after {
  background-color: hsl(353, 100%, 29.2%);
  content: "";
  display: block;
  width: 30%;
  height: 2px;
  transition: all 0.4s;
}

.smProduct a:hover h4::after {
  width: 100%;
}

/* 滑到變圓角，可惜手機不能有 */
.smProduct a:hover img {
  scale: 1.03;
  border-radius: 20px;
  z-index: 2;
}

.smProduct a:hover span {
  color: hsl(36, 50.7%, 71.4%);
}

/* 改尺寸 */
/* 659~577 */
@media (min-width: 577px) and (max-width: 670px) {

  /* 分類頁的作者跟價格 */
  .PandChartBtn {
    flex-direction: column;
    margin: 0px auto;
  }

  .PandChartBtn h3 {
    font-size: 20px;
    margin-bottom: 0px;
  }

  .PandChartBtn button {
    width: 100%;
    border-radius: 0px;
  }

  /* 小圖區 */
  .col3 {
    margin: 0 auto;
    height: 370px;
    width: 150px;
  }

  .PandChartBtn {
    margin-top: auto;
  }
}

/* 中尺寸 */
@media (min-width: 670px) and (max-width: 992px) {

  /* 分類頁的作者跟價格 */
  /* 書名超連結 */
  .smProduct span {
    font-size: 23px;
  }

  /* 價格 */
  .PandChartBtn {
    align-items: end;
  }

  .PandChartBtn h3 {
    font-size: 22px;
  }

  .PandChartBtn button {
    width: 110px;
  }

  /* 小圖區 */
  .col3 {
    margin: 10px auto;
    padding: 0 2px;
    width: 190px;
    height: 360px;
  }

  /* 細項的li整體往左移 */
  .subA {
    margin-left: -20px;
  }
}

/* 小尺寸 */
@media (max-width: 577px) {
  .container {
    margin: 0 auto;
    width: 100%;
  }

  /* 分類頁的作者跟價格 */
  .PandChartBtn {
    flex-direction: column;
  }

  /* 價格 */
  .PandChartBtn h3 {
    font-size: 25px;
    margin-bottom: 0px;
  }

  .PandChartBtn button {
    width: 100%;
    border-radius: 0px;
  }

  /* 小圖區 */
  .col3 {
    margin: 10px auto;
    padding: 0 15px;
    width: 170px;
    height: 330px;
  }

  .smProduct span {
    font-size: 23px;
  }
}

/* 我的程式碼 */
main {
  margin: 0 10vw;
}

a {
  cursor: pointer;
}

/* 輪播 */
.carousel {
  width: 100%;
  height: 50vw;
  /* 隨寬度調整，因為手機都是高度較長，而我的圖片是寬度較長 */
  position: relative;
  top: 70px;
}

/* 因為輪播圖的大小沒有統一，因此要設定盒子和圖片寬高，然後設定拉伸樣式，避免盒子大小變動導致頁面變動 */
.carousel img {
  width: 100%;
  height: 100%;
  /* 縮小圖片以符合容器大小 */
  object-fit: contain;
}

/* 開始動畫與結束動畫都是指定逐禎動畫名稱 */
.slideLeft-enter-active {
  animation: slideLeft-in 0.1s;
}

.slideLeft-leave-active {
  animation: slideLeft-out 0.1s;
}

/* 逐禎動畫可以使用百分比，如果只設置開始動畫與結束動畫，則可以使用from, to */
@keyframes slideLeft-in {
  0% {
    transform: translateX(100vw);
  }

  100% {
    transform: translateX(0);
  }
}

@keyframes slideLeft-out {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(-100vw);
  }
}

.slideRight-enter-active {
  animation: slideRight-in 0.1s;
}

.slideRight-leave-active {
  animation: slideRight-out 0.1s;
}

/* 逐禎動畫可以使用百分比，如果只設置開始動畫與結束動畫，則可以使用from, to */
@keyframes slideRight-in {
  0% {
    transform: translateX(-100vw);
  }

  100% {
    transform: translateX(0);
  }
}

@keyframes slideRight-out {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(100vw);
  }
}

/* 輪播區左右按鈕 */
.nextImg {
  font-size: 1em;
  position: absolute;
  top: 50%;
  right: 1%;
  color: hsl(0, 0%, 100%);
  background-color: hsl(353, 100%, 29.2%);
  border-radius: 50%;
}

.nextImg:hover,
.lastImg:hover {
  color: hsl(0, 0%, 70%);
  background-color: hsl(353, 100%, 50%);
}

.nextImg:active,
.lastImg:active {
  color: hsl(353, 100%, 29.2%);
  background-color: hsl(0, 0%, 100%);
}

.lastImg {
  font-size: 1em;
  position: absolute;
  top: 50%;
  left: 1%;
  color: hsl(0, 0%, 100%);
  background-color: hsl(353, 100%, 29.2%);
  border-radius: 50%;
}

@media (min-width: 576px) {

  .lastImg,
  .nextImg {
    font-size: 2em;
  }
}

@media (min-width: 992px) {

  .lastImg,
  .nextImg {
    font-size: 3em;
  }
}

/* 輪播區中間下方按鈕 */
.changeCarouselIndex {
  position: absolute;
  bottom: 0;
  left: 25vw;
}

.setCarouselIndex {
  display: inline-block;
  height: 5vw;
  width: 5vw;
  border-radius: 50%;
  border: 1px solid hsl(353, 100%, 29.2%);
}

.setCarouselIndex:hover {
  background-color: hsl(353, 100%, 50%);
}

.setCarouselIndex:active {
  background-color: hsl(353, 0%, 100%);
}

.active-red {
  background-color: hsl(353, 100%, 29.2%);
}

.setCarouselIndex+.setCarouselIndex {
  margin-left: 1vw;
}

/* 推薦商品 */
.recommend {
  position: relative;
  top: 50px;
}

/* 廣播 */
.broadcast {
  width: 100%;
  height: 50vw;
  position: relative;
}

.broadcast img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

/* 廣播區中間下方按鈕 */
.changeBroadcastIndex {
  position: absolute;
  bottom: 0;
  left: 30vw;
}

.setBroadcastIndex {
  display: inline-block;
  height: 5vw;
  width: 5vw;
  border-radius: 50%;
  border: 1px solid hsl(353, 100%, 29.2%);
}

.setBroadcastIndex:hover {
  background-color: hsl(353, 100%, 50%);
}

.setBroadcastIndex:active {
  background-color: hsl(353, 0%, 100%);
}

.setBroadcastIndex+.setBroadcastIndex {
  margin-left: 1vw;
}
</style>