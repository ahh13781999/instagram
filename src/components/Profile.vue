<script setup>
import Container from './Container.vue';
import UserBar from "./UserBar.vue";
import ImageGallery from "./ImageGallery.vue";
import { ref, onMounted, watch, reactive } from 'vue';
import { supabase } from '../supabass';
import { useRoute } from 'vue-router';
import { useUserStore } from "../stores/users";
import { storeToRefs } from 'pinia';

const userStore = useUserStore();
const { user: loggedInUser } = storeToRefs(userStore)

const route = useRoute();
const { username } = route.params;

const posts = ref([]);
const user = ref(null);
const loading = ref(false)
const isFollowing = ref(false);

const userInfo = reactive({
  posts: null,
  followers: null,
  followings: null,
})

const addNewPost = (post) => {
  posts.value.unshift(post)
};

const updateIsFollowing = (follow) => {
  isFollowing.value = follow
}

const fetchData = async () => {
  loading.value = true;
  const { data: userData } = await supabase
    .from("users")
    .select()
    .eq("username", username)
    .single();

  if (!userData) {
    loading.value = false;
    return user.value = null
  }

  user.value = userData;

  const { data: postData } = await supabase
    .from("posts")
    .select()
    .eq("owner_id", user.value.id)

  posts.value = postData;

  await fetchIsFollowing()
  const followerCount = await fetchFollowersCount()
  const followingCount = await fetchFollowingCount()

  userInfo.followers = followerCount;
  userInfo.followings = followingCount;
  userInfo.posts = posts.value.length

  loading.value = false;
};

const fetchIsFollowing = async () => {
  if (loggedInUser.value && (loggedInUser.value.id !== user.value.id)) {
    const { data } = await supabase
      .from("followers_following")
      .select()
      .eq("follower_id", loggedInUser.value.id)
      .eq("following_id", user.value.id)
      .single()

    if (data) return isFollowing.value = true
  }
}

const fetchFollowersCount = async () => {
  const { count } = await supabase
    .from("followers_following")
    .select('*', { count: "exact" })
    .eq("following_id", user.value.id);

  return count;
}

const fetchFollowingCount = async () => {
  const { count } = await supabase
    .from("followers_following")
    .select("*", { count: "exact" })
    .eq("follower_id", user.value.id)

  return count
}

watch(loggedInUser, () => {
  fetchIsFollowing()
})

onMounted(() => {
  fetchData()
});
</script>

<template>
  <Container>
    <div class="profile-container" v-if="!loading">
      <UserBar :key="$route.params.username" :user="user" :userInfo="userInfo" :addNewPost="addNewPost"
        :isFollowing="isFollowing" :updateIsFollowing="updateIsFollowing" />
      <ImageGallery :posts="posts" />
    </div>
    <div class="spinner" v-else>
      <a-spin />
    </div>
  </Container>
</template>

<style scoped>
.profile-container {
  display: flex;
  flex-direction: column;
  padding: 20px 0;
}

.spinner {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 85vh;

}
</style>