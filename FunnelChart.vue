<template>
  <div>
    <div class="db-wg-spinner mt-5" v-if="loading">
      Loading ...
    </div>
    <div class="funnel-container" v-if="!loading">
      <svg width="100%" :height="chartHeight">
        <svg:style>
          .value-label {
            font-size: 21px;
          }
          .label-percentage {
            font-size: 21px;
          }
        </svg:style>
        <g
          v-for="(level, index) in funnelLevels"
          :key="index"
          @mouseover="showTooltip(index)"
          @mouseleave="hideTooltip()"
        >
          <path :fill="index == (funnelLevels.length - 1) ? last_color : colors[index]" :d="level.path" />
          <!-- Labels and Values inside the funnel -->
          <text
            :x="level.textX - 7"
            :y="level.textY"
            font-size="14px"
            fill="#000"
            text-anchor="middle"
            dominant-baseline="middle"
            pointer-events="none"
          >
            <tspan class="value-label">
              Ilość {{ level.count ?? 0 }}
            </tspan>
          </text>
          <text
            :x="level.textX - 8"
            :y="level.textY - 1"
            font-size="14px"
            fill="#FFF"
            text-anchor="middle"
            dominant-baseline="middle"
            pointer-events="none"
          >
            <tspan class="value-label">
              Ilość {{ level.count ?? 0 }}
            </tspan>
          </text>
          <!-- Tooltip -->
          <g v-if="isTooltipVisible && tooltipIndex === index">
            <rect
              :x="level.tooltipX"
              :y="level.tooltipY"
              :width="level.tooltipWidth"
              :height="level.tooltipHeight"
              fill="#000"
              opacity="0.8"
              rx="5"
              ry="5"
            />
            <text
              :x="level.tooltipTextX"
              :y="level.tooltipTextY"
              font-size="14px"
              fill="#FFF"
              text-anchor="middle"
              dominant-baseline="middle"
            >
              <tspan v-for="(label, idx) in level.labels" :key="idx" class="value-label-tooltip">
                {{ label }}: {{ level.values[idx] }}
              </tspan>
            </text>
          </g>
        </g>
        <!-- Stage names outside the funnel -->
        <text
          v-for="(level, index) in funnelLevels"
          :key="index"
          :x="10"
          :y="level.textY"
          font-size="14px"
          fill="#000"
          text-anchor="start"
          dominant-baseline="middle"
          pointer-events="none"
        >
          {{ level.stage }} ({{ level.totalValue }})
          {{ maxTotalValue ? getPercentage(level.totalValue, maxTotalValue) : '0%' }}
        </text>
      </svg>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      isTooltipVisible: false,
      tooltipIndex: null,
      chartWidth: 840, // Width of the chart
      chartHeight: 800, // Height of the chart
      funnelData: [], // Data to generate the funnel levels
      funnelLevels: [], // Array to store the levels of the funnel
      maxTotalValue: 0, // Maximum total value among all funnel stages
      last_color: "#00c9a7",
      colors: [
        "#F5EFFF",
        "#CDC1FF",
        "#A594F9",
        "#7371FC",
        "#18357b"
      ],
    };
  },
  created() {
    // this.loading = true;
    this.funnelData = [
      { "stage": "New", "count": 4, "labels": ["Webiste", "Email", "Browser"], "values": [ 500, 210, 600 ] },
      { "stage": "Follow Up", "count": 3, "labels": ["Webiste", "Email", "Browser"], "values": [ 200, 100, 20 ] },
      { "stage": "Prospect", "count": 1, "labels": ["Webiste", "Email", "Browser"], "values": [ 0, 10, 0 ] },
      { "stage": "Negotiation", "count": 2, "labels": ["Webiste", "Email", "Browser"], "values": [ 50, 200, 10 ] },
      { "stage": "won", "count": 2, "labels": ["Webiste", "Email", "Browser"], "values": [ 10, 20, 0 ] }
    ];
    this.generateFunnelLevels(); // Generate funnel levels based on data
    this.calculateMaxTotalValue(); // Calculate the maximum total value
  },
  methods: {
    generateFunnelLevels() {
      const total = this.funnelData.reduce(
        (acc, value) =>
          acc + value.values.reduce((sum, val) => sum + val, 0),
        0
      ); // Calculate the total value of the data
      const baseWidth = this.chartWidth * 0.8; // Width of the base of the funnel
      const levelCount = this.funnelData.length; // Number of levels in the funnel

      const levelHeight = this.chartHeight / levelCount; // Height of each level

      let currentY = 0;
      for (let i = 0; i < levelCount; i++) {
        const values = this.funnelData[i].values;
        const labels = this.funnelData[i].labels;
        const totalValue = this.getTotalValue(values);

        const topWidth = baseWidth * (1 - currentY / this.chartHeight); // Width of the top of the current level
        const bottomWidth =
          baseWidth * (1 - (currentY + levelHeight) / this.chartHeight); // Width of the bottom of the current level
        var yX = currentY + levelHeight; // Current Y plus level height

        // Rectangle in last
        if (i == levelCount - 1) {
          yX = yX * 4;
        }

        const path = `M${baseWidth - topWidth / 2},${currentY} L${baseWidth +
          topWidth / 2},${currentY} L${baseWidth +
          bottomWidth / 2},${yX} L${baseWidth - bottomWidth / 2},${yX} Z`;
        // Construct the path for the level

        const { color, stage } = this.funnelData[i];

        const textX = baseWidth + 10;
        const textY = currentY + levelHeight / 2;

        const tooltipX = 480;
        const tooltipY = currentY + (levelHeight / 2) - 20;
        const tooltipWidth = 400;
        const tooltipHeight = 40;
        const tooltipTextX = baseWidth;
        const tooltipTextY = currentY + levelHeight / 2;

        this.funnelLevels.push({
          path,
          color,
          stage,
          values,
          labels,
          totalValue,
          stageColor: color,
          textX,
          textY,
          tooltipX,
          tooltipY,
          tooltipWidth,
          tooltipHeight,
          tooltipTextX,
          tooltipTextY,
        });

        currentY += levelHeight;
      }
    },
    getTotalValue(values) {
      return values.reduce((sum, val) => sum + val, 0);
    },
    calculateMaxTotalValue() {
      this.maxTotalValue = Math.max(
        ...this.funnelData.map((data) => this.getTotalValue(data.values))
      );
    },
    getPercentage(totalValue, maxTotalValue) {
      return ((totalValue / maxTotalValue) * 100).toFixed(2);
    },
    showTooltip(index) {
      this.isTooltipVisible = true;
      this.tooltipIndex = index;
    },
    hideTooltip() {
      this.isTooltipVisible = false;
      this.tooltipIndex = null;
    },
  },
};
</script>

<style scoped>
.funnel-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}

div {
  text-align: center;
}

.total-value {
  font-size: 16px;
  fill: #fff;
  text-anchor: middle;
  dominant-baseline: middle;
}

.stage-name {
  font-size: 14px;
  fill: #000;
  text-anchor: middle;
  dominant-baseline: middle;
}
</style>