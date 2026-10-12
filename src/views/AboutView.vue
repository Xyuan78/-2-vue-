<template>
  <div class="container">

    <!-- 头部：修复了布局，将医生信息放入 header 内部 -->
    <div class="header">
      <div class="logo">
        <b>Neusoft</b>
        <span>体检报告管理系统</span>
      </div>
      <div class="doctor-info">
        当前医生：{{ doctorName }}
      </div>
    </div>

    <!--主体-->
    <div class="main">

      <!-- 左侧查询 -->
      <div class="sidebar-left">
        <h2>体检客户查询</h2>

        <!-- 手机号码 -->
        <div class="form-item">
          <label class="static-label">手机号码</label>
          <input type="text" v-model="searchForm.phone" placeholder="请输入手机号码">
        </div>

        <!-- 姓名 -->
        <div class="form-item">
          <label class="static-label">姓名</label>
          <input type="text" v-model="searchForm.name" placeholder="请输入姓名">
        </div>

        <!-- 性别 -->
        <div class="form-item">
          <label class="static-label">性别</label>
          <div class="radio-group">
            <label><input type="radio" v-model="searchForm.gender" value="male"> 男</label>
            <label><input type="radio" v-model="searchForm.gender" value="female"> 女</label>
          </div>
        </div>

        <!-- 套餐类型 -->
        <div class="form-item">
          <label class="static-label">套餐类型</label>
          <select v-model="searchForm.packageType">
            <option value="">全部类型</option>
            <option>男士-肝病检查</option>
            <option>男士-肾病检查</option>
            <option>女士-肝病检查</option>
            <option>女士-肾病检查</option>
            <option>基础套餐</option>
          </select>
        </div>

        <!-- 体检日期 -->
        <div class="form-item">
          <label class="static-label">体检日期</label>
          <input v-model="searchForm.date" type="date">
        </div>

        <!-- 是否归档 -->
        <div class="form-item">
          <label class="static-label">是否归档</label>
          <div class="radio-group">
            <label><input type="radio" v-model="searchForm.archive" value="false"> 未归档</label>
            <label><input type="radio" v-model="searchForm.archive" value="true"> 已归档</label>
          </div>
        </div>

        <!-- 按钮 -->
        <div class="buttons">
          <button class="search-btn" @click="handleSearch">查 询</button>
          <button class="reset-btn" @click="handleReset">重 置</button>
        </div>

      </div>
      <!-- 右侧表格 -->
      <div class="content">
        <table  class="table table-striped">
          <thead>
            <tr>
              <th>预约编号</th>
              <th>手机号码</th>
              <th>真实姓名</th>
              <th>性别</th>
              <th>体检套餐类型</th>
              <th>体检医院</th>
              <th>体检日期</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <!-- 使用 v-for 循环渲染数据 -->
            <tr v-for="(item, index) in filteredList" :key="item.id">
              <td>{{ item.id }}</td>
              <td>{{ item.phone }}</td>
              <td>{{ item.name }}</td>
              <td>{{ item.gender === 'male' ? '男' : '女' }}</td>
              <td>{{ item.packageType }}</td>
              <td>{{ item.hospital }}</td>
              <td>{{ item.date }}</td>
              <td><button class="edit-link btn btn-primary" @click="handleEdit(item)" type="button">编辑体检报告</button></td>
            </tr>

            <!-- 无数据提示 -->
            <tr v-if="filteredList.length === 0">
              <td colspan="8" class="empty-state">暂无匹配数据</td>
            </tr>
          </tbody>
        </table>

        <!-- 分页器 -->
        <div class="pagination">
          <button>&lt; 上一页</button>
          <span>第 1 页，共 12 页</span>
          <button>下一页 &gt;</button>
        </div>
      </div>

    </div>
  </div>
</template>


