<!--系统转账管理  系统钱包列表-->
<template>
  <section>
    <!--工具条-->
    <el-form :model="form" ref="form"  :inline="true"  :label-position="labelPosition" label-width="80px" >
      <el-col :span="24" class="toolbar" style="padding-bottom: 0px;margin-bottom: 0;min-width:1080px">
        <el-form-item label="订单号" >
          <el-input v-model="form.trans_id" clearable  placeholder="订单号"></el-input>
        </el-form-item>
        <el-form-item label="币种">
          <el-select v-model="form.wallet_type" placeholder="请选择">
            <el-option
              v-for="item in walletTypes"
              :key="item.walletType"
              :label="item.walletShortEn"
              :value="item.walletType">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="创建时间">
          <el-date-picker
            v-model="dataRange"
            type="daterange"
            start-placeholder="开始时间"
            end-placeholder="结束时间"
            value-format="timestamp"
            @change="selectDate"
          >
          </el-date-picker>
        </el-form-item>
      </el-col>
      <el-col :span="24" class="toolbar" style="padding-bottom: 0px;margin-top: 0;">
        <el-form-item label="地址">
          <el-input v-model="form.wallet_address" clearable placeholder="地址"></el-input>
        </el-form-item>
        <el-form-item label="账户类型">
          <el-select v-model="form.base_type" placeholder="请选择">
            <el-option
              v-for="item in baseTypes"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
       
        <el-form-item>
          <el-button type="primary" @click="handleQuery('form')">查询</el-button>
          <el-button type="primary" @click="handleExportClick()">导出</el-button>
        </el-form-item>
      </el-col>
    </el-form>
    <!--新增-->
    <el-col :span="22" class="" style="padding-bottom: 0px;margin-top: 20px;">
      <template>
        <el-tabs v-model="biz_type" type="card">
          <el-tab-pane label="系统钱包管理"  name="1"></el-tab-pane>
        </el-tabs>
      </template>
    </el-col>
    <el-col :span="2" class="" style="padding-bottom: 0px;margin-top: 20px;text-align:right">
     <!--  <el-button type="primary" @click="handleAdd()">新增钱包</el-button> -->
    </el-col>

    <!--列表 -->
    <el-col :span="24" class="" style="padding-bottom: 0px;margin-top: 0;">
      <el-table :data="listData"  :fit="tableFit" highlight-current-row border   style="width: 100%;">
        
        <el-table-column prop="walletType" label="币种" width="60"  align="center" :formatter="walletTypeFormatter">
        </el-table-column>
        <el-table-column prop="baseType" label="账户类型" width="60" align="center" :formatter="baseTypeFormatter">
        </el-table-column>
        <el-table-column prop="walletAddress" label="地址" width="200" align="center" >
        </el-table-column>
        <el-table-column prop="userMoney" label="可用币数" width="60" align="right" :formatter="namberFormatter">
        </el-table-column>
        <el-table-column prop="addtime" label="创建时间" width="120" align="center" :formatter="tableTimeFormatter">
        </el-table-column>
         <el-table-column prop="transLocked" label="启用状态" width="60"  align="center" :formatter="transLockedFormatter">
        </el-table-column>
         <el-table-column label="操作" width="140" align="center">
          <template  slot-scope="scope">
            <el-button size="small" @click="handleDetailClick(scope.$index, scope.row)" :disabled="roleId==3">修改</el-button>
            <el-button size="small" @click="handlePutInClick(scope.$index, scope.row)" :disabled="roleId==3">从冷钱包转入</el-button>
          </template>
        </el-table-column>

      </el-table>
    </el-col>
    <!--分页 :page-size="10"  :page-sizes="[10, 20, 30, 40]"-->
    <el-col :span="24" class="toolbar" style="background-color:#fff">
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page="form.page_number"
        layout="prev, pager, next, total, jumper"
        :total="listTotal">
      </el-pagination>  
    </el-col>
    <el-dialog title="二维码" :visible.sync="dialogEwmVisible">
      <p>{{imgMsg}}</p>
      <div id="qrcode"></div>

      <!-- <img :src="ewmImg" class="ewm-img" /> -->
    </el-dialog>
  </section>
