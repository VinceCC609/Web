<template>
  <div class="menu">
    <ul class="menu-list">
      <li class="menu-item" v-for="item in menuItems" :key="item.name" @click="selectItem(item)">
        <a href="#" class="menu-link">{{ item.name }}</a>
        <ul v-if="item.submenu" class="submenu">
          <li v-for="subItem in item.submenu" :key="subItem.name" @click.stop="selectItem(subItem)" class="submenu-item">
            <a href="#" class="submenu-link">{{ subItem.name }}</a>
          </li>
        </ul>
      </li>
    </ul>
  </div>

  <div class="image-display">
    <img :src="selectedImage" alt="Selected Item" v-if="selectedImage" />
  </div>

  <!-- 顯示圓餅圖 -->
  <div v-if="selectedAreaChartData" class="chart-container">
    <h2>台北市的各區面積分布</h2>
    <div class="pie-chart" :style="pieChartStyle">
      <div class="pie-chart-legend">
        <div style="width: 500px;" v-for="(area, index) in areas" :key="area.name" :style="{ color: getColorByIndex(index) }">
            {{ area.name }} ({{ area.area }} km²)
        </div>
      </div>
    </div>
    <!-- 顯示各區名稱與顏色，並將其顯示在右邊 -->

  </div>

  <!-- 顯示長條圖 -->
  <div v-if="selectedAreaBarChartData" style="text-align: center;">
    <h2>台北市各區面積</h2>
    <div class="bar-chart">
      <div
        class="bar"
        v-for="(area, index) in areas"
        :key="area.name"
        :style="{
          height: area.area * 10 + 'px',
          backgroundColor: getColorByIndex(index)
        }"
      >
        <div>{{ area.name }} ({{ area.area }} km²)</div>
      </div>
    </div>
  </div>

  <!-- 顯示TabControl -->
  <div class="tab-container" v-if="tabAppear">
    <!-- Tab 頭 -->
    <div class="tabs">
      <div
        class="tab"
        v-for="(tab, index) in tabs"
        :key="tab.name"
        :class="{ active: selectedTab === index }"
        @click="selectTab(index)"
      >
        {{ tab.name }}
      </div>
    </div>

    <!-- Tab 內容 -->
    <div v-for="(tab, index) in tabs" :key="tab.name" class="tab-content" v-show="selectedTab === index">
      {{ tab.content }}
    </div>
  </div>

  <!-- 飲品訂單-->
  <div v-if="teaOrder">
    <div class="checkbox-container">
      <label>
        珍珠奶茶 (NT$ 50)
        <input type="number" v-model.number="orderDetails.pearlMilkTea" min="0" />
      </label>
      <label>
        茉香奶茶 (NT$ 40)
        <input type="number" v-model.number="orderDetails.jasmineMilkTea" min="0" />
      </label>
    </div>

    <p>訂單總金額: NT$ {{ totalPrice }}</p>
    <button @click="addOrder">產生訂單</button>

    <div class="order-list">
      <h3>訂單列表</h3>
      <div v-for="(order, index) in orders" :key="index" class="order-item">
        <div> {{ order.description }} - NT$ {{ order.totalPrice }} </div>
        <button @click="deleteOrder(index)" style="font-size:12px;">刪除</button>
      </div>
    </div>
  </div>
  <div class="container-move" v-if="parallelogram">
    <div class="parallelogram left-to-right"></div>
    <div class="parallelogram right-to-left"></div>
  </div>
</template>

<script>

