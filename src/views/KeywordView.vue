<script setup lang="ts">
import { ref, computed } from "vue";
import { KeywordDataRepository } from "@/data/repository/KeywordDataRepository";
import BaseListSearch from "@/components/BaseListSearch.vue";
import BaseList from "@/components/BaseList.vue";
import KeywordListItem from "@/components/KeywordListItem.vue";
import { marked } from "marked";
import BaseDivider from "@/components/BaseDivider.vue";

const keywordDataRepository = new KeywordDataRepository();
const keywords = keywordDataRepository.findAll();

let filteredKeyword = computed(() =>
  query.value === ""
    ? keywords
    : keywords.filter((keyword) =>
        keyword.keyword.toLowerCase().replace(/\s+/g, "").includes(query.value.toLowerCase().replace(/\s+/g, ""))
      )
);

let query = ref("");
</script>

<template>
  <div class="grid place-items-center w-full">
    <BaseDivider>Keywords</BaseDivider>
    <BaseListSearch
      id="keyword-search"
      @search="query = $event"
      class="z-10 bg-base-300 inset-16 w-full border-0 shadow-xl"
    />
    <BaseList id="keyword-list" class="grid gap-3 w-full">
      <template v-for="keyword in filteredKeyword" :key="keyword.id">
        <KeywordListItem>
          <template #default>
            {{ keyword.keyword }}
          </template>
          <template #description>
            <span v-html="marked.parse(keyword.description)"></span>
          </template>
        </KeywordListItem>
      </template>
    </BaseList>
  </div>
</template>

<style scoped></style>
