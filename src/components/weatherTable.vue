<template>
    <div>
        <table  style="width:100%">
            <tr>
                <th>Date</th>
                <th>Precipitation</th>
                <th colspan="3">Temperature</th>
            </tr>
            <tr v-for="(dates, index) in tableData.dates">
                <td>{{dates}}</td>
                <td>{{tableData.data[1].values.sum[index]}}</td>
                <td>{{tableData.data[5].values.min[index]}}</td>
                <td>{{tableData.data[5].values.avg[index]}}</td>
                <td>{{tableData.data[5].values.max[index]}}</td>
            </tr>
        </table>
    </div>
</template>
<script>
    export default {
        props: ["tableData"],
        name:'weatherTable',
        data() {
            return{}
        },
        mounted () {
        }
    }
</script>
<style>

</style>
