<template>
  <div>
    <p class="tip">
      快捷菜单操作<br>
      <span class="red">（注：树形结构默认不支持 insert、remove 相关方法，如果要往子节点插入或删除节点数据，直接操作数据源即可）</span>
    </p>

    <vxe-toolbar custom>
      <template v-slot:buttons>
        <vxe-button @click="getInsertEvent">获取新增</vxe-button>
      </template>
    </vxe-toolbar>

    <vxe-table
      resizable
      keep-source
      ref="xTree"
      :tree-config="treeConfig"
      :context-menu="{header: {options: headerMenus}, body: {options: bodyMenus}, visibleMethod}"
      :edit-config="{trigger: 'click', mode: 'row', showStatus: true}"
      :data="tableData"
      @context-menu-click="contextMenuClickEvent">
      <vxe-table-column type="checkbox" width="120" tree-node></vxe-table-column>
      <vxe-table-column field="name" title="Name" :edit-render="{name: 'input'}"></vxe-table-column>
      <vxe-table-column field="size" title="Size" :edit-render="{name: 'input'}"></vxe-table-column>
      <vxe-table-column field="type" title="Type" :edit-render="{name: 'input'}"></vxe-table-column>
      <vxe-table-column field="date" title="Date" :edit-render="{name: '$input', props: {type: 'date'}}"></vxe-table-column>
    </vxe-table>

    <pre>
      <code>
        | Arrow Up ↑ | 移动到上一个菜单选项 |
        | Arrow Down ↓ | 移动到下一个菜单选项 |
        | Arrow Left ← | 关闭二级菜单 |
        | Arrow Right → | 打开二级菜单 |
        | Esc | 关闭菜单选项 |
        | Enter | 选中当前菜单选项 |
        | Spacebar | 选中当前菜单选项 |
      </code>
    </pre>

    <p class="demo-code">{{ $t('app.body.button.showCode') }}</p>

    <pre>
      <code class="xml">{{ demoCodes[0] }}</code>
      <code class="javascript">{{ demoCodes[1] }}</code>
    </pre>
  </div>
</template>

<script>
import XEUtils from 'xe-utils'
import hljs from 'highlight.js'

