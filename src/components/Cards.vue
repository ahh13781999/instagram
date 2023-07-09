<script setup>
import { supabase } from '../supabass';
import Card from './Card.vue';
import Observer from "./Observer.vue";
import { useUserStore } from "../stores/users";
import { storeToRefs } from 'pinia';
import { ref, onMounted } from 'vue'

const userStore = useUserStore()
const { user } = storeToRefs()

const posts = ref([])
const lastCardIndex = ref(2)
const ownerIds = ref([])
const reachEnd = ref(false);

const fetchData = async () => {
  const { data: followings } = await supabase
    .from("followers_following")
    .select("following_id")
    .eq("follower_id", user.value.id)

  ownerIds.value = followings.map(f => f.following_id)

  const { data } = await supabase
    .from("posts")
    .select()
    .in("owner_id", ownerIds.value)
    .range(0, lastCardIndex.value)
    .order("created_at", { ascending: false })

  posts.value = data
};

const fetchNextSet = async () => {
  if (!reachEnd.value) {
    const { data } = await supabase
      .from("posts")
      .select()
      .in("owner_id", ownerIds.value)
      .range(lastCardIndex.value + 1, lastCardIndex.value + 3)
      .order("created_at", { ascending: false })

    posts.value = [...posts.value, ...data]

    lastCardIndex.value = lastCardIndex.value + 3;

    if (!data.length) {
      reachEnd.value = true;
    }
  }
}

onMounted(() => {
  fetchData()
});
</script>

<template>
  <div class="timeline-container">
    <Card v-for="post in posts" :key="post.id" :post="post" />
    <Observer v-if="posts.length" @intersect="fetchNextSet" />
  </div>
</template>