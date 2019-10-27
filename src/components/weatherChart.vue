
<script>
    import { Line, mixins } from 'vue-chartjs'
    // chart.js ordinarily doesn't provide an option for an automatic update whenever a dataset changes
    // but that can be done in vue-chartjs by using mixins (reactiveProp)
    const { reactiveProp } = mixins;
    export default {
        extends: Line,
        mixins: [mixins.reactiveProp],
        props: ["data", "options"],
        data() {
            return{
            }
        },
        mounted () {
            this.renderChart(this.data)
        }
    }
</script>
<style>

</style>
