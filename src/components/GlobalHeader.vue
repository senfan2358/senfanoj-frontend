<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="auto">
      <a-menu
        mode="horizontal"
        :selected-keys="selectedKeys"
        @menu-item-click="doMenuClick"
      >
        <a-menu-item
          key="0"
          :style="{ padding: 0, marginRight: '38px' }"
          disabled
        >
          <div class="title-bar">
            <img class="logo" src="../assets/images.png" />
            <div class="title">OJ</div>
          </div>
        </a-menu-item>
        <a-menu-item v-for="item in visibleRoutes" :key="item.path">
          {{ item.name }}
        </a-menu-item>
      </a-menu>
    </a-col>
    <a-col flex="100px">
      <a-space size="large">
        <a-avatar :imageUrl="store.state.user.loginUser.userAvatar"> </a-avatar>
        <a-dropdown>
          <a-button @click="login">
            {{ store.state.user?.loginUser?.userName ?? "登录" }}
          </a-button>
          <template #content>
            <a-doption
              :disabled="store.state.user?.loginUser?.userRole === 'notLogin'"
              @click="toCenter"
              >个人中心
            </a-doption>
            <a-doption
              @click="logout"
              :disabled="store.state.user?.loginUser?.userRole === 'notLogin'"
              >注销
            </a-doption>
          </template>
        </a-dropdown>
      </a-space>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "../router/routes";
import { useRoute, useRouter } from "vue-router";
import { computed, ref } from "vue";
import { useStore } from "vuex";
import checkAccess from "@/access/checkAccess";
import ACCESS_ENUM from "@/access/accessEnum";
import { UserControllerService } from "../../generated";

const router = useRouter();
const store = useStore();

// 展示在菜单的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item, index) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // 根据权限过滤菜单
    if (
      !checkAccess(store.state.user.loginUser, item?.meta?.access as string)
    ) {
      return false;
    }
    return true;
  });
});

// 默认主页
const selectedKeys = ref(["/"]);

// 路由跳转后，更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});
const login = async () => {
  console.log(store.state.user);
  if (store.state.user?.loginUser?.userRole == "notLogin") {
    router.push({
      path: "/user/login",
      replace: true,
    });
  }
};
const logout = async () => {
  const res = await UserControllerService.userLogoutUsingPost();
  store.commit("removeUserRole");
  location.reload();
};
setTimeout(() => {
  store.dispatch("user/getLoginUser", {
    userName: "管理员",
    userRole: ACCESS_ENUM.ADMIN,
  });
}, 3000);

const doMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};
const toCenter = () => {
  router.push({
    path: "/user/center",
  });
};
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: #444;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
