<script>
import Chart from "chart.js/auto";
import thousandsSeparator from "../../../../src/shared/js/utils/thousandsSeparator.js";

export default {
    name: "AirpollutionBarChart",
    components: {},
    props: {
        uom: {
            type: String,
            required: true
        },
        dataset: {
            type: Array,
            required: true
        },
        threshold: {
            type: Array,
            required: true
        }
    },
    data () {
        return {
            defaultFontFamily: "'MasterPortalFont', 'Arial Narrow', 'Arial', 'sans-serif'",
            defaultFontColor: "#333333",
            chartType: "bar",

            chartColorCircle: "rgba(70, 130, 180, 1)",
            chartRadiusCircle: 5,

            chartColorRect: "rgba(255, 0, 0, 1)",
            chartRadiusRect: 6,
            chartPointStyleRect: "rect",

            toolTipBodyFontColor: "rgba(85, 85, 85, 1)",
            toolTipBackgroundColor: "rgba(240, 240, 240, 1)",
            toolTipTitleColor: "rgba(180, 180, 180, 1)",

            scaleGridLinesColor: "rgba(0, 0, 0, 1)"
        };
    },
    watch: {
        dataset () {
            this.drawChart();
        }
    },
    mounted () {
        this.drawChart();
    },
    methods: {
        /**
         * Creates the line chart with chartsJs.
         * If a chart is already drawn, it will be destroyed.
         * @returns {void}
         */
        drawChart: function () {
            const ctx = this.$el.getElementsByTagName("canvas")[0];

            if (this.chart instanceof Chart) {
                this.chart.destroy();
            }

            this.chart = new Chart(ctx, {
                type: this.chartType,
                data: this.createChartData(),
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: this.createChartTooltip()
                    },
                    scales: this.createChartScales(),
                    defaultFontFamily: this.defaultFontFamily,
                    defaultFontColor: this.defaultFontColor
                }
            });
        },

        /**
         * Formats a timestamp to the format YYYY-MM-DDTHH:00:00Z
         * (i.e. sets the minutes and the seconds to 0)
         * @param {string} timestamp - The timestamp
         * @returns {string} The formatted timestamp.
         */
        formatTimestamp (timestamp) {
            const date = new Date(timestamp);

            date.setMinutes(0);
            date.setSeconds(0);

            return date.toISOString().slice(0, 16) + "00:00Z";
        },

        /**
         * Checks if the provided timestamp is the same as the current formatted time.
         * @param {string} timestamp - The timestamp to be compared.
         * @returns {boolean} True if the timestamp is the same as the formatted current time, false otherwise
         */
        isCurrentTime (timestamp) {
            const currentFormattedTime = this.formatTimestamp(new Date().toISOString());

            return currentFormattedTime === this.formatTimestamp(timestamp);
        },

        /**
         * Creates the data for the chart.
         * @returns {Object} The chart data.
         */
        createChartData: function () {

            return {
                labels: this.dataset.map(item => this.formatDate(item[0])),
                datasets: [
                    {
                        data: this.dataset.map(item => item[1]),
                        backgroundColor: this.createBackgroudColors(),
                        borderColor: this.dataset.map(item => this.isCurrentTime(item[0]) ? "black" : "transparent"),
                        borderWidth: this.dataset.map(item => ({
                            top: this.isCurrentTime(item[0]) ? 1.5 : 0.2,
                            right: this.isCurrentTime(item[0]) ? 1.5 : 0.2,
                            bottom: 0,
                            left: this.isCurrentTime(item[0]) ? 1.5 : 0.2
                        })),
                        categoryPercentage: 1.0,
                        barPercentage: 0.83,
                        barThickness: this.dataset.map(item => this.isCurrentTime(item[0]) ? 9 : "flex")
                    }
                ]
            };
        },


        /**
        * @param {String} dateString a Zulu time date string.
        * @returns {String} The date and time for Germany: 'ddd, DD.MM., hh' and a trailing 'h'
        */
        formatDate: function (dateString) {
            const date = new Date(dateString),
                options = {weekday: "short", day: "2-digit", month: "2-digit", hour: "numeric", minute: "2-digit", timeZone: "Europe/Berlin"};

            let formattedDate = date.toLocaleDateString("de-DE", options).replace(/:00$/, "");

            // remove the leading zero when time < 10:00
            if (date.getHours() < 10) {
                formattedDate = formattedDate.replace(/\b0(\d)\b/, "$1");
            }
            // Remove the commay
            formattedDate = formattedDate.replace(/,/g, "");
            return formattedDate + "h";
        },


        /** Prepares the backgroundColors for use in the chart
         * @returns {String[]} The array with the backgroundColors.
         */
        createBackgroudColors: function () {
            const threshold = this.threshold;

            return this.dataset.map(item => {
                for (let i = 0; i < threshold.length - 1; i++) {
                    if (item[1] <= threshold[i + 1].value) {
                        return threshold[i].color;
                    }
                }
                return threshold[threshold.length - 1].color;
            });
        },

        /**
         * Creates the tooltip for the chart.
         * @returns {Object} The chart tooltip.
         */
        createChartTooltip: function () {
            return {
                bodyColor: this.toolTipBodyFontColor,
                backgroundColor: this.toolTipBackgroundColor,
                titleColor: this.toolTipTitleColor,
                titleAlign: "center",
                bodyAlign: "center",
                displayColors: false,
                callbacks: {
                    title: (tooltipItem) => tooltipItem[0].xLabel,
                    afterLabel: (tooltipItem) => this.getAirQualityRating(tooltipItem.raw),
                    label: (tooltipItem) => thousandsSeparator(tooltipItem.raw) + " " + this.uom
                }
            };
        },

        /** Return the rating of the air quality for a specific value
         * @param {Number} airQuality The measured value of an air parameter
         * @returns {String} The rating of the air quality as a string.
         */
        getAirQualityRating: function (airQuality) {
            const threshold = this.threshold;

            for (let i = 0; i < threshold.length - 1; i++) {
                if (airQuality <= threshold[i + 1].value) {
                    return threshold[i].rating;
                }
            }
            return threshold[threshold.length - 1].rating;
        },

        /**
         * Creates the scales for the chart.
         * @param {Number} maxValue The max value for the y-axis.
         * @returns {Object} The chart scales.
         */
        createChartScales: function () {
            return {
                x: {
                    title: {
                        display: true,
                        text: "Zeit" // hard coded!
                    },
                    ticks: {
                        maxTicksLimit: 16,
                        fontSize: 11
                    },
                    grid: this.createGridLines()
                },
                y: {
                    title: {
                        display: true,
                        text: this.uom
                    },
                    ticks: {
                        beginAtZero: true,
                        precision: 0,
                        fontSize: 11,
                        callback: value => thousandsSeparator(value)
                    },
                    grid: this.createGridLines()
                }
            };
        },

        /**
         * Creates the gridLines for scales of the chart.
         * @returns {Object} The gridLines.
         */
        createGridLines: function () {
            return {
                color: this.scaleGridLinesColor,
                display: true,
                drawBorder: true,
                drawOnChartArea: false
            };
        }
    }
};
</script>


<template>
    <div
        v-if="dataset"
        id="airpollution-bar-chart"
    >
        <div id="airpollution-chart-container">
            <canvas />
        </div>
    </div>
</template>

<style lang="scss">
@import "~variables";

#mp-menu-secondaryMenu .airpollution {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    .chart-content {
        flex-grow: 0;
        flex-shrink: 1;
        #airpollution-bar-chart canvas {
            width: 100% !important;
            height: 100% !important;
        }
    }
}
</style>
