<template>
	<view class="uni-data-checklist">
		<checkbox-group v-if="multiple" class="checklist-group" :class="{'is-list':mode==='list','is-wrap':wrap}" @change="chagne">
			<label class="checklist-box" :class="item.labelClass" v-for="(item,index) in dataList" :key="index">
				<checkbox hidden :disabled="item.disable" :value="item.value+''" :checked="item.selected" />
				<!-- mode !== 'list' && -->
				<view v-if=" mode !=='tag' " class="checkbox__inner" :class="item.checkboxBgClass">
					<view class="checkbox__inner-icon" :class="item.checkboxClass"></view>
				</view>
				<view class="checklist-content">
					<text class="checklist-text" :class="item.textClass">{{item.text}}</text>
					<view v-if="mode === 'list'" class="checkobx__list" :class="item.listClass"></view>
				</view>
			</label>
		</checkbox-group>
		<radio-group v-else class="checklist-group" :class="{'is-list':mode==='list','is-wrap':wrap}" @change="chagne">
			<label class="checklist-box" :class="item.labelClass" v-for="(item,index) in dataList" :key="index">
				<radio hidden :disabled="item.disable" :value="item.value+''" :checked="item.selected" />
				<view v-if="mode !=='tag' " class="radio__inner" :class="item.checkboxBgClass">
					<view class="radio__inner-icon" :class="item.checkboxClass"></view>
				</view>
				<view class="checklist-content">
					<text class="checklist-text" :class="item.textClass">{{item.text}}</text>
					<view v-if="mode === 'list'" class="checkobx__list" :class="item.listClass"></view>
				</view>
			</label>
		</radio-group>
	</view>
</template>

