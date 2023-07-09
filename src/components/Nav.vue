<script setup>
import { RouterLink, useRouter } from "vue-router";
import Container from "./Container.vue";
import AuthModal from "./AuthModal.vue";
import { ref } from "vue";
import { storeToRefs } from "pinia";
import { useUserStore } from "../stores/users";

const userStore = useUserStore()

const { user, loadingUser } = storeToRefs(userStore)
const searchUsername = ref("");

const router = useRouter();

const onSearch = () => {
  if (searchUsername.value) {
    router.push(`/profile/${searchUsername.value}`);
    searchUsername.value = ""
  }
};

const handleLogout = async () => {
  await userStore.handleLogout()
};

const goToUserProfile = () => {
  router.push(`/profile/${user.value.username}`)
};
</script>

<template>
  <a-layout-header>
    <Container>
      <div class="nav-container">
        <div class="right-content">
          <RouterLink to="/">Instagram</RouterLink>
          <a-input-search v-model:value="searchUsername" placeholder="username..." style="width: 200px"
            @search="onSearch" />
        </div>
        <div class="content" v-if="!loadingUser">
          <div class="left-content" v-if="!user">
            <AuthModal :isLogin="false" />
            <AuthModal :isLogin="true" />
          </div>
          <div class="left-content" v-else>
            <a-button type="primary" @click="goToUserProfile">Profile</a-button>
            <a-button type="primary" @click="handleLogout">Logout</a-button>
          </div>
        </div>
      </div>
    </Container>
  </a-layout-header>
</template>

<style scoped>
.nav-container {
  display: flex;
  justify-content: space-between;
}

.content {
  display: flex;
  align-items: center;
}

.right-content {
  display: flex;
  align-items: center;
}

.right-content a {
  margin-right: 10px;
}

.left-content {
  display: flex;
  align-items: center;
}

.left-content button {
  margin-left: 10px;
}
</style>