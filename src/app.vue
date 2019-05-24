<template>
<div class="picker">
	<label class="picker_label">地址：</label>
	<div>
		<div class="picker__text" @click="toggleDropdown" >
			<input type="text" class="picker__input" placeholder="请选择" size="middle" readonly v-bind="$attrs" :value="finish" />
		</div>
		<div class="picker__masklayer" v-show="showPanel" @click="toggleDropdown"></div>
		<div class="picker__select" v-if="showPanel">
			<ul class="picker__select-header">
				<li v-for="(item, index) in current" @click="showCurrent(item, index)" :class="index == currentIndex ? 'actived' : ''">{{item[label] ? item[label]: item.name}}</li>
			</ul>
			<ul class="picker__select-content">
				<li v-for="item in showList" @click="getNext(item)" :class="current[currentIndex][clue] == item[clue] ? 'actived' : '' ">{{item[label] ? item[label] : item}}</li>
			</ul>
		</div>
	</div>
</div>
</template>

<script type="text/ecmascript-6">
//不管是一次性加载所有的数据还是分块加载，都需要对数据进行处理。

//如果一次性加载所有的数据，需要用户定义二级名称
import data from "./data.js";
export default {
    name: 'TreePicker',
	props: {
		value: {
			type:String,
			default:'',
		},
		initShowList: {
			type: Array,
			default: () => []
		},
		level:{
			type: Number,
			default: 3,
		}, //省市县层级，省；省市；省市县
		clue: {
			type: String,
			default: "id"
		},
		pid: {
			type: String,
			default: "parent_id"
		},
		initlist: {
			type: Array,
			default: () => []
		},
		label: {
			type: String,
			default: "area_name"
		},
		mode: {
			type: String,
			default: "0"
		},
		child: {
			type: String,
			default: "child"
		},
		prop: String
	},
	data() {
		return {
			showPanel: false,
			list: {},
			showList: [],
			currentIndex: 0, // 1,2,3
			finishArr: [],
			current: [
				{
					name: "请选择"
				}
			],
			finish: ""
		};
	},
	created() {
		//触发用户定义的promise事件；
		//this.showList = await this.getProvince();
		// 初始化回显
		this.getInitList();
		console.log(this.initlist);
		if (Array.isArray(this.initShowList) && this.initShowList.length > 0) {
			this.current = [].concat(this.initShowList);
			this.finish = this.current
				.map(item => (item[this.label] ? item[this.label] : ""))
				.join(" ");
		}
		if (this.mode === "0") {
			this.list = data;
			this.showList = this.list;
			this.handlerRecover();
			return;
		} else if (this.mode === "1" || this.mode === "2") {
			this.showCurrent(this.current[0], 0);
		}
	},
	watch: {
		finish(val) {
			this.$emit("input", val);
		},
		showPanel(val){
			if(val){
				this.$emit('onshow');
			}else{
				this.$emit('onhide');
			}
		},
		initlist: function() {
			this.showList = this.initlist;
			//once load data
			if (this.mode === "1") {
				this.list = this.orgData(this.initlist, 0);
				//存在原始值，读区值
				this.handlerRecover();
				return;
			}
			let { currentIndex } = this;
			switch (currentIndex) {
				case 0:
					this.list = this.convert(this.initlist);
					this.showList = this.list;
					break;
				case 1:
					let opts = this.current[0];
					let city = this.initlist;
					this.list[String(opts[this.clue]).trim()][
						this.child
					] = this.convert(city);
					this.showList = city;
					break;
				case 2:
					let _city = Object.assign({}, this.current[1]);
					let county = this.initlist;
					const province = this.current[0][this.clue];
					this.list[province][this.child][_city[this.clue]][
						this.child
					] = this.convert(county);
					this.showList = county;
			}
		}
	},
	methods: {
		getInitList(){
			let address = this.value.split(';');
			for(let i in data){
				if(data[i].area_name === address[0]){
					this.initShowList.push({ id: i, area_name: address[0] });
				}
				if(data[i].child && address[1]){
					for(let j in data[i].child){
						if(data[i].child[j].area_name === address[1]){
							this.initShowList.push({ id: j, area_name: address[1] });
						}
						let county = data[i].child[j].child;
						if(county && address[2]){
							for(let k in county){
								if(county[k].area_name === address[2]){
									this.initShowList.push({ id: k, area_name: address[2] });
								}
							}
						}
					}
				}
			}
		},
		handlerRecover() {
			const ctl = this.current.length;
			switch (ctl) {
				case 2:
					this.getCity(this.current[0]);
					this.currentIndex = 1;
					break;
				case 3:
					this.getCounty(this.current[1]);
					this.currentIndex = 2;
					break;
			}
			return;
		},
		closeDropdown() {
			this.showPanel = false;
		},
		toggleDropdown() {
			this.showPanel = !this.showPanel;
		},
		//重组数据，
		orgData(data, parent_id) {
			var tree = {};
			var temp;
			for (var i = 0; i < data.length; i++) {
				if (data[i][this.pid] === parent_id) {
					var obj = data[i];
					temp = this.orgData(data, data[i][this.clue]);
					if (JSON.stringify(temp) != "{}") {
						obj[this.child] = temp;
					}
					tree[data[i][this.clue]] = obj;
				}
			}
			return tree;
		},
		showCurrent(item, index) {
			this.currentIndex = index;
			switch (index) {
				case 0: {
					this.getProvince();
					break;
				}
				case 1: {
					this.getCity(this.current[0]);
					break;
				}
				case 2: {
					this.getCounty(this.current[1]);
					break;
				}
				default: {
					console.error("传入参数错误");
				}
			}
		},

		getNext(opts) {
			const index = this.currentIndex;
			switch (index) {
				case 0: {

					const current = [].concat(this.current);
					current[0] = Object.assign(opts);
					if (opts[this.clue] !== this.current[0][this.clue]) {
						this.current = current.slice(0, 2);
					} else {
						this.current = current;
					}
					if(this.level == 1){
						this.getValue();
						return false;
					}
					this.getCity(opts);
					current[1] = Object.assign({
						name: "请选择"
					});
					this.current = current;
					this.currentIndex = 1;
					break;
				}
				case 1: {
					this.current[1] = Object.assign({}, opts);
					if(this.level == 2){
						this.getValue();
						return false;
					}
					this.current[2] = Object.assign({
						name: "请选择"
					});
					this.current = [].concat(this.current);
					this.currentIndex = 2;
					this.getCounty(opts);
					break;
				}
				case 2: {
					this.current[2] = Object.assign({}, opts);
					this.current = [].concat(this.current);
					this.getValue();
					break;
				}
				default: {
					console.error("传入参数错误");
				}
			}
		},
		getValue(){
			this.showPanel = false;
			this.finishArr = [];
			this.finish = this.current
					.map(item => (item[this.label] ? item[this.label] : ""))
					.join(" ");
			this.current.map((item, index) => {
				this.finishArr.push({
					[this.clue]: item[this.clue],
					[this.label]: item[this.label]
				});
			});
			this.$emit("onchange", this.finishArr);
		},
		convert(list) {
			const data = {};
			if (Array.isArray(list)) {
				list.forEach(item => {
					data[String(item[this.clue]).trim()] = item;
				});
			}
			return data;
		},
		reconvert(obj) {
			const data = [];
			for (const i in obj) {
				data.push(obj[i]);
			}
			return data;
		},
		getAllData() {
			this.getAddress(opts);
		},
		getProvince() {
			if (this.list && JSON.stringify(this.list) != "{}") {
				this.showList = this.list;
				return;
			}
			let opts = { level: 1 };
			if(this.level == 1){
				this.showPanel = false;
				return false;
			}
			this.getAddress(opts);
		},
		getCity(opts) {
			if (
				this.list &&
				this.list[String(opts[this.clue]).trim()][this.child]
			) {
				const data = this.list[String(opts[this.clue]).trim()][
					this.child
				];
				this.showList = data;
				return;
			}
			let _opts = { level: 2, level_code: opts[this.clue] };
			if(this.level == 2){
				this.showPanel = false;
				return false;
			}
			this.getAddress(_opts);
		},

		getCounty(opts) {
			const city = Object.assign({}, opts);
			const province = this.current[0][this.clue];
			if (
				this.list &&
				this.list[province][this.child] &&
				this.list[province][this.child][city[this.clue]] &&
				this.list[province][this.child][city[this.clue]][this.child]
			) {
				const data = this.list[province][this.child][city[this.clue]][
					this.child
				];
				this.showList = data;
				return this.reconvert(data);
			}

			let _opts = { level: 3, level_code: city[this.clue] };
			this.getAddress(_opts);
		},
		getAddress(opts) {
			this.$emit("onload", opts);
			return;
		}
	}
};
</script>
<style lang="less">
.picker {
    position: relative;
    display: flex;
    margin: 10px;
    font-size: 12px;
	&_label {
		height: 28px;
		line-height: 28px;
	}
	&__text {
		width: auto;
		padding-left: 8px;
		cursor: pointer;
		overflow: hidden;
	}
	&__input {
		border: 1px solid #ccc;
		line-height: 28px;
		height: 28px;
		border-radius: 2px;
		padding-left: 5px;
		outline: none;
		min-width: 150px;
	}
	&__masklayer {
		position: fixed;
		background: #ccc;
		left: 0px;
		top: 0px;
		right: 0px;
		bottom: 0px;
		opacity: 0.4;
	}
	&__select {
		position: absolute;
		left: 0;
		top: 0;
		margin-top: 32px;
		width: 630px;
		background-color: white;
		z-index: 10;
		border: 1px solid #d1d1d1;
	}
	&__select-header {
		display: flex;
        border-bottom: 1px solid #d1d1d1;
        margin: 0px;
        padding: 0px;
		li {
            padding: 10px 10px;
            list-style:none;
			border-right: 1px solid #d1d1d1;
			cursor: pointer;
		}
		.actived {
			color: white;
			background-color: #49a9ee;
		}
	}
	&__select-content {
		width: 100%;
		display: flex;
		padding: 10px;
		flex-flow: wrap;
		li {
			width: 150px;
            line-height: 28px;
            list-style: none;
			cursor: pointer;
			&:hover {
				color: #49a9ee;
				// font-size: 14px;
				font-weight: bold;
			}
		}
		.actived {
			color: #49a9ee;
			font-weight: bold;
		}
	}
}
</style>
