<template>
  <div
    v-loading="loading"
    class="app-container"
    element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(50, 0, 0, 0.1)"
    style="height: auto"
  >
    <div v-if="isInsider!==`0`">
      <el-card class="operate-container" shadow="never">
        <el-select
          v-model="operateType"
          size="small"
          style="float: right"
          placeholder="排序 "
          @change="fetchData"
        >
          <el-option
            v-for="item in operates"
            :key="item.value"
            :label="item.label"
            :value="item.field"
          />
        </el-select>
        <el-button
          class="btn-add"
          size="medium"
          type="primary"
          style="margin: 0px 0px 0px 50%"
          @click="handleAddProduct()"
        >
          添加商品
        </el-button>
      </el-card>
      <div class="table-container">
        <el-table
          ref="productTable"
          :data="list"
          :default-sort="{prop: 'createdTime', order: 'desc'}"
          style="width: 100%;height: 100%"
          @sort-change="fetchData"
        >
          <template slot="empty">
            <img class="data-pic" src="#" alt="">
          </template>
          <el-table-column label="创建时间" prop="createdTime" sortable align="center">
            <template slot-scope="scope">
              {{ scope.row.createdTime | dateformat('YYYY-MM-DD') }}
            </template>
          </el-table-column>
          <el-table-column label="商品图片" prop="img" align="center">
            <template slot-scope="scope">
              <el-image
                style="width: 120px; height: 120px"
                :src="scope.row.img"
                :preview-src-list="scope.row.list"
              />
            </template>
          </el-table-column>
          <el-table-column label="商品名称" prop="name" sortable align="center">
            <template slot-scope="scope">
              <p>{{ scope.row.name }}</p>
            </template>
          </el-table-column>
          <el-table-column label="商品描述" prop="description" sortable align="center">
            <template slot-scope="props">
              <el-popover
                placement="top"
                width="200"
                trigger="hover"
                :content="props.row.description"
              >
                <el-button slot="reference" type="text" style="color: rgb(90, 94, 102);">{{ props.row.description.substring(0,10) }}...</el-button>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column label="规格/价格" prop="price" align="center">
            <template slot-scope="scope">
              <p style="color: #333">{{ scope.row.unit }}</p>
              <p style="color: #999">{{ scope.row.price }}元</p>
            </template>
          </el-table-column>
          <el-table-column label="分类" align="center">
            <template slot-scope="props">
              <p>{{ props.row.category }}</p>
            </template>
          </el-table-column>
          <el-table-column label="状态" align="center">
            <template slot-scope="props">
              <p v-if="props.row.pass==='已下架'" style="color: #8c939d">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='已上架'" style="color: #409EFF">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='未审核'" style="color: #f06a0a">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='审核未通过'" style="color: #f01214">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='审核未通过'">
                <el-tooltip class="item" effect="light" :content="props.row.passComment|propData('暂无')" placement="top">
                  <el-button type="text" style="color: #f01214">详情</el-button>
                </el-tooltip>
              </p>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="300" align="center">
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="primary"
                @click="handleUpdateProduct(scope.row.id)"
              >编辑
              </el-button>
              <el-button
                v-if="scope.row.pass===`已上架`"
                size="mini"
                type="danger"
                disabled
                @click="handleDelete(scope.row.id)"
              >删除
              </el-button>
              <el-button
                v-if="scope.row.pass!==`已上架`"
                size="mini"
                type="danger"
                @click="handleDelete(scope.row.id)"
              >删除
              </el-button>
              <el-button
                v-if="scope.row.pass===`已上架`"
                size="mini"
                type="primary"
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                下架
              </el-button>
              <el-button
                v-if="scope.row.pass===`审核未通过`"
                size="mini"
                type="primary"
                disabled
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
              <el-button
                v-if="scope.row.pass===`已下架`"
                size="mini"
                type="primary"
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
              <el-button
                v-if="scope.row.pass===`未审核`"
                size="mini"
                type="primary"
                disabled
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div style="text-align: center">
        <el-pagination
          :current-page.sync="currentPage"
          :page-size="size"
          layout="total, prev, pager, next"
          :total="total"
          @size-change="fetchData()"
          @current-change="fetchData()"
        />
      </div>
    </div>
    <div
      v-if="fefresh===1"
      align="center"
    >
      <el-card class="operate-container" shadow="never">
        <el-select
          v-model="operateType"
          size="small"
          style="float: right"
          placeholder="排序 "
          @change="fetchData(val,farmId)"
        >
          <el-option
            v-for="item in operates"
            :key="item.value"
            :label="item.label"
            :value="item.field"
          />
        </el-select>
        <el-button
          class="btn-add"
          size="medium"
          type="primary"
          style="margin: 0px 0px 0px 15%"
          @click="permissionVisble=true"
        >
          切换农场
        </el-button>
        <el-button
          class="btn-add"
          size="medium"
          type="primary"
          @click="handleAddProduct()"
        >
          添加商品
        </el-button>
      </el-card>
      <div class="table-container">
        <el-table
          ref="productTable"
          :data="list"
          :default-sort="{prop: 'createdTime', order: 'desc'}"
          empty-text="暂时还没有商品，请稍后再来"
          style="width: 100%;height: 100%"
          @sort-change="fetchData(undefined,farmId)"
        >
          <!--<el-table-column type="selection" width="60" align="center"></el-table-column>-->
          <template slot="empty">
            <img class="data-pic" src="#" alt="">
          </template>
          <el-table-column label="创建时间" prop="createdTime" sortable align="center">
            <template slot-scope="scope">
              {{ scope.row.createdTime | dateformat('YYYY-MM-DD HH:mm') }}
            </template>
          </el-table-column>
          <el-table-column label="商品图片" prop="img" align="center">
            <template slot-scope="scope">
              <el-image
                style="width: 120px; height: 120px"
                :src="scope.row.img"
                :preview-src-list="scope.row.list"
              />
            </template>
          </el-table-column>
          <el-table-column label="商品名称" prop="name" sortable align="center">
            <template slot-scope="scope">
              <p>{{ scope.row.name }}</p>
              <!--<p>品牌：{{scope.row.brandName}}</p>-->
            </template>
          </el-table-column>
          <el-table-column label="商品描述" sortable prop="description" align="center">
            <template slot-scope="props">
              <el-popover
                placement="top"
                width="200"
                trigger="hover"
                :content="props.row.description"
              >
                <el-button slot="reference" type="text" style="color: rgb(90, 94, 102);">{{ props.row.description.substring(0,10) }}...</el-button>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column label="规格/价格" sortable prop="price" align="center">
            <template slot-scope="scope">
              <p style="color: #333">{{ scope.row.unit }}</p>
              <p style="color: #999">{{ scope.row.price }}元</p>
            </template>
          </el-table-column>
          <el-table-column label="分类" align="center">
            <template slot-scope="props">
              <p>{{ props.row.category }}</p>
            </template>
          </el-table-column>
          <el-table-column label="状态" align="center">
            <template slot-scope="props">
              <p v-if="props.row.pass==='已下架'" style="color: #8c939d">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='已上架'" style="color: #409EFF">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='未审核'" style="color: #f06a0a">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='审核未通过'" style="color: #f01214">
                {{ props.row.pass }}
              </p>
              <p v-if="props.row.pass==='审核未通过'">
                <el-tooltip class="item" effect="light" :content="props.row.passComment" placement="top">
                  <el-button type="text" style="color: #f01214">详情</el-button>
                </el-tooltip>
              </p>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="300" align="center">
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="primary"
                @click="handleUpdateProduct(scope.row.id)"
              >编辑
              </el-button>
              <el-button
                v-if="scope.row.pass===`已上架`"
                size="mini"
                type="danger"
                disabled
                @click="handleDelete(scope.row.id)"
              >删除
              </el-button>
              <el-button
                v-if="scope.row.pass!==`已上架`"
                size="mini"
                type="danger"
                @click="handleDelete(scope.row.id)"
              >删除
              </el-button>
              <el-button
                v-if="scope.row.pass===`已上架`"
                size="mini"
                type="primary"
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                下架
              </el-button>
              <el-button
                v-if="scope.row.pass===`审核未通过`"
                size="mini"
                type="primary"
                disabled
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
              <el-button
                v-if="scope.row.pass===`已下架`"
                size="mini"
                type="primary"
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
              <el-button
                v-if="scope.row.pass===`未审核`"
                size="mini"
                type="primary"
                disabled
                @click="addGoods(scope.row.id,scope.row.pass)"
              >
                上架
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div style="text-align: center">
        <el-pagination
          :current-page.sync="currentPage"
          :page-size="size"
          layout="total, prev, pager, next"
          :total="total"
          @size-change="fetchData(undefined,farmId)"
          @current-change="fetchData(undefined,farmId)"
        />
      </div>
    </div>
    <el-dialog
      title="请选择您要查看的农场"
      :visible.sync="permissionVisble"
      :close-on-click-modal="false"
      custom-class="customWidth"
      width="65%"
      :close-on-press-escape="false"
      :before-close="permissionClose"
      style="text-align: center;"
    >
      <div style="overflow-y: auto; width:100%; height:500px">
        <div class="RadioStyle">
          <div v-for="(val,index) in care" class="Block PaddingL">
            <input :id="val.id" type="radio" name="id" :value="val.id">
            <label :id="val.id" :for="val.id" @click="clickOnSubmit(val.id)">{{ val.name }}</label>
          </div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import {
  fachData,
  itemDelete,
  addGoods,
  permissionFarm,
  findInfo
} from '@/api/farm'
export default {
  data() {
    return {
      list: [],
      listQuery: {},
      loading: false,
      currentPage: 1,
      size: 4,
      total: 0,
      operates: [
        {
          label: '全部商品',
          value: 'publishCom',
          field: 'default'
        },
        {
          label: '已上架的商品',
          value: 'publishOn',
          field: 'up'
        },
        {
          label: '已下架的商品',
          value: 'publishOff',
          field: 'down'
        },
        {
          label: '未审核的商品',
          value: 'recommendOff',
          field: 'init'
        },
        {
          label: '审核未通过的商品',
          value: 'newOn',
          field: 'fail'
        }
      ],
      operateType: null,
      isInsider: ``,
      permissionVisble: false,
      care: [],
      permissionId: ``,
      fefresh: ``,
      farmId: ``
    }
  },
  created() {
    this.permission()
  },
  methods: {
    permission() {
      const user = window.localStorage.getItem('user')
      const farmId = window.localStorage.getItem('farm')
      this.farmId = window.localStorage.getItem('farm')
      const companyId = user.substring(0, user.indexOf(','))
      this.isInsider = user.substring(user.indexOf(',') + 1, user.length)
      if (this.isInsider !== `0`) {
        this.operateType = 'default'
        this.fetchData('default')
      }
      if (this.isInsider === `0`) {
        permissionFarm().then(res => {
          this.care = res.data.queryResult.data.list
          if (farmId !== null) {
            this.care.forEach(item => {
              if (item.id === farmId) {
                this.$message.closeAll()
                this.$message({
                  message: '当前位于:' + item.name,
                  type: 'success',
                  duration: 1000 * 5
                })
              }
            })
            this.fefresh = 1
            this.operateType = 'default'
            this.fetchData('default', farmId)
          }
          if (farmId === null) {
            this.permissionVisble = true
          }
        })
      }
    },
    permissionOnSubmit() {

    },
    clickOnSubmit(id) {
      this.permissionId = id
      window.localStorage.removeItem('farm')
      window.localStorage.setItem('farm', id)
      this.care.forEach(item => {
        if (item.id === this.permissionId) {
          this.$message.closeAll()
          this.$message({
            message: '当前位于:' + item.name,
            type: 'success',
            duration: 1000*5
          })
          window.localStorage.removeItem('farmName')
          window.localStorage.setItem('farmName',item.name)
        }
      })
      this.fefresh = 1
      this.loading = true
      this.permissionVisble = false
      this.operateType = 'default'
      this.fetchData('default', window.localStorage.getItem('farm'))
    },
    permissionClose() {
      this.permissionVisble = false
    },
    fetchData(val, itemId) {
      let order = 'desc'
      let name = 'created_time'
      if (val !== undefined && val.prop === `createdTime`) {
        if (val.order === undefined) {
          name = 'created_time'
          order = 'desc'
        }
        if (val.order === `name`) {
          name = 'name'
          order = 'desc'
        }
        if (val.order === `ascending`) {
          name = 'created_time'
          order = 'asc'
        }
      }
      if (val !== undefined && val.prop === `name`) {
        if (val.order === undefined) {
          name = 'name'
          order = 'desc'
        }
        if (val.order === `descending`) {
          name = 'name'
          order = 'desc'
        }
        if (val.order === `ascending`) {
          name = 'name'
          order = 'asc'
        }
      }
      if (val !== undefined && val.prop === `description`) {
        if (val.order === undefined) {
          name = 'description'
          order = 'desc'
        }
        if (val.order === `descending`) {
          name = 'description'
          order = 'desc'
        }
        if (val.order === `ascending`) {
          name = 'description'
          order = 'asc'
        }
      }
      if (val !== undefined && val.prop === `price`) {
        if (val.order === undefined) {
          name = 'price'
          order = 'desc'
        }
        if (val.order === `descending`) {
          name = 'price'
          order = 'desc'
        }
        if (val.order === `ascending`) {
          name = 'price'
          order = 'asc'
        }
      }
      val = this.operateType
      this.loading = true
      fachData(this.currentPage, this.size, name, order, val, itemId).then(res => {
        this.loading = false
        const dataList = res.data.queryResult.data
        this.list = dataList.list
        this.list.forEach(item => {
          const arr = []
          arr.push(item.img)
            item.list = arr
        })
        for (let x = 0; x < this.list.length; x++) {
          if (this.list[x].pass === 0) {
            this.list[x].pass = '未审核'
          }
          if (this.list[x].pass === -1) {
            this.list[x].pass = '审核未通过'
          }
          if (this.list[x].pass === 1) {
            if (this.list[x].state === 1) {
              this.list[x].pass = '已上架'
            }
            if (this.list[x].state === -1) {
              this.list[x].pass = '已下架'
            }
          }
        }
        this.total = dataList.total
      })
      this.loading = false
    },
    handleAddProduct() {
      this.$router.push({ path: 'farmAdd' })
    },
    handleDelete(id) {
      this.$confirm('此操作将永久删除此商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.listQuery = true
        this.loading = true
        itemDelete(id).then(res => {
          this.loading = false
          if (res.data.success === true) {
            if (window.localStorage.getItem('farm') === null) {
              this.fetchData()
            }
            if (window.localStorage.getItem('farm') !== null) {
              this.fetchData('default', window.localStorage.getItem('farm'))
            }
            this.$message({
              type: 'success',
              message: '删除成功'
            })
            this.listQuery = false
          }
        }).catch(err => {
          this.loading = false
          this.$message({
            type: 'info',
            message: res.data.message
          })
          this.listQuery = false
        })
      }).catch(() => {})
    },
    addGoods(id, pass) {
      let url = ''
      if (pass === '已上架') {
        url = `down`
        const user = {}
        const arr = []
        arr.push(id)
        this.$confirm('是否将此商品下架, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.loading = true
          addGoods(url, arr).then(res => {
            this.loading = false
            if (this.isInsider === '0') {
              this.fetchData(`default`, window.localStorage.getItem('farm'))
            } else {
              this.fetchData()
            }
            if (res.data.success === true) {
              this.$message({
                type: 'success',
                message: '下架成功'
              })
            } else {
              this.$message({
                type: 'info',
                message: res.data.message
              })
            }
          })
        }).catch(() => {})
      }
      if (pass === '已下架') {
        url = `up`
        const user = {}
        const arr = []
        arr.push(id)
        this.$confirm('是否将此商品上架, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.loading = true
          addGoods(url, arr).then(res => {
            this.loading = false
            if (this.isInsider === '0') {
              this.fetchData(`default`, window.localStorage.getItem('farm'))
            } else {
              this.fetchData()
            }
            if (res.data.success === true) {
              this.$message({
                type: 'success',
                message: '上架成功'
              })
            } else {
              this.$message({
                type: 'info',
                message: res.data.message
              })
            }
          })
        }).catch(() => {})
      }
    },
    handleUpdateProduct(id) {
      this.$router.push({ name: '农场编辑商品', query: { id: id }})
    },
    operateChange(val) {
    }
  }
}
</script>
<style>
  .el-input--small .el-input__inner {
    height: 36px;
    line-height: 36px;
  }
</style>
