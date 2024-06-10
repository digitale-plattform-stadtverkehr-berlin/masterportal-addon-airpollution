<script>

import {mapGetters} from "vuex";
import AirpollutionBarChart from "./AirpollutionBarChart.vue";

export default {
    name: "Airpollution",
    components: {
        // AirpollutionLineChart,
        AirpollutionBarChart
    },
    props: {
        feature: {
            type: Object,
            required: true
        }
    },
    computed: {
        ...mapGetters("Language", ["currentLocale"]),
        /**
         * Get the configured gfiParams.
         * @returns {Object} The gfiParams.
         */
        gfiParams: function () {
            return this.feature.getTheme()?.params;
        },
        title: function () {
            const titleField = this.gfiParams.titleField,
                detailField = this.gfiParams.detailField,
                detailLabel = this.gfiParams.detailLabel;
            let title = "";

            if (titleField && this.feature.getMappedProperties()[titleField]) {
                title = this.feature.getMappedProperties()[titleField];
            }
            if (detailField && this.feature.getMappedProperties()[detailField]) {
                title += " (";
                if (detailLabel) {
                    title += detailLabel + ": ";
                }
                title += this.feature.getMappedProperties()[detailField] + ")";
            }
            return title;
        }
    },
    watch: {
        // When the gfi window switched with arrow, the connection will be refreshed
        feature: {
            handler () {
                this.filterProperties();
                this.setContentStyle();
            },
            immediate: true
        },
        // language is switched
        currentLocale: function (newVal, oldVal) {
            if (oldVal) {
                this.dataset = [];
                this.filterProperties();
            }
        }
    },
    mounted () {
        this.setContentStyle();
    },
    methods: {
        /**
         * Parses the mapped properties of gfi into several variables for the graphics and for the info tab.
         * @returns {void}
         */
        filterProperties () {
            const allProperties = this.feature.getMappedProperties();

            this.dataset = Object.entries(JSON.parse(allProperties[this.gfiParams.datastream]));
        },

        /**
         * Setting the gfi content max width the same as graph
         * @returns {void}
         */
        setContentStyle () {
            if (document.getElementsByClassName("gfi-content").length) {
                document.getElementsByClassName("gfi-content")[0].style.maxWidth = "880px";
            }
        },
        /**
         * Reacts on click on download button. Opens the downloadLink.
         * @param {Object} evt the dedicated event
         * @returns {void}
         */
        // onClick (evt) {
        //     evt.stopPropagation();
        //     window.open(this.downloadLink);
        // },

        /**
        * @param {String} dateString a Zulu time date string.
        * @returns {String} The date and time for Germany: 'DD.MM.YYYY, hh:mm'
        */
        formatFullDate: function (dateString) {
            const date = new Date(dateString),
                options = {day: "2-digit", month: "2-digit", year: "numeric", hour: "2-digit", minute: "2-digit", timeZone: "Europe/Berlin"};

            return date.toLocaleDateString("de-DE", options);
        }
    }
};
</script>

<template>
    <div class="airpollution">
        <div class="card header">
            <strong>
                {{ title }}
            </strong>
            <br>
            <small>zuletzt aktualisiert:{{ formatFullDate(feature.getMappedProperties().update_time) }}</small>
        </div>
        <div class="chart-content">
            <AirpollutionBarChart
                :uom="gfiParams.uom"
                :dataset="dataset"
                :threshold="gfiParams.threshold"
                :type="String('diagram')"
            />
        </div>
    </div>
</template>

<style lang="scss" scoped>
@import "~/css/mixins.scss";

.airpollution {
    overflow-x: auto;

    box-sizing: border-box;
    padding: 5px 20px 5px 20px;

    @media (max-width: 767px) {
        width: inherit;
        height: inherit;
        padding-left: 10px;
        padding-right: 10px;

        div.graph {
            width: inherit;
            height: inherit;
        }
    }

    @media (min-width: 768px) {
        width: 80vw;
        height: 60vh;
    }

    @media (min-width: 1024px) {
        width: 50vw;
        height: 60vh;
    }

    .header {
        text-align: center;
        margin-bottom: 20px;
    }

    .nav-pills {
        padding: 6px;

        @include active-pill(0.9375em, 1em);
    }

    .chart-content {
        width: 100%;
        padding: 0 5px 5px 5px;
    }

    .downloadButton {
        padding: 6px;

        button {
            outline: none;
        }
    }

    .bootstrap-icon {
        padding-right: 5px;
    }

}
</style>
