<template>
	<div>
		<el-breadcrumb separator="/">
		  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
		  <el-breadcrumb-item>数据统计</el-breadcrumb-item>
		  <el-breadcrumb-item>数据列表</el-breadcrumb-item>
		</el-breadcrumb>
		
		<el-card>
			 <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
			    <div id="main" style="width: 750px;height:400px;"></div>
		</el-card>
	</div>
</template>

<script>
import * as echarts from 'echarts'
import _ from 'lodash'

export default {
	data() {
		return {
			// 需要合并的数据
			options: {
			            title: {
			                text: ''
			            },
			            tooltip: {},
			            legend: {
			                data:['销量']
			            },
			            xAxis: {
			                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
			            },
			            yAxis: {},
			            series: [{
			                name: '销量',
			                type: 'bar',
			                data: [5, 20, 36, 10, 10, 20]
			            }]
			        }
		}
	},
	created() {},
	async mounted() {
		 // 基于准备好的dom，初始化echarts实例
		        var myChart = echarts.init(document.getElementById('main'))
				
				const {data: res} = await this.$http.get('reports/type/1')
				
				if(res.meta.status !== 200) {
					return this.$message.error('获取折线数据失败!')
				}
				console.log(res.data);
		        // 4。指定图表的配置项和数据
				// const result =  _.merge(res.data, this.options)
		        // 5。使用刚指定的配置项和数据显示图表。
		        myChart.setOption(res.data)
	},
	methods: {}
}
</script>

<style lang="less" scoped>
</style>