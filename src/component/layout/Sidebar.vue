<template>
  <aside class="menu">
    <ul class="menu-list">
      <li v-for="item in menu" class="menu-level1" :class="{'has-chlidren': item.children && item.children.length, 'expand': !item.meta.expanded }">
        <div @click="toggle(item)">
          <template v-if="!item.redirect">
            <router-link :to="item.path" :class="{'is-active': isActive(item.path)}">{{toName(item)}}</router-link>
          </template>
          <template v-else>
            <a href="javascript:void(0)">{{toName(item)}}</a>
          </template>
        </div>
        <expanding  v-if="item.children && item.children.length">
          <ul v-show="item.children && item.meta.expanded" class="menu-level2">
            <li v-for="subItem in inMenuItems(item.children)">
              <router-link :to="item.path + '/'+ subItem.path">{{toName(subItem)}}</router-link>
            </li>
          </ul>
        </expanding>
      </li>
    </ul>
  </aside>
</template>

<script>
import menu from 'menu'
import Expanding from 'component/Expanding.vue'
import { mapGetters } from 'vuex'

export default {
  data() {
    return {
      menu,
      subMenu: []
    }
  },
  computed: {
    ...mapGetters(['currLan']),
  },
  components: {
    Expanding
  },
  methods: {
    expandMatchItem (route) {
      let matched = route.matched
      let lastMatched = matched[matched.length - 1]
      let parent = lastMatched.parent || lastMatched

      if (parent === lastMatched) {
        const p = this.findParentFromMenu(route)
        if (p) {
          parent = p
        }
      }

      if ('expanded' in parent.meta && parent !== lastMatched) {
        parent.meta.expanded = true
      }
    },
    toggle (item) {
      item.meta.expanded = !item.meta.expanded
    },
    findParentFromMenu (route) {
      for (let i = 0, l = menu.length; i < l; i++) {
        const item = menu[i]
        const k = item.children && item.children.length
        if (k) {
          for (let j = 0; j < k; j++) {
            if (item.children[j].name === route.name) {
              return item
            }
          }
        }
      }
    },
    isActive(path) {
      let matched = this.$route.matched
      let lastMatched = matched[matched.length - 1]
      return lastMatched && path === lastMatched.path
    },
    inMenuItems(items) {
      return items.filter(item => {
        var res = true
        if(item.meta && item.meta.inMenu === false) {
          res = false
        }
        return res
      })
    },
    toName(item) {
      if(item.meta && item.meta.showName) {
        return this.$options.filters.toI18nName(item.meta.showName, this.currLan)
      }
      return item.name
    }
  },
  mounted () {
    let route = this.$route
    if (route.name) {
      this.expandMatchItem(route)
    }
  },
  watch: {
    $route (route) {
      this.expandMatchItem(route)
    }
  }
}
</script>

<style scoped lang="sass">
  .menu{
    position: fixed;
    z-index: 1;
    left: 0;
    top: 52px;
    bottom: 0;
    width: 16.66%;
    border-right: 1px solid #e7e7e7;
    background-color: #f8f8f8;
    padding-top: 20px;
    ul{
      margin: 0;
      list-style: none;
    }
  }
  .menu-list{
    padding-left: 0;
    ul{
      padding-left: 0;
    }
    a{
      display: block;
      line-height: 2;
      color: #333;
      text-decoration: none;
      font-size: 14px;
    }
  }
  .menu-level1{
    a{
      padding-left: 20px;
    }
    & .is-active, .menu-level2 .router-link-active{
      background-color: #00d1b2;
      color: #fff !important;
    }
    &.has-chlidren{
      position: relative;
      &:after{
        content: '\e080';
        font-family: 'Glyphicons Halflings';
        position: absolute;
        top: 0;
        right: 10px;
        transform: rotate(90deg);
        transition: transform .5s ease-in;
      }
      .menu-level2{
        padding-left: 40px;
        a{
            padding-left: 10px;
            margin-right: 10px;
          }
      }
      &.expand{
        &:after{
          transform: rotate(0);
        }
      }
    }

  }


</style>
