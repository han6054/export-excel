> 引言：

最近使用vue在做一个后台系统，技术栈 `vue + iView`，在页面中生成表格后，
iView可以实现表格的导出，不过只能导出csv格式的，并不适合项目需求。

### 如果想要导出Excel
- 在src目录下创建一个文件(vendor)进入`Blob.js`和`Export2Excel.js`
- `npm install -S file-saver` 用来生成文件的web应用程序
- `npm install -S xlsx` 电子表格格式的解析器
- `npm install -D script-loader` 将js挂在在全局下
#### 表结构
- 渲染页面中的表结构是由`columns`列 和`tableData` 行，来渲染的 `columns`为表头数据`tableData`为表实体内容
```
columns1: [
          {
            title: '序号',
            key: 'serNum'
          },
          {
            title: '选择',
            key: 'choose',
            align: 'center',
            render: (h, params) => {
              if (params.row.status !== '1' && params.row.status !== '2') {
                return h('div', [
                  h('checkbox', {
                    props: {
                      type: 'selection'
                    },
                    on: {
                      'input': (val) => {
                        console.log(val)
                      }
                    }
                  })
                ])
              } else {
                return h('span', {
                  style: {
                    color: '#587dde',
                    cursor: 'pointer'
                  },
                  on: {
                    click: () => {
                      // this.$router.push({name: '', query: { id: params.row.id }})
                    }
                  }
                }, '查看订单')
              }
            }
          },
          ...
        ],
```
tableData 就不写了，具体数据结构查看[iViewAPI](https://www.iviewui.com/components/table)

在build 目录下`webpack.base.conf.js`配置 我们要加载时的路径
```
 alias: {
      'src': path.resolve(__dirname, '../src'),
    }
```
在当前页面中引入依赖
```
  require('script-loader!file-saver')
  // 转二进制用
  require('script-loader!src/vendor/Blob')
  // xlsx核心
  require('script-loader!xlsx/dist/xlsx.core.min')
```

当我们要导出表格执行`@click`事件调用`handleDownload`函数
```
 handleDownload() {
        this.downloadLoading = true
        require.ensure([], () => {
          const {export_json_to_excel} = require('src/vendor/Export2Excel')
          const tHeader = Util.cutValue(this.columns1, 'title')
          const filterVal = Util.cutValue(this.columns1, 'key')
          const list = this.tableData1
          const data = this.formatJson(filterVal, list)
          export_json_to_excel(tHeader, data, '列表excel')
          this.downloadLoading = false
        })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => v[j]))
      }
```
`Util.cutValue `是公共方法，目的是为了将，tHeader 和filterVal 的值转成数组从而生成表格
```
 ### Util module
// 截取value值
utils.cutValue = function (target, name) {
  let arr = []
  for (let i = 0; i < target.length; i++) {
    arr.push(target[i][name])
  }
  return arr
}
```
