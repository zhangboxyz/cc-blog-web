<template>
  <a-tree :checkable="checkable" :treeData="categoryTree" :defaultExpandAll="true" :checkedKeys="categoryIds" @check="onCheck" @select="onSelect">
  </a-tree>
</template>

<script>
import categoryApi from '@/api/category'

export default {
  name: 'CategoryTree',
  model: {
    prop: 'categoryIds',
    event: 'check'
  },
  props: {
    checkable: {
      type: Boolean,
      required: false,
      default: true
    },
    categoryIds: {
      type: Array,
      required: false,
      default: () => []
    },
    categories: {
      type: Array,
      required: false,
      default: () => []
    }
  },
  computed: {
    categoryTree() {
      return categoryApi.concreteTree(this.categories)
    }
  },
  methods: {
    onCheck(checkedKeys, e) {
      this.$log.debug('Chekced keys', checkedKeys)
      this.$log.debug('e', e)
      const categoryIds = e.checkedNodes
          .filter(node => {
            return node.data.props.isLeaf
          })
          .map(node => node.key)

      this.$log.debug('Effectively selected category ids', categoryIds)

      this.$emit('check', categoryIds)
    },
    onSelect(checkedKeys, info) {
      this.$emit("select", checkedKeys)
    }
  }
}
</script>
