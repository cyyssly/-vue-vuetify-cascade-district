# vuetify-cascade-district
中国行政区划级联选择器，基于vuetify

### 1. 前置/Dependencies：vue、vuetify

### 2. 特性/Features:
(1) 定制好的省、市、区三级行政区划
(2) 包含手工整理的港澳台地区详细区划
(3) 异步加载数据
(4) 后文含数据更新方法

![image](https://github.com/cyyssly/vue-vuetify-cascade/blob/master/1.JPG)

### 3. 使用方法/Instructions: 

#### 3.1. 复制 Cascade-district.vue 到项目目录下，例如 src/components/ 下。

#### 3.2. 调用/Use：
```vue
  <v-menu v-model="item.menu" :close-on-content-click="false">
    <template v-slot:activator="{ on }">
      <v-text-field
        v-model="item.model"
        :label="item.label"
        readonly
        v-on="on"
      ></v-text-field>
    </template>
    <CascadeD
      v-model="item.model"
      :Height="item.height"
      :ApiHref="item.api"
      @input="item.menu = !item.menu"
    ></CascadeD>
  </v-menu>  
```
```js
<script>
import CascadeD from '@/components/Cascade-district.vue'

export default {
  components: {
    CascadeD
  },
  data: () => ({
    item: {
      label: "省市区",
      height: "300px",
      api: "/console/checkarea", // 默认的后端接口 API 地址，请根据你的项目实际情况修改
      model: "", // 默认值
      menu: false // 是否展开组件
    }
  })
</script>
```

### 4. 源码解析
Cascade 接受2个传入的参数，并通过 input 事件传出用户选择结果。  

#### 4.1. 入参

(1) Height: String  
组件高度：当选项数量较多时，用于限制组件的高度，默认为300px，可选。  

(2) ApiHref: String  
获取后台数据的 API 地址。
组件访问地址时会提供以下参数：1.level：数据层级，从0开始, 数值；2.pid：上级父节点的id, 字符串；
API 需要实现的功能：根据传入的 pid 查询该节点的全部下级节点；
API 返回数据的格式要求是一个含有 id 和 text 属性的对象数组，例如：
```js
  [ {id:1, text:'item1'}, {id:2, text:'item2'}, {id:3, text:'item3'} ]  
```
后端伪代码示例：
```node
async function(req, res) {
  const strSQL = "SELECT `code` as id,`name` as text FROM extarea where `parentcode`=?";
  const param = [req.body["pid"]];
  try {
    const results = await MysqlPool.dataQuery(strSQL, param);
    res.send(results);
  } catch (err) {
    err => res.status(500).send(err);
  }
}
```
#### 4.2. 返回值

返回值为数组格式，分别是用户在每个级别选中的值，例如：
```js
['北京市','北京','朝阳区']  
```
### 4. 数据更新方法:

(1)下载原始数据  
民政部行政区划代码下载地址：http://www.mca.gov.cn/article/sj/xzqh  
页面没有提供下载功能，可以复制后直接粘贴到excel里。  
但这个数据不含港澳台地区的下辖区划，内地也有5个不分区的地级市。为统一使用体验，建议手工添加其下级行政区划。我已经整理放入 extraDistrict.xlsx 文件中(更新到2019年8月)，把这些数据复制粘贴到民政局数据后面就齐全了。  
(2)数据库建表  
以 mysql 为例：表名为 extarea，包含3个字段：
code varchar(45) PK 
name varchar(45) 
parentcode varchar(45)  
(3)导入并清洗数据
导入数据到 mysql 并进行以下处理，目的是把数据规范化，便于使用：
1. update  extarea set  code = replace(code,' ',''); 
2. update  extarea set  name = replace(name,' ',''); 
3. update extarea set  parentcode = CONCAT(left(code,2),'0000') where right(code,2) = '00' and right(code,4) <> '0000';
4. update extarea set  parentcode = CONCAT(left(code,4),'00') where right(code,2) <> '00';
5. insert into extarea values('110100','北京','110000'), ('120100','天津','120000'), ('310100','上海','310000'), ('500100','重庆','500000');
6. update extarea set  parentcode = '' where right(code,4) = '0000';
至此后台数据准备完毕。

不想自己动手的，请下载项目中的 fullDistrict.sql 直接导入 mysql 即可。（更新到2019年8月）
    