export default {
  data () {
    return {
      tableData: [],
      treeConfig: {
        children: 'children'
      },
      headerMenus: [
        [
          { code: 'hideColumn', name: '隐藏列', disabled: false },
          { code: 'showAllColumn', name: '取消所有隐藏列', disabled: false }
        ]
      ],
      bodyMenus: [
        [
          { code: 'insertAt', name: '插入一行', disabled: false },
          { code: 'expand', name: '展开节点', disabled: false },
          { code: 'contract', name: '收缩节点', disabled: false }
        ]
      ],
      demoCodes: [
        `
        <vxe-toolbar custom>
          <template v-slot:buttons>
            <vxe-button @click="getInsertEvent">获取新增</vxe-button>
          </template>
        </vxe-toolbar>

        <vxe-table
          resizable
          keep-source
          ref="xTree"
          :tree-config="treeConfig"
          :context-menu="{header: {options: headerMenus}, body: {options: bodyMenus}, visibleMethod}"
          :edit-config="{trigger: 'click', mode: 'row', showStatus: true}"
          :data="tableData"
          @context-menu-click="contextMenuClickEvent">
          <vxe-table-column type="checkbox" width="120" tree-node></vxe-table-column>
          <vxe-table-column field="name" title="Name" :edit-render="{name: 'input'}"></vxe-table-column>
          <vxe-table-column field="size" title="Size" :edit-render="{name: 'input'}"></vxe-table-column>
          <vxe-table-column field="type" title="Type" :edit-render="{name: 'input'}"></vxe-table-column>
          <vxe-table-column field="date" title="Date" :edit-render="{name: '$input', props: {type: 'date'}}"></vxe-table-column>
        </vxe-table>
        `,
        `
        export default {
          data () {
            return {
              tableData: [],
              treeConfig: {
                children: 'children'
              },
              headerMenus: [
                [
                  { code: 'hideColumn', name: '隐藏列', disabled: false },
                  { code: 'showAllColumn', name: '取消所有隐藏列', disabled: false }
                ]
              ],
              bodyMenus: [
                [
                  { code: 'insertAt', name: '插入一行', disabled: false },
                  { code: 'expand', name: '展开节点', disabled: false },
                  { code: 'contract', name: '收缩节点', disabled: false }
                ]
              ]
            }
          },
          created () {
            this.tableData = window.MOCK_TREE_DATA_LIST
          },
          methods: {
            insertAtEvent (row, column) {
              let xTree = this.$refs.xTree
              xTree.createRow({
                name: '新数据',
                date: XEUtils.toDateString(new Date(), 'yyyy-MM-dd'),
                isNew: true
              }).then(newRow => {
                // 插入到指定节点位置中
                let rowNode = XEUtils.findTree(this.tableData, item => item === row, this.treeConfig)
                if (rowNode) {
                  rowNode.items.splice(rowNode.index, 0, newRow)
                  xTree.syncData().then(() => xTree.setActiveCell(newRow, column.property))
                }
              })
            },
            getInsertEvent () {
              let insertRecords = XEUtils.filterTree(this.tableData, item => item.isNew, this.treeConfig)
              this.$XModal.alert(insertRecords.length)
            },
            visibleMethod  ({ row, type }) {
              let xTree = this.$refs.xTree
              let treeConfig = this.treeConfig
              if (type === 'body') {
                this.bodyMenus.forEach(list => {
                  list.forEach(item => {
                    if (['expand', 'contract'].includes(item.code)) {
                      if (row[treeConfig.children] && row[treeConfig.children].length) {
                        let isExpand = xTree.isTreeExpandByRow(row)
                        item.disabled = ['expand'].includes(item.code) ? isExpand : !isExpand
                      } else {
                        item.disabled = true
                      }
                    }
                  })
                })
              }
              return true
            },
            contextMenuClickEvent ({ menu, row, column }) {
              let xTree = this.$refs.xTree
              switch (menu.code) {
                case 'hideColumn':
                  xTree.hideColumn(column)
                  break
                case 'showAllColumn':
                  xTree.resetColumn()
                  break
                case 'insertAt':
                  this.insertAtEvent(row, column)
                  break
                case 'expandOrFold':
                  xTree.toggleTreeExpand(row)
                  break
              }
            }
          }
        }
        `
      ]
    }
  },
  created () {
    this.tableData = XEUtils.clone(window.MOCK_TREE_DATA_LIST, true)
  },
  mounted () {
    Array.from(this.$el.querySelectorAll('pre code')).forEach((block) => {
      hljs.highlightBlock(block)
    })
  },
  methods: {
    insertAtEvent (row, column) {
      const xTree = this.$refs.xTree
      xTree.createRow({
        name: '新数据',
        date: XEUtils.toDateString(new Date(), 'yyyy-MM-dd'),
        isNew: true
      }).then(newRow => {
        // 插入到指定节点位置中
        const rowNode = XEUtils.findTree(this.tableData, item => item === row, this.treeConfig)
        if (rowNode) {
          rowNode.items.splice(rowNode.index, 0, newRow)
          xTree.syncData().then(() => xTree.setActiveCell(newRow, column.property))
        }
      })
    },
    getInsertEvent () {
      const insertRecords = XEUtils.filterTree(this.tableData, item => item.isNew, this.treeConfig)
      this.$XModal.alert(insertRecords.length)
    },
    visibleMethod  ({ row, type }) {
      const xTree = this.$refs.xTree
      const treeConfig = this.treeConfig
      if (type === 'body') {
        this.bodyMenus.forEach(list => {
          list.forEach(item => {
            if (['expand', 'contract'].includes(item.code)) {
              if (row[treeConfig.children] && row[treeConfig.children].length) {
                const isExpand = xTree.isTreeExpandByRow(row)
                item.disabled = ['expand'].includes(item.code) ? isExpand : !isExpand
              } else {
                item.disabled = true
              }
            }
          })
        })
      }
      return true
    },
    contextMenuClickEvent ({ menu, row, column }) {
      const xTree = this.$refs.xTree
      switch (menu.code) {
        case 'hideColumn':
          xTree.hideColumn(column)
          break
        case 'showAllColumn':
          xTree.resetColumn()
          break
        case 'insertAt':
          this.insertAtEvent(row, column)
          break
        case 'expand':
          xTree.setTreeExpand(row, true)
          break
        case 'contract':
          xTree.setTreeExpand(row, false)
          break
      }
    }
  }
}
</script>
