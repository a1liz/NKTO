<template>
	<Card>
    <Row>
    <Col span="6"><img :src="imgUrl" height="180px"></Col>
      <Col span="15" >
        <div>
          <h2>
            {{name}}
          </h2>
        </div>
        <div style ="float: right;margin-top:20px;font-size:15px" v-if="showprice">
          交易价格：{{price}}
        </div>
        <div style ="margin-top: 50px">
          <h4 v-if="showdealtime">
          订单交易时间：{{dealtime}}
          </h4>
          <h4 v-if="showdealpos">
          订单交易地点：{{dealpos}}
          </h4>
          <h4 v-if="showuptime">
          	上传时间：{{uptime}}
          </h4>
        </div>
        <div v-if="showcate">
          分类：{{cate}}
        </div>
        <div v-if="showstate">
          状态：{{state === 'finished' ? '已完成' : '未完成'}}
        </div>
      </Col>
      <Col span="8" class="finished" v-if="showstate  === true && state === 'finished'">
        <img src="/static/img/finished-red.png" alt="" class="finished-red">
      </Col>
    </Row>
  </Card>
</template>
<script>
	export default {
		props: ['name', 'dealtime', 'dealpos', 'cate', 'state', 'uptime', 'price', 'imgUrl'],
		data () {
			return {
				// 由于item组件要给其他不同的组件调用，所以下面的字段会根据上面的type来显示
				showname: true,
				showdealtime: true,
				showdealpos: true,
				showcate: true,
				showstate: true,
				showuptime: true,
        showprice: true,
			}
		},
		created () {
			// 根据id和type从数据库获取商品信息
			if (this.type === 'order') {
				this.showuptime = false
			} else if (this.type === 'history') {
				this.showstate = false
				this.showdealtime = false
				this.showdealpos = false
			}
		}
	}
</script>
<style scoped>
.finished-red {
  width: 200px;
  height: 200px;
}
.finished {
  position: absolute;
  right: 130px;
}
</style>