</template>
<script>
  import QRCode from 'qrcodejs2'
  import { requestApi ,exportApi} from '../../api/axios.js';
  import util from '../../util.js';
  export default{
    components:{
      QRCode:QRCode
    },
    data:function(){
      return{
        roleId:null,
        tableFit:false,
        labelPosition:'left',
        dialogEwmVisible:false, 
        imgMsg:'',
        dataRange:'',
        listTotal:0,//  列表数据总量
        exportNumber:10,
        biz_type:'1',
        //查询集合
        form:{
          api_method:'WalletAddressListByPlat',
          trans_id:null,//交易id 订单号 
          wallet_address:null,//地址
          wallet_type:null, //币种 钱包类型
          addtime_begin:null,//开始时间
          addtime_end:null,//结束时间
          base_type:null,//账户类型
          page_number:1,//表格当前页
          page_size:10
        },
        //币种 钱包类型 选择器数据
        walletTypes:[
          /*{
            walletType:null,
            walletShortEn:'全部'
          },
          {
            walletType:1,
            walletShortEn:'BTC'
          },
          {
            walletType:2,
            walletShortEn:'ETH'
          },
          {
            walletType:3,
            walletShortEn:'USDT'
          } */
        ],
        //账户类型 
        baseTypes:[
          {
            value:null,
            label:'全部'
          },
          {
            value:1,
            label:'系统'
          },
         /* {
            value:2,
            label:'用户'
          },*/
          {
            value:3,
            label:'冷钱包'
          },
          {
            value:4,
            label:'找零地址'
          },
          {
            value:5,
            label:'总账户'
          }
        ],
         //启用状态  选择器数据
        transLockeds:[
          {
            value:null,
            label:'全部'
          },
          {
            value:0,
            label:'已启用'//使用
          },
          {
            value:1,
            label:'未启用'//锁定
          }
        ],
        //列表
        listData:[]
      }
    },
    create(){
    },
    mounted(){
      this.roleId = sessionStorage.getItem('BITKER_ROLE_ID');
      this.query();
      this.setwalletTypeList();

    },
    methods: {
      //二维码
      qrcodes(msg) {
        let qrcoder = new QRCode('qrcode', {
          width: 200,
          height: 200, // 高度
          text: msg // 二维码内容
          // render: 'canvas' // 设置渲染方式（有两种方式 table和canvas，默认是canvas）
          // background: '#f0f'
          // foreground: '#ff0'
        })
        console.log('qrcoder',qrcoder)
      },
      //选择时间范围
      selectDate(dateRange){
        let dr = {
          addtime_begin:dateRange?dateRange[0]/1000:null,
          addtime_end:dateRange?dateRange[1]/1000:null
        };
        Object.assign(this.form,dr);
      },
      walletTypeFormatter(row, column, cellValue, index){
        for(let item of this.walletTypes){
          if(item.walletType == cellValue){
            return item.walletShortEn;
          }
        }
      },
      namberFormatter(row, column, cellValue, index){
        return cellValue.toFixed(8);
      },
      tableTimeFormatter(row, column, cellValue, index){
        if(cellValue == 0){return '--';}
        let cellTime =new Date(parseInt(cellValue) * 1000);
        return util.formatDate.format(cellTime);
        //return cellValue
      },
      baseTypeFormatter(row, column, cellValue, index){
        //console.log('cellValue',cellValue);
        for(let item of this.baseTypes){
          if(item.value == cellValue){
            return item.label;
          }
        }
      },
      transLockedFormatter(row, column, cellValue, index){
        console.log('cellValue',cellValue);
        for(let item of this.transLockeds){
          if(item.value == cellValue+''){
            return item.label;
          }
        }
      },
      //查询
      handleQuery(form){
        this.form.page_number = 1;
        this.$refs[form].validate((valid) => {
          if (valid) {
           this.query();   
          } else {
            console.log('error submit!!');
            return false;
          }
        });
        //console.log(JSON.stringify(this.form));
      },
      //分页查询
      handleCurrentChange(val) {
        this.form.page_number = val;
        this.query();
        //this.getUsers();
      },
      handleAdd(){
        this.$router.push({ path: '/WalletAddressModify'});
      },
      //详情查看 修改
      handleDetailClick(index,row){
        //console.log('index',index);
        //console.log('row',row.id);
        this.$router.push({ path: '/WalletAddressModify', query: { id: row.id } });
      },
      //从冷钱包转入
      handlePutInClick(index,row){
        let params = {
          api_method:'WalletAddressDecode',
          uid:row.uid,
          wallet_type:row.walletType,
          wallet_address:row.walletAddress,
        }
        this.WalletAddressDecode(params);
      },
      WalletAddressDecode(params = {api_method:'WalletAddressDecode',uid:-1,wallet_type:1,wallet_address:''}){
        requestApi(params).then((res) => {
          let {data,msg,status} = res;
          if(status !== '200'){
            this.$message({
              message: msg,
              type: 'error'
            });
            if(status == '211'){
              this.$router.push({ path: '/login'}); 
            }
          }else{
            //alert(firstEwm)
    
            this.dialogEwmVisible = true;

            document.getElementById('qrcode').innerHTML = "";
             //this.$nextTick(() => {
            this.qrcodes(msg); 
            this.imgMsg = msg;
             //})
            //this.qrcodes(msg);
            //this.ewmImg = 'data:image/png;base64,' + msg
          }
          //console.log(res);
        }).catch(() => {
        });

      },
      query(){
        //this.form.biz_type = parseInt(this.form.biz_type);
        requestApi(this.form).then((res) => {
          let {data,total,msg,status} = res;
          if(status !== '200'){
            this.$message({
              message: msg,
              type: 'error'
            });
            if(status == '211'){
              this.$router.push({ path: '/login'}); 
            }
          }else{
            this.listData = data;
            this.listTotal = total;
          }
          console.log(res);
        }).catch(() => {
        });
      },
      handleExportClick(){//导出
         this.$prompt('请输入导出数据的条数', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /^[0-9]*$/,
          inputErrorMessage: '数字格式不正确'
        }).then(({ value }) => {
          this.exportNumber = value;
          this.export();
          /*this.$message({
            type: 'success',
            message: '导出的条数是: ' + value
          });*/
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消导出'
          });       
        });
      },
      export(){
        let params = this.form;
        params.api_method = 'WalletAddressListByPlatExp';
        params.page_number = 1;
        params.page_size = this.exportNumber;
        exportApi(params).then((res) => {
         if(res){
            this.form.api_method = 'WalletAddressListByPlat'; 
            this.$message({
              message: '导出成功',
              type: 'success'
            });
          }else{
            this.$message({
              message: msg,
              type: 'error'
            });
            if(status == '211'){
              this.$router.push({ path: '/login'}); 
            } 
          }
        }).catch(() => {
        });
      },
       //设置系统钱包列表 热钱包
      setwalletTypeList(){ 
        let walletTypeList = sessionStorage.getItem('WalletTypeList');
        this.walletTypes = JSON.parse(walletTypeList);
        this.walletTypes.unshift({walletType:null,walletShortEn:'全部'});
      }  
    }
  }
</script>

<style>
.ewm-img{
  width: 200px;
  height: 200px;
}
.el-table table{
  width:100% !important;
  table-layout: initial; 
}
.el-table th{
  background:#e5e9f2;
  color: #000;   
}
.el-table td{
  padding:4px 0;  
}
.el-table tr:odd{

}
.el-table--border th{
  border-right: 1px solid #cfd1d7;
}
.el-table__body, .el-table__footer, .el-table__header {
     table-layout: initial; 
}
.el-dialog__body{
  text-align: center;
}

.el-dialog {
  width: 30%;
}
#qrcode img{
  margin: 0 auto;
}
</style>