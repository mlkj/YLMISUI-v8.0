<template>
    <div>
        <yl-table ref="table"
            @reload="_reload"
            @current-change="_currentChange"
            :configs="tableConfig" 
            :tableloading="mainTableLoading">
        </yl-table>
    </div>
</template>

<script type="text/babel">
export default {
    data(){
        mainTableLoading:false
    },
    props:{
        _getReportList(){

        },
        _reload(){
            this._getReportList();
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
