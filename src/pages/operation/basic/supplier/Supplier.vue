<template>
<yl-panelpage :titleName="'供应商管理'" >
<div slot="content">
    <yl-layout>
        <div slot="fristbox" >
          <yl-toolbar>
                <el-form  :inline="true"  >
                        <el-form-item class="form-content-vertical">
                        <el-input  placeholder="供应商名称" size="small" style="width:200px" v-model="searchModel.supplierName"></el-input>  
                        </el-form-item>
                        <el-form-item class="form-content-vertical">
                            <el-button type="primary" icon="search" size="small"  @click="_reload">查 询</el-button>
                        </el-form-item>
                </el-form>
          </yl-toolbar>
        <yl-toolbar>
            <el-button-group>
                    <el-button type="primary" v-permissionSetting="'Pages.Basic.Data.Suppliers.Create'"   icon="plus" size="small"  @click="_add" >添加</el-button>
                    <el-button type="primary" v-permissionSetting="'Pages.Basic.Data.Suppliers.Edit'"   icon="edit"  size="small" :disabled="selectRows.length===0" @click="_edit">编辑</el-button>
                    <el-button type="danger"  v-permissionSetting="'Pages.Basic.Data.Suppliers.Delete'"   icon="delete" size="small"  :disabled="selectRows.length===0" @click="_delete" >删除</el-button>
                    <el-button type="primary" v-permissionSetting="'Pages.Basic.Data.Suppliers.BlackList'"  class="icon-cog2"  size="small" :disabled="selectRows.length===0" @click="_blackList">黑名单</el-button>
            </el-button-group>
        </yl-toolbar>
        </div>
         <div slot="secondbox" class="flexbox">
                <yl-table ref="table"
                    @reload="_reload"
                    @current-change="_currentChange"
                    :configs="tableConfig"
                    :input="mainInput.inputModel"
                    :tableloading="mainTableLoading" 
                    >
                       <template slot="isBlackList" scope="scope">
                                <el-tag type="danger" v-if="scope.row.isBlackList">是</el-tag>
                                <el-tag type="primary" v-else>否</el-tag>
                        </template>
                        <template slot="supplierState" scope="scope">
                           <el-tag type="success" v-if="scope.row.supplierState=='合格'">合格</el-tag>
                           <el-tag type="danger" v-else>不合格</el-tag>
                    </template>

                </yl-table>
         </div>
    </yl-layout>
    <!--编辑界面-->
         <el-dialog ref="mainDialog" title="编辑" v-model="addFormVisible" size="small"  top="10%" 
                    :modal-append-to-body="false" :close-on-click-modal="false"  >
                    <Edit :selectRow="currentRows" @close="_close"  v-if="addFormVisible"></Edit>
         </el-dialog>
 </div>
</yl-panelpage>
</template>
<script type="text/babel">
import Edit from './SupplierEdit';
import  {inputModel} from 'api/inputmodel';
import {
            requestGetSupplierPageList,
            requestDeleteSupplier,
            requestUpdateSupplierBlackListAsync
        }   from 'api/supplier';

