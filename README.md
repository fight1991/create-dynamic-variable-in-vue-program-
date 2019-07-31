# create-dynamic-variable-in-vue-program-
## 在vue项目中动态创建变量
+ 场景
> 在数据录入时, 录入项又是动态创建的, 有的录入项涉及到字典的远程搜索, 为了使每个list(存储字典列表)互不影响,需要动态创建
```js
<el-table-column prop="unit" width="100" label="计量单位" align="center">
  <template slot-scope="scope">
    <div class="table-select align-c" v-if="optionsType === 'edit'">
      <el-select  v-model="scope.row.unit" placeholder="计量单位"
        filterable remote default-first-option clearable
        @focus="tipsFill('unitList','SAAS_SEA_UNIT', 'unitR'+ scope.$index)"
        :remote-method="checkParamsList"
        style="width:100%">
        <el-option
          v-for="item in unitList['unitR'+ scope.$index]" // 动态创建unitList属性
          :key="item.codeField"
          :label="item.nameField"
          :value="item.codeField">
        </el-option>
      </el-select>
    </div>
    <div class="cell-div" v-else>{{scope.row.unitValue || '-'}}</div>
  </template>
</el-table-column>
```
+ 当DOM渲染时,就有创建了 
```js
 unitList: {
    unitR0: ..
    unitR1: ...
 }
 ```
### 注意点
> 当利用远程搜索时,需要注意的是,这些动态创建的属性不是响应式的,要做如下处理
```js
checkParamsList (query) { // 远程搜索
  let {obj, params, index} = this.selectObj
  let temp = []
  if (query !== '') {
    let keyValue = query.toString().trim()
    let list = JSON.parse(localStorage.getItem(params))
    let filterList = []
    if (util.isEmpty(keyValue)) {
      temp = list.slice(0, 30)
    } else {
      filterList = list.filter(item => {
        let str = item.codeField + '-' + item.nameField
        return str.toLowerCase().indexOf(keyValue.toLowerCase()) > -1
      })
      temp = filterList.slice(0, 30)
    }
  } else {
    if (!util.isEmpty(JSON.parse(localStorage.getItem(params)))) {
      temp = JSON.parse(localStorage.getItem(params)).slice(0, 30)
    }
  }
  // 添加响应式
  if (index) {
    this[obj][index] = temp
    this.$delete(this[obj], index) // 先删除
    this.$set(this[obj], index, temp) // 后添加
  } else {
    this[obj] = temp
  }
},
// 创建字典参数列表
tipsFill (obj, params, index) {
  this.selectObj = {
    obj,
    params,
    index
  }
}
