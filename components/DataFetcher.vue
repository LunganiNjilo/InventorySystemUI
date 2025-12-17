<template>
  <div>
    <slot
      :data="data"
      :create="createItem"
      :edit="editItem"
      :deleteItem="deleteItem"
      :refresh="fetchData"
    />
  </div>
</template>

<script setup>
defineOptions({ name: "DataFetcher" });

import { ref, onMounted, watch } from "vue";

/* =========================
   PROPS
========================= */
const props = defineProps({
  uri: {
    type: String,
    required: true
  }
});

/* =========================
   STATE
========================= */
const data = ref({
  result: [],
  totalRecordCount: 0,
  totalPages: 1,
  pageNumberMessage: "",
  isPrevious: false,
  isNext: false
});

function normalizeResponse(response) {
  if (!response) {
    return {
      result: [],
      totalRecordCount: 0,
      totalPages: 1,
      pageNumberMessage: "",
      isPrevious: false,
      isNext: false
    };
  }

  const rawResult = Array.isArray(response)
    ? response
    : response.result ?? response.data ?? [];

  return {
    result: [...rawResult],
    totalRecordCount: response.totalRecordCount ?? rawResult.length,
    totalPages: response.totalPages ?? 1,
    pageNumberMessage: response.pageNumberMessage ?? "",
    isPrevious: !!response.isPrevious,
    isNext: !!response.isNext
  };
}

function authHeaders() {
  const token = localStorage.getItem("token");
  return token ? { Authorization: `Bearer ${token}` } : {};
}

async function fetchData() {
  if (!props.uri) return;

  try {
    const fetched = await $fetch(props.uri, {
      headers: authHeaders()
    });
    data.value = normalizeResponse(fetched);
  } catch (err) {
    console.error("[DataFetcher] fetch error:", err);
    data.value = normalizeResponse(null);
  }
}

async function createItem(payload) {
  return await $fetch(props.uri, {
    method: "POST",
    headers: authHeaders(),
    body: payload
  });
}

async function editItem(id, payload) {
  return await $fetch(`${props.uri}/${id}`, {
    method: "PUT",
    headers: authHeaders(),
    body: payload
  });
}

async function deleteItem(id) {
  return await $fetch(`${props.uri}/${id}`, {
    method: "DELETE",
    headers: authHeaders()
  });
}

onMounted(fetchData);
watch(() => props.uri, fetchData);
</script>
