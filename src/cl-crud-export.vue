<template>
	<el-button
		:size="size"
		:type="type"
		:plain="plain"
		:round="round"
		:circle="circle"
		:icon="icon"
		:loading="loading"
		:disabled="disabled"
		@click="toExport"
	>
		<slot>导出</slot>
	</el-button>
</template>

<script>
import { export_json_to_excel } from './utils/export2excel';
import { isArray, isFunction, isEmpty } from './utils/index';

export default {
	props: {
		filename: String,
		autoWidth: {
			type: Boolean,
			default: true
		},
		bookType: {
			type: String,
			default: 'xlsx'
		},
		header: [Function, Array],
		fields: Array,
		data: [Function, Array],
		size: {
			type: String,
			default: 'mini'
		},
		disabled: Boolean,
		type: String,
		plain: Boolean,
		round: Boolean,
		circle: Boolean,
		icon: String
	},

	data() {
		return {
			loading: false
		};
	},

	methods: {
		async toExport() {
			if (!this.$crud) {
				return console.error('未获取到 $crud，请注入该插件');
			}

			// cl-crud loaded
			const { app, ctx } = this.$crud;

			// 表格列
			const columns = app
				.getData('table.columns')
				.filter(e => !['selection', 'expand'].includes(e.type));

			// 字段
			const fields = isEmpty(this.fields)
				? columns.filter(e => !(e.filterExport || e['filter-export'])).map(e => e.prop)
				: this.fields;

			// 表头
			let header = [];

			if (isArray(this.header)) {
				header = this.header;
			} else if (isFunction(this.header)) {
				header = await this.header();
			} else {
				header = columns.filter(e => fields.includes(e.prop)).map(e => e.label);
			}

			// 数据
			let data = [];

			if (isArray(this.data)) {
				data = this.data;
			} else if (isFunction(this.data)) {
				data = await this.data();
			} else {
				data = app.getData('table.data');
			}

			// 过滤
			data = data.map(d => fields.map(f => d[f]));

			this.loading = true;

			// 导出 excel
			export_json_to_excel({
				header,
				data,
				filename: this.filename,
				autoWidth: this.autoWidth,
				bookType: this.bookType
			});

			this.loading = false;
		}
	}
};
</script>