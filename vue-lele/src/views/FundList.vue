<template>
<div class="fillcontain">
    <div>
        <el-form
                :inline="true"
                ref="search_data" 
                :model="search_data"
                >
                <el-form-item label="投标时间筛选:">
                    <el-date-picker
                        v-model="search_data.startTime"
                        type="datetime"
                        placeholder="选择开始时间">
                    </el-date-picker> --
                    <el-date-picker
                        v-model="search_data.endTime"
                        type="datetime"
                        placeholder="选择结束时间">
                    </el-date-picker>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" size ="small" icon="search" @click='handleSearch()'>筛选</el-button>
                </el-form-item>

                 <el-form-item class="btnRight">
                    <el-button type="primary" size ="small" icon="view" v-if="user.identity=='manager'" @click='AddMoney()'>添加</el-button>
                </el-form-item>
            </el-form>

    </div>
    <div class="table_contaniner">
 
        <el-table
                v-if="tableData.length>0"
                :data="tableData"
                max-height="450"
                border
                style="width: 100%">
                 <el-table-column
                type="index"
                label="序号"
                align='center'
                width="70">   
                </el-table-column>
                <el-table-column
                prop="date"
                label="创建时间"
                align='center'
                width="250">  
                <template slot-scope="scope">
                        <el-icon name="time"></el-icon>
                        <span style="margin-left: 10px">{{ scope.row.date }}</span>
                    </template> 
                </el-table-column>
                <el-table-column
                    prop="type"
                    label="收支类型"
                    align='center'
                    width="150">
                </el-table-column>
                <el-table-column
                    prop="describe"
                    label="收支描述"
                    align='center'
                    width="170">
                </el-table-column>
                <el-table-column
                    prop="income"
                    label="收入"
                    align='center'
                    width="170"> 
                    <template slot-scope="scope">  
                        <span style="color:#00d053">+ {{ scope.row.income }}</span>
                    </template>
                </el-table-column>
                <el-table-column
                    prop="expend"
                    label="支出"
                    align='center'
                    width="170">
                    <template slot-scope="scope">  
                        <span style="color:#f56767">- {{ scope.row.expend }}</span>
                    </template>
                </el-table-column>
                <el-table-column
                    prop="cash"
                    label="账户现金"
                    align='center'
                    
                    width="170">
                    <template slot-scope="scope">  
                        <span style="color:#4db3ff">{{ scope.row.cash }}</span>
                    </template>
                </el-table-column>
                 <el-table-column
                    prop="remark"
                    label="备注"
                    align='center'
                    width="220">
                </el-table-column>
                <el-table-column 
                  prop="operation"
                  label="操作" 
                  
                  align='center'
                  width="320"
                  v-if="user.identity=='manager'">
                 <template slot-scope="scope">
                    <el-button
                    type="warning"
                    align='center'
                     size="small"
                    @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button
                    align='center'
                    size="small"
                    type="danger"
                    
                     @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
                 </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-row :span="24">
            <div class="pagination">
                <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page.sync="paginations.page_index"
                    :page-sizes="paginations.page_sizes"
                    :page-size="paginations.page_size"
                    :layout="paginations.layout"
                    :total="paginations.total">
                </el-pagination>
            </div>
            
        </el-row>
    </div>

    <DialogFound :dialog="dialog" :formData="formData" @update="getProfile"></DialogFound> 
    </div>
</template>
<script>
import DialogFound from "../components/DialogFound";
export default {
    name:'fundlist',
    data(){
        return {
            search_data:{
                startTime:"",
                endTime:""
            },
            filerTableData:{},
            paginations:{
                page_index:1,//当前位于哪页
                total:0,//当前数据总数
                page_size:5,//一页显示多少条
                page_sizes:[5,10,15,20],//每页显示多少条
                layout:"total,sizes,prev,pager,next,jumper"//翻页属性
            },
            tableData:{},
            allTableData:{},
            formData:{
            type:'',
            describe:'',
            income:'',
            expend:'',
            cash:'',
            remark:'',
            id:''
         } ,
            dialog:{
                show:false,
                title:"",
                option:"edit"
            }
        };
    },
    computed:{
        user(){
            return  this.$store.getters.user;
        }
    },
    components:{
     DialogFound
    },
    created(){
        this.getProfile();
    },
    methods:{
    getProfile(){
        //获取表格数据
        this.$axios.get('/api/profiles')
        .then(res=>{
             this.allTableData=res.data;
             this.filerTableData=res.data;
             //设置分页数据
             this.setPaginations();
            //console.log(res)
        })
        .catch(err=>console.log(err));
    },
    handleEdit(index,row){
        //编辑
        this.dialog={
            show:true,
            title:"修改基金信息",
            option:'edit'
        }
        this.formData={

            type:row.type,
            describe:row.describe,
            income:row.income,
            expend:row.expend,
            cash:row.cash,
            remark:row.remark,
            id:row._id

        }

    },
    handleDelete(index,row){
        this.$axios.delete(`api/profiles/delete/${row._id}`).then((result)=>{
                this.$message('删除成功');
                this.getProfile();

        })

    },
    AddMoney(){
         this.dialog={
            show:true,
            title:"添加基金信息",
            option:'add'
        },
        this.formData={

            type:"",
            describe:"",
            income:"",
            expend:"",
            cash:"",
            remark:"",
            id:""

        }
    },
    handleSizeChange(page_size){
        //切换size
        this.paginations.page_index=1;
        this.paginations.page_size=page_size;
         this.tableData=this.allTableData.filter((item,index)=>{
            return index <page_size;
        })
    },
    handleCurrentChange(page){
        //获取当前页
        let index=this.paginations.page_size*(page-1);
        //数据总数
        let nums=this.paginations.page_size*page;
        let tables=[];
        for (let i =index;i<nums;i++){

            if(this.allTableData[i]){
                tables.push(this.allTableData[i]);
            }
            this.tableData=tables;
        }
    },
    setPaginations(){
        //分页属性设置
        this.paginations.total=this.allTableData.length;
        this.paginations.page_index=1;
        this.paginations.page_size=5;
        //设置默认分页数据
        this.tableData=this.allTableData.filter((item,index)=>{
            return index <this.paginations.page_size;
        })

    },
    handleSearch(){
        if(!this.search_data.startTime||!this.search_data.endTime){
            this.$message({
                type:"warning",
                message:"请选择时间区间"
            })
            this.getProfile();
            return;
        };
        const sTime=this.search_data.startTime.getTime();
        const eTime=this.search_data.endTime.getTime();

        this.allTableData=this.filerTableData.filter(item=>{

            let date=new Date(item.date);
            let  time=date.getTime();
            return time >=sTime && time<=eTime;
        });
        //分页数据调用
        this.setPaginations();

    }
    }
    

}

</script>

<style scoped>
.fillcontain {
  width: 100%;
  height: 100%;
  padding: 16px;
  box-sizing: border-box;
}
/* .btnRight {
  /* float: right; */
/* }  */
.pagination {
  text-align: center;
  margin-top: 10px;
}
</style>

