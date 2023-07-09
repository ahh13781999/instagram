<script setup>
import { ref } from 'vue';
import { supabase } from "../supabass";
import { useUserStore } from "../stores/users";
import { storeToRefs } from 'pinia';

const visible = ref(false);
const caption = ref("");
const file = ref(null);
const loading = ref(false)
const errorMessage = ref("")

const userStore = useUserStore()
const { user } = storeToRefs(userStore);

const props = defineProps(['addNewPost'])

const showModal = () => {
  visible.value = true;
};

const handleOk = async () => {
  loading.value = true;
  const fileName = Math.floor(Math.random() * 1_000_000_000_000)
  let filePath
  if (file.value) {
    const { data, error } = await supabase.storage.from("images").upload("public/" + fileName, file.value)

    if (error) {
      loading.value = false
      return errorMessage.value = "Unable to upload image"
    }
    filePath = data.path
    await supabase.from("posts").insert({
      url: filePath,
      caption: caption.value,
      owner_id: user.value.id
    })

  }  
  
  props.addNewPost({
    url: filePath,
    caption: caption.value,
  })

  loading.value = false;
  visible.value = false;
  caption.value = "";
};

const hanleUploadChange = (e) => {
  if (e.target.files[0]) {
    file.value = e.target.files[0]
  }
};
</script>

<template>
  <div>
    <a-button @click="showModal">Upload Photo</a-button>
    <a-modal v-model:visible="visible" title="Upload Photo" @ok="handleOk">
      <div v-if="!loading">
        <input type="file" accept=".jpeg,.png,.jpg" @change="hanleUploadChange">
        <a-input v-model:value="caption" placeholder="Caption..." :maxLength="50" />
      </div>
      <div class="spinner" v-else>
        <a-spin />
      </div>
      <a-typography-text v-if="errorMessage" type="danger">{{ errorMessage }}</a-typography-text>
    </a-modal>
  </div>
</template>

<style scoped>
input {
  margin-top: 10px;
}

.spinner {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>