export default ({
  name: 'HorizontalMenu',
  data () {
    return {
      parallelogram: false,
      menuItems: [
        {
          name: '選項',
          submenu: [
            { name: 'TabControl', tabAppear: true },
            { name: '奶茶訂單', teaOrder: true },
            { name: '移動', parallelogram: true }
          ]
        },
        {
          name: '聖獸',
          submenu: [
            { name: '朱雀', image: './image/0.jpg' },
            { name: '玄武', image: './image/1.jpg' },
            { name: '青龍', image: './image/2.jpg' },
            { name: '白虎', image: './image/3.jpg' }
          ]
        },
        {
          name: '面積圖',
          submenu: [
            { name: '圓餅圖', selectedArea: 'chartData' },
            { name: '長條圖', selectedArea: 'barChartData' }
          ]
        },
        { name: '聯繫我們', image: './image/7.jpg' }
      ],
      tabAppear: false,
      selectedTab: 0,
      tabs: [
        { name: 'Tab 1', content: '這是Tab 1的內容' },
        { name: 'Tab 2', content: '這是Tab 2的內容' },
        { name: 'Tab 3', content: '這是Tab 3的內容' }
      ],
      tabContent: '',
      selectedImage: '',
      selectedAreaChartData: '',
      selectedAreaBarChartData: '',
      areas: [
        { name: '中正區', area: 2.6 },
        { name: '大同區', area: 2.5 },
        { name: '中山區', area: 4.3 },
        { name: '松山區', area: 11.2 },
        { name: '大安區', area: 14.3 },
        { name: '萬華區', area: 7.4 },
        { name: '信義區', area: 4.6 },
        { name: '士林區', area: 6.6 },
        { name: '北投區', area: 6.3 },
        { name: '內湖區', area: 19.2 },
        { name: '南港區', area: 12.1 },
        { name: '文山區', area: 13.5 }
      ],
      // 每種飲品的數量
      orderDetails: {
        pearlMilkTea: 0,
        jasmineMilkTea: 0
      },
      // 儲存訂單
      orders: [],
      // 飲品價格
      drinkPrices: {
        pearlMilkTea: 50,
        jasmineMilkTea: 40
      },
      // 顯示奶茶訂單與否
      teaOrder: false
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      total += this.orderDetails.pearlMilkTea * this.drinkPrices.pearlMilkTea
      total += this.orderDetails.jasmineMilkTea * this.drinkPrices.jasmineMilkTea
      return total
    },
    pieChartStyle () {
      const totalArea = this.areas.reduce((sum, area) => sum + area.area, 0)
      let startAngle = 0
      const gradients = this.areas.map((area, index) => {
        const percentage = (area.area / totalArea) * 100
        const endAngle = startAngle + (percentage / 100) * 360
        const color = this.getColorByIndex(index)

        const gradient = `${color} ${startAngle}deg ${endAngle}deg`
        startAngle = endAngle
        return gradient
      })

      return {
        background: `conic-gradient(${gradients.join(', ')})`
      }
    }
  },
  methods: {
    init () {
      this.parallelogram = null
      this.teaOrder = null
      this.tabAppear = null
      this.selectedImage = null
      this.selectedAreaChartData = null
      this.selectedAreaBarChartData = null
    },
    // 產生訂單
    addOrder () {
      // 檢查是否有選擇的數量
      if (this.orderDetails.pearlMilkTea > 0 || this.orderDetails.jasmineMilkTea > 0) {
        const order = {
          description: `珍珠奶茶: ${this.orderDetails.pearlMilkTea}杯, 茉香奶茶: ${this.orderDetails.jasmineMilkTea}杯`,
          totalPrice: this.totalPrice
        }
        this.orders.push(order)
        // 清空當前的數量
        this.orderDetails.pearlMilkTea = 0
        this.orderDetails.jasmineMilkTea = 0
      } else {
        alert('請輸入訂購數量')
      }
    },
    // 刪除訂單
    deleteOrder (index) {
      this.orders.splice(index, 1)
    },
    selectTab (index) {
      this.selectedTab = index // 設置選中的 Tab
    },
    getColorByIndex (index) {
      const colors = [
        '#FF9999', '#66B3FF', '#99FF99', '#FFCC99', '#FF6666',
        '#FFB6C1', '#FF6347', '#32CD32', '#FFD700', '#8A2BE2', '#A52A2A', '#D2691E'
      ]
      return colors[index % colors.length]
    },
    selectItem (item) {
      this.init()
      if (item.image) {
        this.selectedImage = item.image
      }
      if (item.selectedArea) {
        if (item.selectedArea === 'chartData') {
          this.selectedAreaChartData = item.name
        } else if (item.selectedArea === 'barChartData') {
          this.selectedAreaBarChartData = item.name
        }
      }
      if (item.tabAppear) {
        this.tabAppear = true
      }

      if (item.teaOrder) {
        this.teaOrder = item.teaOrder
      }

      if (item.parallelogram) {
        this.parallelogram = true
      }
    }
  }
})
</script>

