<template>
<yl-panelpage :titleName="'周转材料动态表'">
    <div slot="content" >
         <yl-layout> 
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
                                <el-button v-if="scope.row.mon!='合计'" @click="_currentDetail(scope.row)" type="text" :value="scope.row.mon" size="small">{{scope.row.mon}}</el-button>
                                <div  v-else>{{scope.row.mon}}</div>
                            </template>  
                </yl-table>
            </div>
     </yl-layout>
         <el-dialog ref="mainDialog" :title="currentRow.mon" v-model="addFormVisible" size="full"  top="10%" 
                    :modal-append-to-body="false" :close-on-click-modal="false"  >
                    <Detail :selectRow="currentRow" @close="_close"  v-if="addFormVisible"></Detail>
         </el-dialog>
    </div>
    </yl-panelpage>
</template>

 <script type="text/babel">
import{inputModel} from  'api/inputmodel';
import Reports from 'ocomponents/report/Reports'
import {requestExecuteSqlAndProcCommand} from 'api/sqlexecute'
import Detail from './Detail'
export default {
    data(){
        return{
            selectRows:[],
            addFormVisible:false,
            mainTableLoading:false,
            tableData:{},
            mainInput:new inputModel(),
            currentRow:{},
            funBtnConf:{
                            audit:{
                                name:'audit',
                                icon:'icon-user-plus',
                                type:'primary',
                                size:'small',
                                text:'审核',
                                disabled:true,
                                isShow:true,
                                clickEvent:"",
                                permissionSetting:""
                            },
                             unAudit:{
                                name:'unAudit',
                                icon:'icon-user-minus',
                                type:'primary',
                                size:'small',
                                text:'撤销审核',
                                disabled:true,
                                isShow:true,
                                clickEvent:"",
                                permissionSetting:""
                            },
            },
            columns1:[ 
                        {attr: { prop: 'mon', label: '月份', width:100,  scopedSlot:'mon'} },
                        {attr: { prop: 'isAudit', label: '是否审核', width:100,  scopedSlot:'isAudit', } },
                        {attr: { prop: 'lastMonthJC', label: '上期结存', width:100,  } }, 
                        {attr: { prop: 'currMonReceive', label: '本月收料', width:100,  } },
                        // {attr: { prop: 'accReceive', label: '开累收料', width:100,  } },
                        {attr: { prop: 'currMonAllotIn', label: '本月调入', width:100,  } },
                        // {attr: { prop: 'accAllotIn', label: '开累调入', width:100,  } },
                        {attr: { prop: 'currMonTurnover', label: '本月摊销', width:100,  } },
                        // {attr: { prop: 'accDelivery', label: '开累发料', width:100,  } },
                        {attr: { prop: 'currMonAllot', label: '本月调拨', width:100,  } },
                        // {attr: { prop: 'accAllot', label: '开累调拨', width:100,  } },
                        {attr: { prop: 'currMonScrap', label: '本月报废', width:100,  } },
                        // {attr: { prop: 'accScrap', label: '开累报废', width:100,  } },
                        {attr: { prop: 'currMonSell', label: '本月让售', width:100,  } },
                        {attr: { prop: 'currJC', label: '本月结存', width:100,  } },]
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
                            columns:this.columns1,
                        }
                    }
                }
    },
    methods:{
        _funcState(){
            if(this.selectRows.length>0){
               this.funBtnConf['audit'].disabled=false;
               this.funBtnConf['unAudit'].disabled=false;
               if(this.selectRows[0].isAudit){
                   this.funBtnConf['audit'].disabled=true;
                   this.funBtnConf['unAudit'].disabled=false;
               }else{
                   this.funBtnConf['audit'].disabled=false;
                   this.funBtnConf['unAudit'].disabled=true;
               }
         }else{
               this.funBtnConf['audit'].disabled=true;
               this.funBtnConf['unAudit'].disabled=true;
         }
        },
        _currentChange(val){
            // console.log(val);
            if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }
                this._funcState();
        },
        _currentDetail(val){
                this.currentRow=val;
                this.addFormVisible=true;
        },
        _close(){
                this.addFormVisible=false;
        },
        _getReportList(){
            this.mainTableLoading=true;
            let _this=this;
            let params={};
            params={
                procType:0,
                firstKeys:"OrgId",
                firstValues:this.getUserInfo().user.manageOrgId,
                secondKeys:'',
                secondOperates:'',
                secondValues:'',
                procName:'Report_TurnoverMaterialDymatic'
            },
           //执行通用存储过程获取表格数据
            this.commonSqlExcute(params,(val)=>{
                this.tableData=val
                this.mainTableLoading=false;
            });
        },
        _reload(){
            this._getReportList();
        },
    },
    components:{
        'yl-Reports':Reports,
        Detail
    },
    mounted(){
        this._getReportList();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