<script setup>
import { ref } from 'vue'
// 当前医生姓名
const doctorName = ref('孙思邈')
// 查询表单默认值（工厂函数：查询/重置共用一份，避免字段不一致）
const createDefaultForm = () => ({
  phone: '',
  name: '',
  gender: '', // '' 表示不限
  packageType: '',
  date: '',
  archive: 'false' // 'false' 未归档，'true' 已归档
});
// 查询表单数据
const searchForm = ref(createDefaultForm());
// 原始数据（isArchived: 是否归档）
const originalList = ref([
  { id: '100569833', phone: '12345671111', name: '叶文洁', gender: 'female', packageType: '女士-肾病检查', hospital: '沈阳熙康云医院-和平院区', date: '2022-01-19', isArchived: false },
  { id: '100569832', phone: '12345678222', name: '东方延绪', gender: 'female', packageType: '女士-肾病检查', hospital: '沈阳熙康云医院-浑南院区', date: '2022-01-18', isArchived: false },
  { id: '100569831', phone: '12345678111', name: '托马斯·维德', gender: 'male', packageType: '男士-肝病检查', hospital: '沈阳熙康云医院-浑南院区', date: '2022-01-18', isArchived: true },
  { id: '100569830', phone: '12345679999', name: '杨冬', gender: 'female', packageType: '女士-肝病检查', hospital: '沈阳熙康云医院-和平院区', date: '2022-01-17', isArchived: true },
  { id: '100569829', phone: '12345678888', name: '丁仪', gender: 'male', packageType: '男士-肝病检查', hospital: '沈阳熙康云医院-浑南院区', date: '2022-01-17', isArchived: false },
  { id: '100569828', phone: '12345677777', name: '程心', gender: 'female', packageType: '女士-肾病检查', hospital: '沈阳熙康云医院-和平院区', date: '2022-01-17', isArchived: false },
  { id: '100569827', phone: '12345676666', name: '云天明', gender: 'male', packageType: '男士-肾病检查', hospital: '沈阳熙康云医院-浑南院区', date: '2022-01-17', isArchived: true },
  { id: '100569826', phone: '12345675555', name: '章北海', gender: 'male', packageType: '男士-基础套餐', hospital: '沈阳熙康云医院-和平院区', date: '2022-01-14', isArchived: true },
  { id: '100569825', phone: '12345674444', name: '罗辑', gender: 'male', packageType: '男士-肝病检查', hospital: '沈阳熙康云医院-浑南院区', date: '2022-01-13', isArchived: false },
  { id: '100569824', phone: '12345673333', name: '史强', gender: 'male', packageType: '男士-肾病检查', hospital: '沈阳熙康云医院-和平院区', date: '2022-01-12', isArchived: true },
]);
// 过滤后的数据
const filteredList = ref(originalList.value); 


// 处理查询
const handleSearch = () => {
  const f = searchForm.value
  filteredList.value = originalList.value.filter(item => {
    if (f.phone && !item.phone.includes(f.phone)) return false
    if (f.name && !item.name.includes(f.name)) return false
    if (f.gender && item.gender !== f.gender) return false
    if (f.packageType && item.packageType !== f.packageType) return false
    if (f.date && item.date !== f.date) return false
    if (f.archive && item.isArchived !== (f.archive === 'true')) return false
    return true
  })
}
// 处理重置
const handleReset = () => {
  searchForm.value = createDefaultForm()
  filteredList.value = originalList.value
}
// 处理编辑
const handleEdit = (item) => {
  console.log(item)
}

</script>



<style scoped>
/* =========================
       1. 整体页面
       ========================= */

* {
  box-sizing: border-box;
}

/* scoped 样式无法直接选中 body，需用 :global 穿透 */
:global(body) {
  margin: 0;
  background-color: #f0f2f5;
  /* 稍微加深一点背景色，突出主体 */
  font-family: Arial, "Microsoft YaHei", sans-serif;
  color: #666;
}

/* 容器样式 */
.container {
  width: 95%;
  min-height: 700px;
  /* 改为最小高度，防止内容溢出被截断 */
  margin: 40px auto;
  background-color: #eaf0f6;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  /* 添加阴影增加立体感 */
  border-radius: 4px;
  overflow: hidden;
  /* 确保圆角生效 */
}

