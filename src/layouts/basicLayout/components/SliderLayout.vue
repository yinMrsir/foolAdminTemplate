<template>
  <a-layout-sider class="layout-sider" :collapsed="collapsed" :trigger="null" collapsible>
    <div :class="['logo', collapsed && 'mini']" style="cursor: pointer" @click="$router.push({ path: '/' })" />
    <a-menu
        v-model:openKeys="openKeys"
        v-model:selectedKeys="selectedKeys"
        mode="inline"
        theme="dark"
        @click="onChangeMenu">
      <template v-for="(item, index) in sortMenu($router.options.routes)">
        <!--  如果是允许显示的菜单  -->
        <template v-if="!item.hidden">
          <!-- 存在子菜单 -->
          <template v-if="item.children && !item.hideChildrenInMenu">
            <a-sub-menu :key="'sub-' + index" v-if="item.meta">
              <template #title>
                <span>
                  <component :is="$antIcons[item.meta.icon]" />
                  <span>{{ item.meta.title }}</span>
                </span>
              </template>
              <!-- 遍历二级菜单显示 -->
              <template v-for="(childrenItem, childrenIndex) in item.children">
                <template v-if="!childrenItem.hidden">
                  <a-menu-item :key="index + '-' + childrenIndex">
                    <router-link :to="{ name: childrenItem.name }">
                      {{ childrenItem.meta.title }}
                    </router-link>
                  </a-menu-item>
                </template>
              </template>
            </a-sub-menu>
          </template>
          <!-- 没有子菜单 -->
          <template v-else-if="item.meta">
            <a-menu-item :key="index + ''">
              <router-link :to="{ name: item.name }">
                <component :is="$antIcons[item.meta.icon]" />
                <span class="menu-item-a">
                  {{ item.meta.title }}
                </span>
              </router-link>
            </a-menu-item>
          </template>
        </template>
      </template>
    </a-menu>
  </a-layout-sider>
</template>

<script>
import { computed, ref, onMounted } from 'vue'
import { useStore } from 'vuex'
import { useRouter, onBeforeRouteUpdate } from "vue-router";

export default {
  setup() {
    const store = useStore();
    const collapsed = computed(() => store.state.basicLayout.collapsed);

    const selectedKeys = ref([])
    const openKeys = ref([])

    const router = useRouter();
    /**
     * 通过当前路由地址设置选中状态
     * @private
     */
    const setSelectStatus = () => {
      const currRoute = router.currentRoute.value
      let name = currRoute.meta.parentName || currRoute.name
      const routes = router.options.routes
      for (let [index, value] of routes.entries()) {
        if (value.children) {
          for (let [childrenIndex, childrenValue] of value.children.entries()) {
            if (childrenValue.name === name) {
              openKeys.value = [`sub-${index}`]
              if (!value.hideChildrenInMenu) {
                selectedKeys.value = [`${index}-${childrenIndex}`]
              } else {
                selectedKeys.value = [`${index}`]
              }
              break
            }
          }
        } else if (value.meta) {
          if (value.name === name) {
            selectedKeys.value = [`${index}`]
            break
          }
        }
      }
    }

    /**
     * 通过点击menu获取到key设置选中状态
     * @param key
     * @private
     */
    const onChangeMenu = ({ key }) => {
      selectedKeys.value = [`${key}`]
    }

    /**
     * 屏幕尺寸变化时
     */
    const onResize = () => {
      if ( window.innerWidth < 1200) {
        store.commit('basicLayout/SET_COLLAPSED_STATUS', true)
      }
    }

    onMounted(() => {
      setSelectStatus()
      window.addEventListener('resize', onResize);
    })

    onBeforeRouteUpdate((to) => {
      setSelectStatus()
    });

    /**
     * router排序
     * @param menuModules
     * @return {void | this }
     */
    const sortMenu = (menuModules) => {
      return menuModules.sort((a, b) => {
        return (a.orderNo || 0) - (b.orderNo || 0);
      });
    }

    return {
      collapsed,
      selectedKeys,
      openKeys,
      onChangeMenu,
      sortMenu
    }
  }
}
</script>

<style lang="less" scoped>
@import '../../../assets/less/var.less';

.layout-sider {
  .logo {
    height: 42px;
    margin: @margin;
    background: url('../../../assets/images/logo.png') no-repeat center center;
    background-size: auto 100%;

    &.mini {
      background: url('../../../assets/images/logo-mini.png') no-repeat center center;
      background-size: 80% auto;
    }
  }

  .menu-item-a {
    a {
      color: #fff;
    }
  }
}
</style>
