<template>
    <div slot="content" >
            <div slot="fristbox">
                 <yl-toolbar>
                    <el-form  :inline="true"  >
                        <el-form-item class="form-content-vertical">
                            <el-date-picker  
                                v-model="searchModel.beginDate" 
                                type="month" 
                                style="width:100px"
                                :editable="false"
                                size="small"  
                                placeholder="选择起始月份">
                            </el-date-picker> 
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-date-picker  
                                v-model="searchModel.endDate" 
                                type="month" 
                                style="width:100px"
                                :editable="false"
                                size="small"  
                                placeholder="选择结束月份">
                            </el-date-picker> 
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-input  placeholder="材料名称" size="small" style="width:100px" v-model="searchModel.infoName"></el-input> 
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-input  placeholder="规格型号" size="small" style="width:100px" v-model="searchModel.infoModel"></el-input> 
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-input  placeholder="单位" size="small" style="width:100px" v-model="searchModel.infoUnit"></el-input> 
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-button type="primary" icon="search" size="small"  @click="_reload">查 询</el-button>
                        </el-form-item>
                    </el-form >
                </yl-toolbar>
            </div>
            <div slot="secondbox" style="height:500px">
                <yl-table ref="table" 
                        @reload="_reload" 
                        @current-change="_currentChange" 
                        :configs="tableConfig" 
                        :input="mainInput.inputModel" 
                        :tableloading="mainTableLoading">
                           
                </yl-table>
            </div>

    </div>
</template>

<script type="text/babel">
import {requestExecuteSqlAndProcCommand} from 'api/sqlexecute'
import{inputModel} from  'api/inputmodel';
export default {
    data(){
        return{
            selectRows:[],
            mainTableLoading:false,
            tableData:{},
            mainInput:new inputModel(),
            searchModel:{
                beginDate:new Date(),
                endDate:new Date,
                infoName:'',
                infoModel:'',
                infoUnit:''
            }
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
                            columns:[ {attr: { prop: 'infoName', label: '材料名称', width:100,  } },
                                        {attr: { prop: 'infoModel', label: '规格型号', width:100,  } },
                                        {attr: { prop: 'infoUnit', label: '单位', width:100,  } }, 
                                        {attr: { prop: 'bookPrice', label: '账面价', width:100,  } },
                                        {attr: { prop: 'lastQuantity', label: '上期结存数量', width:120,  } },
                                        {attr: { prop: 'lastBookSum', label: '上期结存金额', width:120,  } },
                                        {attr: { prop: 'currRecQuantiy', label: '本期收料数量', width:120,  } },
                                        {attr: { prop: 'currRecSum', label: '本期收料金额', width:120,  } },
                                        {attr: { prop: 'currAliQuantity', label: '本期调入数量', width:120,  } },
                                        {attr: { prop: 'currAliSum', label: '本期调入金额', width:120,  } },
                                        {attr: { prop: 'currDelQuantity', label: '本期发料数量', width:120,  } },
                                        {attr: { prop: 'currDelSum', label: '本期发料金额', width:120,  } },
                                        {attr: { prop: 'currAlQuantity', label: '本期调拨数量', width:120,  } },
                                        {attr: { prop: 'currAlSum', label: '本期调拨金额', width:120,  } },
                                        {attr: { prop: 'currScpQuantity', label: '本期报废数量', width:120,  } },
                                        {attr: { prop: 'currScpSum', label: '本期报废金额', width:120,  } },
                                        {attr: { prop: 'currJcQuantity', label: '本期结存数量', width:120,  } },
                                        {attr: { prop: 'currJcSum', label: '本期结存金额', width:120,  } },
                                        {attr: { prop: 'barCode', label: '条码', width:100,  } },
                                        {attr: { prop: 'manufacturer', label: '生产厂商', width:100,  } },
                                        {attr: { prop: 'batchNo', label: '炉批号', width:100,  } },
                                        {attr: { prop: 'infoRemark', label: '备注', width:100,  } },
                                        {attr: { prop: 'storeRoom', label: '库别', width:100,  } },
                                        ],
                        }
                    }
                }
    },
    props:{
        selectRow:{
            type:Object,
            default:{},
        },
    },
    methods:{
        _getReportList(){
            this.mainTableLoading=true;
            let _this=this;
            let params={};
            this.searchModel.beginDate=this.formatDate(this.searchModel.beginDate,"YYYY-MM");
            this.searchModel.endDate=this.formatDate(this.searchModel.endDate,"YYYY-MM");
            //参数设置
            params= {
                procType:0,
                firstKeys:"OrgId,BeginDate,EndDate",
                firstValues:this.getUserInfo().user.manageOrgId+","+this.searchModel.beginDate+","+this.searchModel.endDate,
                secondKeys:"InfoName,InfoModel,InfoUnit",
                secondOperates:"Like,Like,Like",
                secondValues:"'%"+this.searchModel.infoName+"%','%"+this.searchModel.infoModel+"%','%"+this.searchModel.infoUnit+"%'",
                procName:'Report_MaterialDymaticDetails'
            };
            //执行通用存储过程获取表格数据
            this.commonSqlExcute(params,(val)=>{
                this.tableData=val
                this.mainTableLoading=false;
            });
        },
        _reload(){
            this._getReportList();
        },
        _currentChange(val){
            // console.log(val);
            if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }
        },
    },
    mounted(){
        this.searchModel.beginDate=this.selectRow.mon;
        this.searchModel.endDate=this.selectRow.mon;
        this._getReportList();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