/* 头部样式 - 修复布局 */
.header {
  height: 70px;
  background-color: hsl(9, 83%, 51%);
  display: flex;
  align-items: center;
  justify-content: space-between;
  /* 两端对齐 */
  padding: 0 22px;
}

.logo {
  font-size: 25px;
  font-weight: bold;
  color: white;
  /* 修改为白色，因为在红色背景上蓝色看不清 */
  display: flex;
  align-items: center;
}

.logo b {
  color: white;
}

.logo span {
  margin-left: 10px;
  color: white;
}

/* 医生信息移至 Header 内部 */
.doctor-info {
  color: white;
  font-size: 18px;
  font-weight: bold;
  background-color: rgba(0, 0, 0, 0.1);
  /* 轻微背景区分 */
  padding: 5px 15px;
  border-radius: 20px;
}

/* 主体区域 */
.main {
  display: flex;
  min-height: 630px;
}

/* 左侧查询区域 */
.sidebar-left {
  width: 260px;
  min-width: 260px;
  min-height: 630px;
  background-color: #d3dfeb;
  padding: 20px 18px;
  border-right: 1px solid #c0cdda;
}

.sidebar-left h2 {
  font-size: 18px;
  margin: 0 0 20px;
  color: #333;
}

/* 每一行查询条件 */
.form-item {
  display: flex;
  align-items: center;
  margin-bottom: 14px;
}

.form-item label.static-label {
  width: 58px;
  flex: 0 0 58px;
  text-align: right;
  margin-right: 10px;
  color: #555;
  font-size: 14px;
}

/* 输入框样式 */
.form-item input[type="text"],
.form-item select,
.form-item input[type="date"] {
  flex: 1;
  min-width: 0;
  height: 36px;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0 9px;
  color: #333;
  outline: none;
}

.form-item input:focus,
.form-item select:focus {
  border-color: #2196f3;
}

/* 性别/归档选择框容器 */
.radio-group {
  display: flex;
  align-items: center;
  gap: 15px;
  flex: 1;
}

.radio-group label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 14px;
}

.radio-group input {
  margin: 0 4px 0 0;
}

.form-item:nth-of-type(6) {
  align-items: flex-start;
}

.form-item:nth-of-type(6) .radio-group {
  flex-direction: column;
  align-items: flex-start;
  gap: 8px;
}

/* 查询按钮样式 */
.buttons {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;
}

.buttons button {
  border: none;
  min-width: 54px;
  padding: 9px 13px;
  color: white;
  font-size: 14px;
  border-radius: 4px;
  cursor: pointer;
  transition: opacity 0.2s;
}

.buttons button:hover {
  opacity: 0.9;
}

.search-btn {
  background-color: #2196f3;
}

.reset-btn {
  background-color: #e9a12b;
}

/* 右侧表格区域 */
.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 630px;
  padding: 20px;
  background-color: #fff;
  /* 表格区域白色背景 */
}

table {
  width: 100%;
  border-collapse: collapse;
  /* background-color: white; 移到这里由 .content 控制 */
}

th,
td {
  height: 48px;
  border-bottom: 1px solid #e5e5e5;
  text-align: left;
  padding: 0 13px;
  white-space: nowrap;
}

th {
  color: #777;
  font-weight: bold;
  background-color: #f9fafb;
  /* 表头浅灰背景 */
}

td {
  color: #555;
  font-size: 14px;
}

/* 鼠标放上去 */
tbody tr:hover {
  background-color: #f0f7ff;
}

/* 编辑按钮链接样式 */
.edit-link {
  color: #fff;
  cursor: pointer;
  text-decoration: none;
  font-weight: 500;
}

.edit-link:hover {
  color: #fff;
}

/* 分页器 */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: auto;
  padding-top: 30px;
  gap: 15px;
}

.pagination button {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  border-radius: 4px;
}

.pagination button:hover {
  background-color: #f5f5f5;
}
</style>
