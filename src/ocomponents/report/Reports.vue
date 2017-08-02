<template>
    <div>
            <div slot="fristbox">
                 <yl-toolbar>
                    <el-button-group ref="funtoolbar">
                            <el-button v-for="(item,index) in funBtnConf" 
                                :type="item.type" 
                                :name="item.name" 
                                :size="item.size" 
                                v-show="item.isShow"
                                :disabled="item.disabled"  
                                @click="item.clickEvent" >
                                <i :class="item.icon"></i>{{item.text}}</el-button>
                    </el-button-group>
                  </yl-toolbar>
            </div>
            <div slot="secondbox" class="flexbox">
                <yl-table ref="table" 
                        @reload="_reload" 
                        @current-change="_currentChange" 
                        :configs="tableConfig" 
                        :input="mainInput.inputModel" 
                        :tableloading="mainTableLoading">
                             <template slot="isAudit" scope="scope">
                                <el-tag type="primary" v-if="scope.row.isAudit==1">已审核</el-tag>
                                <el-tag type="danger" v-else-if="scope.row.isAudit==0">未审核</el-tag>
                                <div  v-else></div>
                            </template> 
                             <template slot="mon" scope="scope">
                                <el-button v-if="scope.row.mon!='合计'" @click="_currentDetail" type="text" :value="scope.row.mon" size="small">{{scope.row.mon}}</el-button>
                                <div  v-else>{{scope.row.mon}}</div>
                            </template>  
                </yl-table>
            </div>
       </div> 
</template>

 <script type="text/babel">
import  {inputModel} from 'api/inputmodel';
import {requestExecuteSqlAndProcCommand} from 'api/sqlexecute'

export default {
    data(){
        return{
            mainTableLoading:false,
            tableData:{},
            mainInput:new inputModel(),
        }
    },
    props:{
        requestParms:{
            type:Object,
            default:{}
        },
        tcolomns:{
            type:Array
        },
        titleName:{
            type:String
        },
        funBtnConf:{
            type:Object,
            default:{}
        }
    },
    computed:{
         tableConfig: {
                    get () {
                        return {
                            table: {
                                attr: {
                                    data: this.tableData,
                                    highlightCurrent:true,
                                }
                            },
                            columns:this.tcolomns,
                        }
                    }
                }
                
    },
    methods:{
        _getReportList(){
            let _this=this;
            let params={};
            params=this.requestParms;
            let obj={};
            requestExecuteSqlAndProcCommand(params).then(data=>{
                if(data.success){
                    obj.data=data.result.items[0];
                    obj.draw=1;
                    obj.recordsFiltered=obj.data.length;
                    obj.recordsTotal=obj.data.length;
                    this.tableData=obj;
                }else{
                    this.$message.error('失败！'+data.error.message);
                }
                this.mainTableLoading=false;
            }).catch(function(error){
                       _this.mainTableLoading=false;
                });
            
        },
        _reload(){
            this._getReportList();
        },
        _currentDetail(val){
            this.$emit('currentDetail'); 
        },
        _currentChange(row){
            this.$emit('currentChange',row); 
        }
    },
    mounted(){
        this._reload();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
