<!--
  * 传统菜单-递归菜单
  * 
  * @Author:    1024创新实验室-主任：卓大 
  * @Date:      2022-09-06 20:29:12 
  * @Wechat:    zhuda1024 
  * @Email:     lab1024@163.com 
  * @Copyright  1024创新实验室 （ https://1024lab.net ），Since 2012 
-->
<template>
  <a-menu :open-keys="openKeys" v-model:selectedKeys="selectedKeys" class="smart-menu" mode="inline" :theme="theme" @openChange="onOpenChange">
    <template v-for="item in menuTree" :key="item.menuId">
      <template v-if="item.visibleFlag && !item.disabledFlag">
        <template v-if="$lodash.isEmpty(item.children)">
          <a-menu-item :key="item.menuId" @click="turnToPage(item)">
            <template #icon>
              <component :is="$antIcons[item.icon]" />
            </template>
            {{ item.menuName }}
          </a-menu-item>
        </template>
        <template v-else>
          <SubMenu :menu-info="item" :key="item.menuId" @turnToPage="turnToPage" />
        </template>
      </template>
    </template>
  </a-menu>
</template>
<script setup>
  import _ from 'lodash';
  import { computed, ref, watch } from 'vue';
  import { useRoute } from 'vue-router';
  import SubMenu from './sub-menu.vue';
  import { router } from '/@/router/index';
  import { useAppConfigStore } from '/@/store/modules/system/app-config';
  import { useUserStore } from '/@/store/modules/system/user';

  const theme = computed(() => useAppConfigStore().$state.sideMenuTheme);
  const flatPattern = computed(() => useAppConfigStore().$state.flatPattern);

  const props = defineProps({
    collapsed: {
      type: Boolean,
      default: false,
    },
  });

  const menuTree = computed(() => useUserStore().getMenuTree || []);
  const rootSubmenuKeys = computed(()=>menuTree.value.map(item=>item.menuId));


  //展开的菜单
  let currentRoute = useRoute();
  const selectedKeys = ref([]);
  const openKeys = ref([]);

  // 页面跳转
  function turnToPage(menu) {
    useUserStore().deleteKeepAliveIncludes(menu.menuId.toString());
    router.push({ path: menu.path });
  }

  /**
   * SmartAdmin中 router的name 就是 后端存储menu的id
   * 所以此处可以直接监听路由，根据路由更新菜单的选中和展开
   */
  function updateOpenKeysAndSelectKeys() {
    // 更新选中
    selectedKeys.value = [_.toNumber(currentRoute.name)];

    /**
     * 更新展开（1、获取新展开的menu key集合；2、保留原有的openkeys，然后把新展开的与之合并）
     */
    //获取需要展开的menu key集合
    let menuParentIdListMap = useUserStore().getMenuParentIdListMap;
    let parentList = menuParentIdListMap.get(currentRoute.name) || [];

    // 如果是折叠菜单的话，则不需要设置openkey
    if (!props.collapsed) {
      // 使用lodash的union函数，进行 去重合并两个数组
      let needOpenKeys = _.map(parentList, 'name').map(Number);
      openKeys.value = _.union(openKeys.value, needOpenKeys);
    }
  }

  watch(
    currentRoute,
    () => {
      updateOpenKeysAndSelectKeys();
    },
    {
      immediate: true,
    }
  );
  function onOpenChange(openKeysParams){
  if(flatPattern.value){
    return;
  }
  const latestOpenKey = openKeysParams.find(key => openKeys.value.indexOf(key) === -1);
  if (rootSubmenuKeys.value.indexOf(latestOpenKey) === -1) {
    openKeys.value = openKeysParams;
  } else {
    openKeys.value = latestOpenKey ? [latestOpenKey] : [];
  }
};
  defineExpose({
    updateOpenKeysAndSelectKeys,
  });
</script>

<style lang="less" scoped>
  .smart-menu {
    position: relative;
  }
</style>
