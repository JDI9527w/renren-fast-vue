<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.orderId" placeholder="订单id" clearable style="width: 140px"></el-input>
        <el-input v-model="dataForm.trackingNo" placeholder="物流单号" clearable style="width: 140px"></el-input>
        <el-input v-model="dataForm.deliveryAddress" placeholder="收货地址" clearable style="width: 280px"></el-input>
        <el-input v-model="dataForm.consigneeTel" placeholder="收货人电话" clearable style="width: 140px"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('ware:wareordertask:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('ware:wareordertask:delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="id">
      </el-table-column>
      <el-table-column
        prop="orderId"
        header-align="center"
        align="center"
        label="order_id">
      </el-table-column>
      <el-table-column
        prop="orderSn"
        header-align="center"
        align="center"
        label="order_sn">
      </el-table-column>
      <el-table-column
        prop="consignee"
        header-align="center"
        align="center"
        label="收货人">
      </el-table-column>
      <el-table-column
        prop="consigneeTel"
        header-align="center"
        align="center"
        label="收货人电话">
      </el-table-column>
      <el-table-column
        prop="deliveryAddress"
        header-align="center"
        align="center"
        label="配送地址">
      </el-table-column>
      <el-table-column
        prop="orderComment"
        header-align="center"
        align="center"
        label="订单备注">
      </el-table-column>
      <el-table-column
        prop="paymentWay"
        header-align="center"
        align="center"
        label="付款方式">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.payment==1">在线付款</el-tag>
          <el-tag v-if="scope.row.payment==2">货到付款</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="taskStatus"
        header-align="center"
        align="center"
        label="任务状态">
      </el-table-column>
      <el-table-column
        prop="orderBody"
        header-align="center"
        align="center"
        label="订单描述">
      </el-table-column>
      <el-table-column
        prop="trackingNo"
        header-align="center"
        align="center"
        label="物流单号">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="wareId"
        header-align="center"
        align="center"
        label="仓库id">
      </el-table-column>
      <el-table-column
        prop="taskComment"
        header-align="center"
        align="center"
        label="工作单备注">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './wareordertask-add-or-update'

export default {
  data() {
    return {
      dataForm: {
        // 订单id
        orderId:null,
        // 物流单号
        trackingNo:null,
        // 配送地址
        deliveryAddress:null,
        // 收货人电话
        consigneeTel:null
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/ware/wareordertask/list'),
        method: 'get',
        params: this.$http.adornParams({
          'pageNum': this.pageIndex,
          'pageSize': this.pageSize,
          'orderId':this.dataForm.orderId,
          'trackingNo':this.dataForm.trackingNo,
          'deliveryAddress':this.dataForm.deliveryAddress,
          'consigneeTel':this.dataForm.consigneeTel
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.data.records
          this.totalPage = data.data.total
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle(id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/ware/wareordertask/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>