<script>
	export default {
		name: 'uniDataChecklist',
		props: {
			// 模s式，可选值 default row ，list，button，tag
			mode: {
				type: String,
				default: 'default'
			},
			multiple: {
				type: Boolean,
				default: false
			},
			// 默认值 ,如果不指定，则取 range 的 selected 属性
			value: {
				type: [Array, String, Number],
				default () {
					return ''
				}
			},
			// 范围
			range: {
				type: Array,
				default () {
					return []
				}
			},
			// 最大值
			min: {
				type: [Number, String],
				default: ''
			},
			// 最小值
			max: {
				type: [Number, String],
				default: ''
			},
			wrap: {
				type: Boolean,
				default: false
			},
			// style:{
			// 	type : Object,
			// 	default(){
			// 		return {}
			// 	}
			// }
		},
		watch: {
			range: {
				handler(newVal) {
					this.dataList = this.getDataList(this.getSelectedValue(newVal))
				},
				deep: true
			},
			value(newVal) {
				this.dataList = this.getDataList(newVal)
			}
		},
		data() {
			return {
				dataList: [],
				styles: {
					selectedBackgroudColor: 'red',
					selectedColor: 'blue',
					backgroundColor: '#ffffff',
					color: '#333'
				}
			};
		},
		created() {
			this.dataList = this.getDataList(this.getSelectedValue(this.range))
		},
		methods: {
			chagne(e) {
				const values = e.detail.value

				let detail = {
					value: [],
					data: []
				}

				if (this.multiple) {
					this.range.forEach(item => {
						if (values.includes(item.value + '')) {
							detail.value.push(item.value)
							detail.data.push(item)
						}
					})
				} else {
					const range = this.range.find(item => (item.value + '') === values)
					if (range) {
						detail = {
							value: range.value,
							data: range
						}
					}
				}

				this.$emit('input', detail.value)
				this.$emit('change', {
					detail
				})

				if (this.multiple) {
					// 如果 v-model 没有绑定 ，则走内部逻辑
					if (this.value.length === 0) {
						this.dataList = this.getDataList(detail.value, true)
					}
				} else {
					this.dataList = this.getDataList(detail.value)
				}
			},
			getLabelClass(item, index) {
				let classes = []
				switch (this.mode) {
					case 'default':
						item.disable && classes.push('disabled-cursor')
						break
					case 'button':
						classes.push(...['is-button', ...this.getClasses(item, 'button')])
						break
					case 'list':
						if (this.multiple) {
							classes.push('is-list-multiple-box')
						} else {
							classes.push('is-list-box')
						}
						item.disable && classes.push('is-list-disabled')
						index !== 0 && classes.push('is-list-border')
						break
					case 'tag':
						classes.push(...['is-tag', ...this.getClasses(item, 'tag')])
						break
				}
				classes = classes.join(' ')
				return classes
			},
			getCheckboxClass(item, type = '') {
				let classes = []
				if (this.multiple) {
					classes.push(...this.getClasses(item, 'default-multiple', type))
				} else {
					classes.push(...this.getClasses(item, 'default', type))
				}
				classes = classes.join(' ')
				return classes
			},
			getTextClass(item) {
				let classes = []
				switch (this.mode) {
					case 'default':
						classes.push(...this.getClasses(item, 'list'))
						break
					case 'button':
						classes.push(...this.getClasses(item, 'list'))
						break
					case 'list':
						classes.push(...this.getClasses(item, 'list'))
						break
					case 'tag':
						classes.push(...['is-tag-text', ...this.getClasses(item, 'tag-text')])
						break
				}
				classes = classes.join(' ')
				return classes
			},

			/**
			 * 获取渲染的新数组
			 * @param {Object} value 选中内容
			 */
			getDataList(value) {
				// 解除引用关系，破坏原引用关系，避免污染源数据
				let dataList = JSON.parse(JSON.stringify(this.range))
				let list = []
				if (this.multiple) {
					if (!Array.isArray(value)) {
						value = []
						// console.error('props 类型错误');
					}
				}
				dataList.forEach((item, index) => {
					if (this.multiple) {
						if (value.length > 0) {
							let have = value.find(val => val === item.value)
							item.selected = have !== undefined
						} else {
							item.selected = false
						}
					} else {
						item.selected = value === item.value
					}

					list.push(item)
				})
				return this.setRange(list)
			},
			/**
			 * 处理最大最小值
			 * @param {Object} list
			 */
			setRange(list) {
				let selectList = list.filter(item => item.selected)
				let min = Number(this.min) || 0
				let max = Number(this.max) || ''
				list.forEach((item, index) => {
					if (this.multiple) {
						if (selectList.length <= min) {
							let have = selectList.find(val => val.value === item.value)
							if (have !== undefined) {
								item.disable = true
							}
						}

						if (selectList.length >= max && max !== '') {
							let have = selectList.find(val => val.value === item.value)
							if (have === undefined) {
								item.disable = true
							}
						}
					}
					this.setClass(item, index)
					list[index] = item
				})
				return list
			},
			/**
			 * 设置 class
			 * @param {Object} item
			 * @param {Object} index
			 */
			setClass(item, index) {
				// 设置 label 的 class
				item.labelClass = this.getLabelClass(item, index)
				// 设置 checkbox外层样式
				item.checkboxBgClass = this.getCheckboxClass(item, '-bg')
				// 设置 checkbox 内层样式
				item.checkboxClass = this.getCheckboxClass(item)
				// 设置文本样式
				item.textClass = this.getTextClass(item)
				// 设置 list 对勾右侧样式
				item.listClass = this.getCheckboxClass(item, '-list')
			},
			/**
			 * 获取 class
			 */
			getClasses(item, name, type = '') {
				let classes = []
				item.disable && classes.push('is-' + name + '-disabled' + type)
				item.selected && classes.push('is-' + name + '-checked' + type)

				if(this.mode !== 'button' || name === 'button'){
					item.selected && item.disable && classes.push('is-' + name + '-disabled-checked' + type)
				}

				return classes
			},
			/**
			 * 获取选中值
			 * @param {Object} range
			 */
			getSelectedValue(range) {
				if (!this.multiple) return this.value
				let selectedArr = []
				range.forEach((item) => {
					if (item.selected) {
						selectedArr.push(item.value)
					}
				})
				return this.value.length > 0 ? this.value : selectedArr
			}
		}
	}
</script>

