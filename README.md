# tree-picker

> tree-picker is sample Three-level linkage selection, you can use it to choose area, department,or personnel,

## 安装

```
npm i tree-picker
```

#### 使用指南
##### 使用默认数据,
可以获取通过onchange方法，获取更改后的数据。

```
<template>
    <treePicker @onchange="getChoosed"/>
</template>

<script>
import treePicker from 'tree-picker';
export default {
	components: {
		treePicker,
	},
	data() {
		return {};
	},
	methods: {
		getChoosed(val) {
			console.log(val);
		}
	}
};
</script>
```
##### 设置默认值,

```
<tree-picker
            value="陕西省 西安市 新城区"
            split=" "
            :level=3
	    	@change ="getChoosed"
            @hide="areaHide"
			@show="areaShow"
    	></tree-picker>

   export default {
        data() {
            return {};
        },
        methods: {
            getChoosed(val) {
                console.log(val);
            },
            areaHide() {
                console.log('隐藏');
            },
            areaShow() {
                console.log('显示');
            },
        },
        components: { 'tree-picker': treePicker },
    };
```



##### 一次性加载所有数据
注意：mode = 1,可以使用v-model获取修改后的数据；
```
<template>
    <tree-picker
			value=""
	    	:data = "getAreas"
	    	:mode=1
	    	child="child",
	    	pid="parent_id",
	    	orgdata={true}
	    	@change ="getChoosed">
    	</tree-picker>
</template>

<script>
import treePicker from 'tree-picker';
export default {
	components: {
		treePicker,
	},
	data() {
		return {};
	},
	methods: {
		getChoosed(val) {
			console.log(val);
		},
		getAreas(){
		    //返回一个promise
		    return Promise
		}
	}
};
</script>
```


##### 分块加载数据
注意：设置mode=2;
```
<template>
    <tree-picker
	    	:data = "getAreas"
	    	:mode=2
	    	@change ="getChoosed">
    	</tree-picker>
</template>

<script>
import treePicker from 'tree-picker';
export default {
	components: {
		treePicker,
	},
	data() {
		return {};
	},
	methods: {
		getChoosed(val) {
			console.log(val);
		},
		getAreas(){
		    //返回一个promise
		    return Promise
		}
	}
};
</script>
```

##### 兼容form-item
注意：vbeauty = true; prop设置成关联字段名称，v-model设置成关联字段，具体用法和form-item用法一样。
```
<template>
    <v-form :model="customForm" :rules="customRules" ref='customRuleForm'>
        <tree-picker
	    	:data = "getAreas"
	    	:initShowList="customForm.address"
	    	:vbeauty=true
	    	:mode=2
	    	prop="address"
	    	v-model="customForm.address"
	    	@change ="getChoosed">
    	</tree-picker>
    </v-form>
</template>

<script>
import treePicker from 'tree-picker';
export default {
	components: {
		treePicker,
	},
	data() {
	    //可参照form-item设置自定义验证规则
		return {
		    customForm: {
				[
					{ code: "360000", name: "江西省" },
					{ code: "360100", name: "南昌市" },
					{ code: "360101", name: "直辖区" }
				]
			},
			customRules: {
				address: [
					{
						required: true,
						message: "请选择地址"
					}
				]
			},
		};
	},
	methods: {
		getChoosed(val) {
			console.log(val);
		},
		getAreas(){
		    //返回一个promise
		    return Promise
		}
	}
};
</script>
```



#### API
##### tree-picker props

参数  | 说明 | 类型 | 默认值
---|---|---|---|
 data | 获取展示数据的函数，初始化必需参数，返回值必须是Promise对象,该函数默认接收一个请求参数 | Function | -
 value | 指定默认值，可以为"四川省,成都市,武侯区",如果分隔符非;时，需要传递下面一个参数 | String | -
 split | 当传递的默认值为"四川省 成都市 武侯区"这种格式时，指定数据分割方式， | String | ;
 level | 指定选择层级，"level=1；省。level=2；省市。level=3；省市县" | Number | 3
 mode | 0:使用默认数据；1:一次加载所有数据；2:分块加载数据 | Number | 0
 clue | 选项的id | String | id
 name | 选项的名称 | String | area_name
 label | 标签名 | String | "地址"
 initShowList | 当存在value时，value的优先级高于initShowList 设置默认值 | Array | -
 pid | 当一次性加载数据时，重组数据时的关联字段 | String | parent_id
 child | 数据层级名称 | String | child
 orgdata | 是否使用默认重组数据方法 | Boolean | false
 vbeauty | 是否使用form-item,如果为true,则可以设置表单验证内容 | Boolean | false
 v-model| 双向绑定数据【兼容form-item时需要】| - | -
 prop | form-item 的prop属性，配合vbeauty=true使用 | String | -




#####  area-picker Method

事件  | 说明 | 参数
---|---|---
change | 选择的值发生变化的时候触发，默认返回选中区域数组 | value
hide | 隐藏选择面板时触发 | value
show | 打开选择面板时触发 | value












