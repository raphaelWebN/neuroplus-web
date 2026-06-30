<template>
  <div>
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script>
import { ref, watch, onMounted, onUnmounted } from "vue";
import { Chart, registerables } from "chart.js";
import ChartDataLabels from "chartjs-plugin-datalabels";

Chart.register(...registerables, ChartDataLabels);

export default {
  props: {
    bioageData: {
      type: Array,
      required: true,
    },
  },
  setup(props) {
    const chartCanvas = ref(null);
    let chartInstance = null;

    // 初始化圖表
    const initChart = () => {
      if (!chartCanvas.value || !props.bioageData.length) return;

      const ctx = chartCanvas.value.getContext("2d");

      // 格式化日期為 "MM/DD"
      const labels = props.bioageData.map((item) => {
        const dateParts = item.DateTime.split(" ")[0].split("/");
        return `${dateParts[1]}/${dateParts[2]}`;
      });

      const dataset1 = props.bioageData.map((item) => parseInt(item.BcBioage, 10) || 0);
      const dataset2 = props.bioageData.map((item) => parseInt(item.AfBioage, 10) || 0);
      const difference = dataset2.map((val, index) => val - dataset1[index]);

      // 如果圖表已存在，先銷毀再重建
      if (chartInstance) {
        chartInstance.destroy();
      }

      chartInstance = new Chart(ctx, {
        type: "bar",
        data: {
          labels: labels,
          datasets: [
            {
              label: "改善幅度",
              data: difference,
              type: "line",
              borderColor: "#FEC37D",
              backgroundColor: "#FEC37D",
              pointBackgroundColor: "#FFFFFF",
              pointBorderColor: "#FEC37D",
              pointRadius: 5,
              pointBorderWidth: 2,
              fill: false,
              tension: 0.3,
              datalabels: {
                display: false,
              },
            },
            {
              label: "使用前",
              data: dataset1,
              backgroundColor: "#74BC1F",
              borderColor: "#74BC1F",
              borderWidth: 1,
              datalabels: {
                color: "#FFFFFF",
                font: { size: 10 },
                offset: 2,
              },
            },
            {
              label: "使用后",
              data: dataset2,
              backgroundColor: "#1FBCB3",
              borderColor: "#1FBCB3",
              borderWidth: 1,
              datalabels: {
                color: "#FFFFFF",
                font: { size: 10 },
                offset: 2,
              },
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            tooltip: {
              mode: "index",
              intersect: false,
            },
            legend: {
              display: true,
              position: "bottom",
              labels: {
                usePointStyle: true,
                pointStyle: "circle",
                boxWidth: 12,
                boxHeight: 12,
                padding: 20,
              },
            },
            datalabels: {
              display: true,
              anchor: "end",
              align: "start",
            },
          },
          scales: {
            y: {
              type: "linear",
              position: "left",
              ticks: {
                stepSize: 15,
                callback: (value) => (value % 15 === 0 ? value : null),
              },
            },
            x: {
              grid: {
                drawOnChartArea: false,
              },
            },
          },
        },
      });
    };

    // 銷毀圖表
    const destroyChart = () => {
      if (chartInstance) {
        chartInstance.destroy();
        chartInstance = null;
      }
    };

    // 監聽 bioageData 的變化
    watch(
      () => props.bioageData,
      (newData) => {
        if (newData.length) {
          initChart();
        }
      },
      { immediate: true } // 初始化時立即執行一次
    );

    // 註冊生命周期
    onMounted(() => {
      initChart();
    });

    onUnmounted(() => {
      destroyChart();
    });

    return {
      chartCanvas,
    };
  },
};
</script>

<style>
canvas {
  max-width: 100%;
}
</style>
