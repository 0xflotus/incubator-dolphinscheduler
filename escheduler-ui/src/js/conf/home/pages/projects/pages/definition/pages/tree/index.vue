<template>
  <m-list-construction :title="$t('TreeView')">
    <template slot="conditions"></template>
    <template slot="content">
      <div class="tree-view-index-model">
        <div class="tree-limit-select">
          <x-select v-model="limit" style="width: 70px;" @on-change="_onChangeSelect">
            <x-option
                    v-for="city in [{value:25},{value:50},{value:75},{value:100}]"
                    :key="city.value"
                    :value="city.value"
                    :label="city.value">
            </x-option>
          </x-select>
          <x-button
                  @click="_rtTasksDag"
                  v-if="$route.query.subProcessIds"
                  type="primary"
                  size="default"
                  icon="fa fa-reply">
            {{$t('Return_1')}}
          </x-button>
        </div>
        <div class="tasks-color">
          <div class="toolbar-color-sp">
            <a href="javascript:">
              <span>Node Type</span>
            </a>
            <a href="javascript:" v-for="(k,v) in tasksType">
              <i class="fa fa-circle" :style="{color:k.color}"></i>
              <span>{{v}}</span>
            </a>
          </div>
          <div class="state-tasks-color-sp">
            <a href="javascript:">
              <span>{{$t('Task Status')}}</span>
            </a>
            <a href="javascript:" v-for="(item) in tasksState">
              <i class="fa fa-square" :style="{color:item.color}"></i>
              <span>{{item.desc}}</span>
            </a>
          </div>
        </div>
        <div class="tree-model" v-show="!isNodata">
          <div class="d3-tree">
            <svg class='tree' width="100%"></svg>
          </div>
        </div>
        <m-no-data v-if="isNodata"></m-no-data>
      </div>
      <m-spin :is-spin="isLoading"></m-spin>
    </template>
  </m-list-construction>

</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import Tree from './_source/tree'
  import { uuid } from '@/module/util'
  import mSpin from '@/module/components/spin/spin'
  import mNoData from '@/module/components/noData/noData'
  import { tasksType, tasksState } from '@/conf/home/pages/dag/_source/config'
  import mSecondaryMenu from '@/module/components/secondaryMenu/secondaryMenu'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'tree-view-index-index',
    data () {
      return {
        // limit
        limit: 25,
        // loading
        isLoading: true,
        // node type
        tasksType: tasksType,
        // node state
        tasksState: tasksState,
        // tree data
        treeData: {},
        // is data
        isNodata: false
      }
    },
    props: {},
    methods: {
      ...mapActions('dag', ['getViewTree']),
      /**
       * get tree data
       */
      _getViewTree () {
        this.isLoading = true

        Tree.reset()

        this.getViewTree({
          processId: this.$route.params.id,
          limit: this.limit
        }).then(res => {
          let data = _.cloneDeep(res)
          this.treeData = data
          if (!this.treeData.children) {
            this.isLoading = false
            this.isNodata = true
            return
          }
          let recursiveChildren = (children) => {
            if (children.length) {
              _.map(children, v => {
                v.uuid = `${uuid('uuid_')}${uuid() + uuid()}`
                if (v.children.length) {
                  recursiveChildren(v.children)
                }
              })
            }
          }
          recursiveChildren(data.children)
          // init tree
          Tree.init({
            data: _.cloneDeep(data),
            limit: this.limit,
            selfTree: this
          }).then(() => {
            setTimeout(() => {
              // this.isLoading = false
            }, 100)
          })
        }).catch(e => {
          this.isLoading = false
          if (!e.data) {
            this.isNodata = true
          }
        })
      },

      /**
       * Return to the previous child node
       */
      _rtTasksDag () {
        let getIds = this.$route.query.subProcessIds
        let idsArr = getIds.split(',')
        let ids = idsArr.slice(0, idsArr.length - 1)
        let id = idsArr[idsArr.length - 1]
        let query = {}

        if (id !== idsArr[0]) {
          query = { subProcessIds: ids.join(',') }
        }
        this.$router.push({ path: `/projects/definition/tree/${id}`, query: query })
      },
      /**
       * Subprocess processing
       * @param subProcessId 子流程Id
       */
      _subProcessHandle (subProcessId) {
        let subProcessIds = []
        let getIds = this.$route.query.subProcessIds
        if (getIds) {
          let newId = getIds.split(',')
          newId.push(this.$route.params.id)
          subProcessIds = newId
        } else {
          subProcessIds.push(this.$route.params.id)
        }
        this.$router.push({ path: `/projects/definition/tree/${subProcessId}`, query: { subProcessIds: subProcessIds.join(',') } })
      },
      _onChangeSelect (o) {
        this.limit = o.value
        this._getViewTree()
      }
    },
    watch: {
      '$route.params.id' () {
        this._getViewTree()
      }
    },
    created () {
      this._getViewTree()
    },
    mounted () {
    },
    components: { mSpin, mSecondaryMenu, mListConstruction, mNoData }
  }
</script>

<style lang="scss" rel="stylesheet/scss">

  .tree-view-index-model {
    background: url('img/dag_bg.png');
    position: relative;
    .tree-limit-select {
      position: absolute;
      right: 20px;
      top: 22px;
      z-index: 1;
    }
    .tasks-color {
      min-height: 76px;
      background: #fff;
      padding-left: 20px;
      position: relative;
      padding-bottom: 10px;
      .toolbar-color-sp {
        padding: 12px 0;
      }

    }
    .tree-model {
      width: calc(100%);
      height: calc(100vh - 224px);
      overflow-x: scroll;
    }
    .d3-tree {
      padding-left: 30px;
      .node {
        text {
          font: 11px sans-serif;
          pointer-events: none;
        }
      }
      rect {
        cursor: pointer;
        &.state {
          stroke: #666;
          shape-rendering: crispEdges;
        }
      }
      path {
        &.link{
          fill: none;
          stroke: #666;
          stroke-width: 2px;
        }
      }
      circle {
        stroke: #666;
        fill: #0097e0;
        stroke-width: 1.5px;
        cursor: pointer;
      }
    }
  }


</style>
