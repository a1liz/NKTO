<template>
  <div id="app">
    <div class="head">
      <Input v-model="searchWord" @on-enter="search" placeholder="输入ID,邮箱,姓名查找用户" class="search-input"></Input>
      <Button @click="search">查询</Button>
      <div class="head-right">
      </div>
    </div>
    <Row class="table">
        <Table border :columns="userForm" :data="userDataShow" ref="table"></Table>
        <Page :total="userData.length" @on-change="changePage" :page-size="pageSize" style="margin-top:1vh;"></Page>
    </Row>
    <br>
    <Button type="primary" size="large" @click="exportData(1)"><Icon type="ios-download-outline"></Icon> 导出原始数据</Button>
    </div>
</template>
<script>
  export default {
    data () {
      return {
        stateMap: {
          '-1': '已被注销',
          '0': '未激活',
          '1': '不在线',
          '2': '休息中',
          '3': '工作中'
        }, // (要改)
        sortList: ['按关键字排序', '状态', '邮箱', '手机'], // 排序的所有关键字(要改)
        sortKeyWord: '按关键字排序', // 排序关键字
        sortOrder: '升序', // 升序或降序
        orderList: ['升序', '降序'],
        searchWord: '', // 搜索用户的关键词
        userEmail: '', // 邀请用户输入的邮箱
        userForm: [
          {
            'title': 'ID',
            'key': 'uid'
          }, {
            'title': '状态',
            'key': 'state'
          }, {
            'title': '邮箱',
            'key': 'email'
          }, {
            'title': '手机',
            'key': 'phone'
          }, {
            'title': '操作',
            'key': 'action',
            width: 150,
            align: 'center',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'primary',
                    size: 'small'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.changeState(params.index)
                    }
                  }
                }, '切换')
              ])
            }
          }
        ], // 用户表格格式
        userDataAll: [], // 所有用户数据
        userData: [], // 所有页面数据
        userDataShow: [], // 当前页的用户数据
        current: 1, // 当前页码
        pageSize: 10 // 每页数据条数
      }
    },
    methods: {
      init (iWantToChangePage) {
        // 根据当前情况将userDataAll中的数据传给userData和userDataShow
        this.userData = []
        if (this.searchWord.trim() === '') {
          this.userData = this.userDataAll
        } else {
          for (let i = 0; i < this.userDataAll.length; ++i) {
            if (this.userDataAll[i]['uid'].indexOf(this.searchWord) !== -1 ||
                this.userDataAll[i]['phone'].indexOf(this.searchWord) !== -1 ||
                this.userDataAll[i]['email'].indexOf(this.searchWord) !== -1) {
              this.userData.push(this.userDataAll[i])
            }
          }
        }
        if (iWantToChangePage) {
          this.userDataShow = this.userData.slice(0, Math.min(this.pageSize, this.userData.length))
          this.current = 1
        } else {
          this.userDataShow = this.userData.slice((this.current - 1) * this.pageSize, Math.min((this.current - 1) * this.pageSize + this.pageSize, this.userData.length))
        }
      },
      fetchBase (url, body) {
        return fetch(url, {
          method: 'post',
          credentials: 'same-origin',
          headers: {
            'X-CSRFToken': this.getCookie('csrftoken'),
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(body)
        })
        .then((res) => res.json())
      },
      checkEmail (email) {
        var ePattern = /^([A-Za-z0-9])+@([A-Za-z0-9])+\.([A-Za-z]{2,4})$/g
        return ePattern.test(email)
      },
      changePage (current) {
        this.current = current
        this.init(false)
      },
      changeState (index) {
        // 修改用户状态
        let uid = this.userDataShow[index].uid
        let state = this.userDataShow[index].state
        let newState = ''
        if (state === this.stateMap['-1']) {
          newState = this.stateMap['0']
        } else if (state === this.stateMap['0']) {
          newState = this.stateMap['1']
        } else if (state === this.stateMap['1']) {
          newState = this.stateMap['2']
        } else if (state === this.stateMap['2']) {
          newState = this.stateMap['3']
        } else if (state === this.stateMap['3']) {
          newState = this.stateMap['-1']
        }
        for (let i = 0; i < this.userDataAll.length; ++i) {
          if (this.userDataAll[i].uid === uid) {
            this.userDataAll[i].state = newState
            this.userDataShow[index].state = newState
            break
          }
        }
        // 向服务器发送修改用户状态请求(先空着)
        this.$Message.success('状态修改成功')
      },
      search () {
        // 根据信息查找用户，支持模糊查询
        this.init(true)
        this.searchWord = ''
      },
      changeSort () {
        // 排序（需要修改）
        let key = ''
        if (this.sortKeyWord === '状态') {
          key = 'state'
        } else if (this.sortKeyWord === '邮箱') {
          key = 'email'
        } else if (this.sortKeyWord === '手机') {
          key = 'phone'
        } else if (this.sortKeyWord === '按关键字排序') {
          return
        }
        let num = 1
        if (this.sortOrder === '降序') {
          num = -1
        }
        this.userData.sort((item1, item2) => {
          if (item1[key] > item2[key]) {
            return num
          } else if (item1[key] < item2[key]) {
            return -num
          } else {
            return 0
          }
        })
        this.init(true)
      },
      exportData (type) {
        // 根据传入的type导出不同的数据
        let csv = '\ufeff'
        let keys = []
        let temp = this.userDataAll
        if (type === 2) {
          temp = this.userData
        }
        this.userForm.forEach(function (item) {
          csv += '"' + item['title'] + '",'
          keys.push(item['key'])
        })
        csv = csv.replace(/,$/, '\n')
        temp.forEach(function (item) {
          keys.forEach(function (key) {
            csv += '"' + item[key] + '",'
          })
          csv = csv.replace(/,$/, '\n')
        })
        var blob = new window.Blob([csv], {
          type: 'text/csv,charset=UTF-8'
        })
        let csvUrl = window.URL.createObjectURL(blob)
        let a = document.createElement('a')
        a.download = '用户.csv'
        a.href = csvUrl
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
      },
      getCookie (cName) {
        if (document.cookie.length > 0) {
          let cStart = document.cookie.indexOf(cName + '=')
          if (cStart !== -1) {
            cStart = cStart + cName.length + 1
            let cEnd = document.cookie.indexOf(';', cStart)
            if (cEnd === -1) {
              cEnd = document.cookie.length
            }
            return unescape(document.cookie.substring(cStart, cEnd))
          }
        }
        return ''
      }
    },
    mounted () {
      // 初始化用户信息，这里应该是从数据库中获取到数据的，但是先手动模拟
      this.userDataAll = [
        {'uid': '001', 'email': 'email1', 'phone': 'phone1', 'state': this.stateMap['1']},
        {'uid': '002', 'email': 'email2', 'phone': 'phone2', 'state': this.stateMap['1']},
        {'uid': '003', 'email': 'email3', 'phone': 'phone3', 'state': this.stateMap['3']},
        {'uid': '004', 'email': 'email4', 'phone': 'phone4', 'state': this.stateMap['1']},
        {'uid': '005', 'email': 'email5', 'phone': 'phone5', 'state': this.stateMap['1']},
        {'uid': '006', 'email': 'email6', 'phone': 'phone6', 'state': this.stateMap['3']},
        {'uid': '007', 'email': 'email7', 'phone': 'phone7', 'state': this.stateMap['1']},
        {'uid': '008', 'email': 'email8', 'phone': 'phone8', 'state': this.stateMap['1']},
        {'uid': '009', 'email': 'email9', 'phone': 'phone9', 'state': this.stateMap['2']},
        {'uid': '010', 'email': 'email10', 'phone': 'phone10', 'state': this.stateMap['-1']},
        {'uid': '011', 'email': 'email11', 'phone': 'phone11', 'state': this.stateMap['0']},
        {'uid': '012', 'email': 'email12', 'phone': 'phone12', 'state': this.stateMap['1']}
      ]
      this.init(true)
    }
  }
</script>
<style scoped>
.search-input {
  width: 15%;
  height: 5%;
}
.table {
  margin-top: 2vh;
}
.head {
  display: block;
  width: 100%;
}
.head-left {
  display: inline-block;
  width: 35%;
}
.head-right {
  display: inline-block;
  position: absolute;
  right: 28px;
  width: 50%;
  text-align: right;
}
.head-right span {
  text-align: left;
}
</style>