export default {
    data(){
        return {
             tableData: {},
                mainTableLoading:false,
                selectRows:[],
                currentRows:{},
                searchModel:{
                    supplierName:'',  
                    },
                addFormVisible:false,
                mainInput:new inputModel(),
               
            }
    },
     computed: {
            tableConfig: {
                    get () {
                        return {
                            table: {
                                attr: {
                                    data: this.tableData,
                                    highlightCurrent:true,
                                }
                            },
                            columns: [
                                {attr: { type: 'index',label: '序号', width:80,align: 'center' }},
                                {attr: { prop: 'isBlackList', label: '黑名单', width:80, scopedSlot: 'isBlackList',  } },
                               {attr: { prop: 'supplierState', label: '状态', width:90, scopedSlot: 'supplierState', } },
                                {attr: { prop: 'supplierCode', label: '编码', width:80,  } },   
                                {attr: { prop: 'supplierName', label: '名称', width:160,  } },   
                                {attr: { prop: 'supplierAbbName', label: '简称', width:120,  } },   
                                {attr: { prop: 'supplierTypeName', label: '供应商类别', width:160,  } },  
                                //{attr: { prop: 'orgId', label: '添加机构id', width:120,  } },   
                                {attr: { prop: 'orgName', label: '添加机构名称', width:160,  } },   
                                {attr: { prop: 'supplierFRDB', label: '法人代表', width:120,  } },   
                                {attr: { prop: 'supplierWTDB', label: '委托代表', width:120,  } },   
                                {attr: { prop: 'supplierContact', label: '联系人', width:140,  } },   
                                {attr: { prop: 'supplierPhone', label: '电话', width:140,  } },   
                                {attr: { prop: 'supplierFax', label: '传真', width:140,  } },   
                                {attr: { prop: 'supplierAddress', label: '地址', width:180,  } },   
                                {attr: { prop: 'supplierPostCode', label: '邮编', width:120,  } },   
                                {attr: { prop: 'supplierWebSit', label: '网址', width:140,  } },   
                                {attr: { prop: 'supplierRegisterCapital', label: '注册资金', width:120,  } },   
                                {attr: { prop: 'supplierFixCaptial', label: '固定资本', width:120,  } },   
                                {attr: { prop: 'supplierDepositBank', label: '开户银行', width:120,  } },   
                                {attr: { prop: 'supplierBankAccount', label: '银行账号', width:200,  } },   
                                {attr: { prop: 'supplierOrgCode', label: '统一社会信用代码', width:150,  } }, 
                                {attr: { prop: 'remark', label: '备注', width:200,  } }, 
                            ]
                        }
                    }
                }
        },
    methods:{
        _getSupplierPageList(){
                var _this=this;
                this.selectRows=[];
                this.mainTableLoading=true; //开启加载
                var inputArr=[ 
                               {key:"SupplierName",op:"LIKE",value:this.searchModel.supplierName},
                             ];
                this.mainInput.addqueryConditionItem(inputArr);
                requestGetSupplierPageList(this.mainInput.inputModel).then(data =>{
                            if(data.success){
                                this.tableData=data.result;
                                }
                            else {
                                 this.$message.error('失败！'+data.error.message);
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                       _this.mainTableLoading=false;
                });
        },
         _blackList(){
             if(this.selectRows[0].isBlackList=="0"){
                  this.selectRows[0].isBlackList=1;
             }else{
                   this.selectRows[0].isBlackList=0; 
             }
              let params={};
              params.supplier=this.selectRows[0];  
              requestUpdateSupplierBlackListAsync(params).then(data =>{
                     if(data.success){
                                this.$notify({
                                        title: '成功',
                                        message: '操作成功！',
                                        type: 'success',
                                        });
                                this._reload();
                                }
                            else {
                                this.$message.error('失败！'+data.error.message);
                                this._reload();
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                       _this.mainTableLoading=false;
                });
        },
        _currentChange(val){
                //单选时的方法
                if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }      
            },
        _selectionChange(val){
            //多选时的方法
            if(val!=null){
                this.selectRows=[];
                this.selectRows=this.selectRows.concat(val);
            }
        },
        toggleRowSelection (row) {
            this.$refs.table.toggleRowSelection(row)
        },
        _reload(){
            this._getSupplierPageList();
        },
        _add(){
            this.addFormVisible=true;
            this.currentRows={};
        },
        _edit(){
                this.addFormVisible=true;
                this.currentRows=this.selectRows[0];
        },
        _delete(){
            this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning'
            }).then(() => {  
                var obj={}; 
                obj.id=this.selectRows[0].id;
                requestDeleteSupplier(obj).then(data =>{
                        if(data.success){
                                this._reload();
                                this.$notify({
                                    title: '成功',
                                    message: '删除数据成功！',
                                    type: 'success'
                                    });
                            }
                            else {
                                this.$notify.error({
                                    title: '错误',
                                      message: '删除数据失败！'+data.error.message,
                                    });
                            }
                });
            });
        },
        _close(){
                this.addFormVisible=false;
                this._reload();
        },
    },
    components:{
         Edit,
        },
    mounted(){
        this._reload();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>