<style scoped>
.container-move {
  position: relative;
  width: 100vw;
  height: 100vh;
}

.parallelogram {
  position: absolute;
  width: 200px;
  height: 100px;
  background-color: #00bcd4;
  transform: skew(-20deg);
}

.left-to-right {
  left: -250px;
  top: 30%;
  animation: moveRight 3s ease-out forwards;
}

.right-to-left {
  right: -250px;
  top: 60%;
  animation: moveLeft 3s ease-out forwards;
}

@keyframes moveRight {
  to {
    left: 50%;
    transform: translateX(-50%) skew(-20deg);
  }
}

@keyframes moveLeft {
  to {
    right: 50%;
    transform: translateX(50%) skew(-20deg);
  }
}
.checkbox-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.order-list {
  margin-top: 20px;
}
.order-item {
  display: flex;
  justify-content: left;
  margin-bottom: 10px;
}
button {
  margin-bottom: 0px;
  margin-left: 20px;
  background-color: #e74c3c;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
.tab-container {
  display: flex;
  flex-direction: column;
  width: 100%; /* 父容器的寬度 */
}
/* Tabs */
.tabs {
  display: flex;
  justify-content: flex-end; /* 使Tab項目靠右 */
}
/* Tab項目 */
.tab {
  padding: 10px 20px;
  cursor: pointer;
  font-weight: bold;
  text-align: center;
  transition: border-bottom 0.3s ease;
  margin-right: 10px;
  justify-content: flex-end; /* 使Tab項目靠右 */
}

/* 只有選中的 Tab 顯示底部邊框 */
.tab.active {
  border-bottom: 3px solid blue;  /* 設定底部藍色邊框 */
}

/* Tab 內容 */
.tab-content {
  display: flex;
  width: 96%; /* 父容器的寬度 */
  padding: 20px;
  justify-content: flex-end; /* 使Tab項目靠右 */
}

/* 顯示選中的 Tab 內容 */
.tab-content.active {
  display: block;
}

.tab-content {
  padding: 20px;
}

.pie-chart {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 500px;
  height: 500px;
  border-radius: 50%;
  text-align: center;
}

.pie-chart-legend {
  text-align: left; /* 改成左對齊 */
  margin-top: 20px;
  margin-left: 1200px; /* 調整一些距離 */
  display: inline-block; /* 讓 legend 顯示在右邊 */
  vertical-align: top; /* 保證 legend 和圓餅圖對齊 */
}

.pie-chart-legend ul {
  list-style-type: none;
  padding: 0;
}

.pie-chart-legend li {
  font-size: 14px;
  margin-bottom: 5px;
}

/* 讓 chart-container 容器將圓餅圖和圖例居中顯示 */
.chart-container {
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
  flex-direction: column; /* 圓餅圖和圖例垂直排列 */
  text-align: center; /* 確保文字居中 */
}

.bar-chart {
  display: flex;
  justify-content: center;
  width: 100%;
  height: 300px;
}

.bar {
  width: 100px;
  display: inline-block;
  transition: height 0.5s;
  background-color: #4CAF50;
}

.bar div {
  text-align: center;
  color: black;
  font-size: 12px;
  margin-top: 5px;
}

.menu {
  background-color: #350ee2;
  text-align: left;
  position: relative;
}

.menu-list {
  display: flex;
  list-style-type: none;
  padding: 0px;
  margin: 0px;
  justify-content: flex-end;
}

.menu-item {
  position: relative;
  width: 100px;
  justify-content: center;
}

.menu-link {
  color: white;
  text-decoration: none;
  font-size: 16px;
  padding: 10px;
  display: block;
}

.submenu {
  position: absolute;
  top: 100%;
  left: 0;
  display: none;
  background-color: #16bce6;
  list-style-type: none;
  padding: 0;
  margin: 0;
  text-align: right;
}

.submenu-item {
  margin: 0;
}

.submenu-link {
  color: white;
  padding: 10px;
  text-decoration: none;
  display: block;
}

.menu-item:hover .submenu {
  display: block;
}

.image-display {
  margin-top: 20px;
  text-align: center;
}

.image-display img {
  max-width: 100%;
  height: auto;
}
</style>
