<script setup>
import { ref, onMounted, onUnmounted } from "vue";

// 目前位置的經緯度
const currentLocation = ref({ lat: 0, lng: 0 });

/*if ("geolocation" in navigator) {
	// geolocation is available 
	navigator.geolocation.getCurrentPosition((position) => {
		currentLocation.value = {
	  lat: position.coords.latitude,
	  lng: position.coords.longitude,
		};
	});
	//創建一個窗口顯示目前的位置
	const arWindow = document.createElement("div");
	arWindow.style.position = "absolute";
	arWindow.style.top = "50%";
	arWindow.style.left = "50%";
	arWindow.style.transform = "translate(-50%, -50%)";
	arWindow.style.backgroundColor = "rgba(255, 255, 255, 0.8)";
} else {
	// geolocation IS NOT available 
}*/

// AR視窗相關的資料
const arWindow = ref(null);
const camera = ref(null);
const nearbyLocations = ref([
	{ lat: 25.033964, lng: 121.564472 }, // 台北101
	{ lat: 25.041689, lng: 121.549537 }, // 國立故宮博物院
]);


// 計算目前位置與附近位置的距離
function calculateDistance(location) {
	const lat1 = currentLocation.value.lat;
	const lng1 = currentLocation.value.lng;
	const lat2 = location.lat;
	const lng2 = location.lng;

	// 使用 Haversine 公式計算距離
	const R = 6371; // 地球半徑 (單位：公里)
	const dLat = (lat2 - lat1) * (Math.PI / 180);
	const dLng = (lng2 - lng1) * (Math.PI / 180);
	const a =
		Math.sin(dLat / 2) * Math.sin(dLat / 2) +
		Math.cos(lat1 * (Math.PI / 180)) *
			Math.cos(lat2 * (Math.PI / 180)) *
			Math.sin(dLng / 2) *
			Math.sin(dLng / 2);
	const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
	const distance = R * c;

	return distance.toFixed(2); // 四捨五入到小數點後兩位
}

// 監聽裝置的方向變化
function handleDeviceOrientation(event) {
	const { alpha } = event; // 裝置的方向角度 (單位：度)
	const threshold = 10; // 方向對準的閾值 (單位：度)

	// 檢查是否對準了某個附近的位置
	for (const location of nearbyLocations.value) {
		const direction = Math.abs(
			alpha - calculateBearing(currentLocation.value, location)
		);
		if (direction <= threshold) {
			const distance = calculateDistance(location);
			const content = `經度：${location.lat}，緯度：${location.lng}\n距離：${distance} 公里`;
			showARContent(content);
			break;
		}
	}
}

// 顯示AR內容
function showARContent(content) {
	const arWindowElement = arWindow.value;
	if (arWindowElement) {
		arWindowElement.innerHTML = content;
	}
}

// 計算兩個經緯度之間的方位角度
function calculateBearing(location1, location2) {
	const lat1 = location1.lat;
	const lng1 = location1.lng;
	const lat2 = location2.lat;
	const lng2 = location2.lng;

	const dLng = (lng2 - lng1) * (Math.PI / 180);
	const y = Math.sin(dLng) * Math.cos(lat2 * (Math.PI / 180));
	const x =
		Math.cos(lat1 * (Math.PI / 180)) * Math.sin(lat2 * (Math.PI / 180)) -
		Math.sin(lat1 * (Math.PI / 180)) *
			Math.cos(lat2 * (Math.PI / 180)) *
			Math.cos(dLng);

	const bearing = Math.atan2(y, x) * (180 / Math.PI);

	return (bearing + 360) % 360; // 轉換為 0-360 的角度範圍
}

// 啟動攝影機
async function startCamera() {
	try {
		const stream = await navigator.mediaDevices.getUserMedia({
			video: true,
		});
		camera.value.srcObject = stream;
	} catch (error) {
		console.error("無法開啟攝影機：", error);
	}
}

// 初始化
onMounted(() => {
	startCamera();
	window.addEventListener("deviceorientation", handleDeviceOrientation);
});

// 在組件被卸載時清除事件監聽
onUnmounted(() => {
	window.removeEventListener("deviceorientation", handleDeviceOrientation);
});
</script>

<template>
  <div>
    <video
      ref="camera"
      autoplay
    >
      <div ref="arWindow" />
    </video>
  </div>
</template>