<style>
	.checklist-group {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
	}

	.checklist-box {
		display: flex;
		flex-direction: row;
		align-items: center;
		margin: 5px 0;
		margin-right: 25px;
	}

	.checklist-text {
		font-size: 14px;
		color: #333;
		margin-left: 5px;
		transition: color 0.2s;
	}

	.is-button {
		margin-right: 10px;
		padding: 3px 15px;
		border: 1px #DCDFE6 solid;
		border-radius: 3px;
		transition: border-color 0.2s;
	}

	.is-list {
		flex-direction: column;
	}

	.is-list-box {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		padding: 10px 15px;
		padding-left: 0;
		margin: 0;
	}

	.checklist-content {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex: 1;
		flex-direction: row;
		align-items: center;
		justify-content: space-between;
	}

	.is-list-multiple-box {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		padding: 10px 15px;
		padding-left: 0;
		margin: 0;
	}

	.is-list-border {
		border-top: 1px #eee solid;
	}

	.is-tag {
		margin-right: 10px;
		padding: 3px 10px;
		border: 1px #eee solid;
		border-radius: 3px;
		background-color: #f5f5f5;
		transition: border-color 0.2s;
	}

	.is-tag-text {
		margin: 0;
		color: #666;
	}

	.checkbox__inner {
		flex-shrink: 0;
		position: relative;
		border: 1px solid #DCDFE6;
		border-radius: 2px;
		box-sizing: border-box;
		width: 16px;
		height: 16px;
		background-color: #fff;
		z-index: 1;
		transition: border-color 0.1s;
	}

	.checkbox__inner-icon {
		border: 1px solid #fff;
		border-left: 0;
		border-top: 0;
		height: 8px;
		left: 5px;
		position: absolute;
		top: 1px;
		width: 3px;
		opacity: 0;
		transition: transform .1s;
		transform-origin: center;
		transform: rotate(40deg) scaleY(0.4);
	}

	.radio__inner {
		flex-shrink: 0;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		justify-content: center;
		align-items: center;
		position: relative;
		border: 1px solid #DCDFE6;
		border-radius: 2px;
		box-sizing: border-box;
		width: 16px;
		height: 16px;
		border-radius: 16px;
		background-color: #fff;
		z-index: 1;
		transition: border-color .3s;
	}

	.radio__inner-icon {
		width: 8px;
		height: 8px;
		border-radius: 10px;
		opacity: 0;
		transition: transform .3s;
	}

	.checkobx__list {
		border: 1px solid #fff;
		border-left: 0;
		border-top: 0;
		height: 12px;
		width: 6px;
		transform-origin: center;
		opacity: 0;
		transition: all 0.2s;
	}


	/* 禁用样式 */
	.is-default-disabled-bg {
		background-color: #F2F6FC;
		border-color: #DCDFE6;
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
	}

	.is-default-multiple-disabled-bg {
		background-color: #F2F6FC;
		border-color: #DCDFE6;
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
	}

	.is-default-disabled {
		border-color: #F2F6FC;
	}

	.is-default-multiple-disabled {
		border-color: #F2F6FC;
	}

	.is-list-disabled {
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
		color: #999;
	}

	.is-list-disabled-checked {
		color: #a1dcc1;
	}


	.is-button-disabled {
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
		border-color: #EBEEF5;
	}

	.is-button-text-disabled {
		color: #C0C4CC;
	}

	.is-button-disabled-checked {
		border-color: #a1dcc1;
	}

	.is-tag-disabled {
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
		border-color: #e9e9eb;
		background-color: #f4f4f5;
	}

	.is-tag-text-disabled {
		color: #bcbec2;
	}

	/* 选中样式 */
	.is-default-checked-bg {
		border-color: #007aff;
	}

	.is-default-multiple-checked-bg {
		border-color: #007aff;
		background-color: #007aff;
	}

	.is-default-checked {
		opacity: 1;
		background-color: #007aff;
		transform: rotate(45deg) scaleY(1);
	}

	.is-default-multiple-checked {
		opacity: 1;
		transform: rotate(45deg) scaleY(1);
	}

	.is-default-disabled-checked-bg {
		opacity: 0.4;
	}

	.is-default-multiple-disabled-checked-bg {
		opacity: 0.4;
	}


	.is-default-checked-list {
		border-color: #007aff;
		opacity: 1;
		transform: rotate(45deg) scaleY(1);
	}

	.is-default-multiple-checked-list {
		border-color: #007aff;
		opacity: 1;
		transform: rotate(45deg) scaleY(1);
	}
	.is-list-disabled-checked {
		opacity: 0.4;
	}

	.is-default-disabled-checked-list {
		opacity: 0.4;
	}

	.is-default-multiple-disabled-checked-list {
		opacity: 0.4;
	}
	.is-button-checked {
		border-color: #007aff;
	}

	.is-button-disabled-checked {
		opacity: 0.4;
	}

	.is-list-checked {
		color: #007aff;
	}

	.is-tag-checked {
		border-color: #007aff;
		background-color: #007aff;
	}

	.is-tag-text-checked {
		color: #fff;
	}

	.is-tag-disabled-checked {
		opacity: 0.4;
	}


	.disabled-cursor {
		/* #ifdef H5 */
		cursor: not-allowed;
		/* #endif */
	}

	.is-wrap {
		flex-direction: column;
	}
</style>
