<script setup lang="ts">
import { ref, computed } from "vue";
import type { HeroData } from "@/data/repository/HeroData";
import BaseModal from "@/components/BaseModal.vue";
import BaseList from "@/components/BaseList.vue";
import BaseListItem from "@/components/BaseListItem.vue";
import BaseListSearch from "@/components/BaseListSearch.vue";
import { HeroDataRepository } from "@/data/repository/HeroDataRepository";
import { XMarkIcon } from "@heroicons/vue/24/solid";
import { HeroStore } from "@/store/HeroStore";

const props = defineProps<{
  campaignId: string;
}>();

const isOpen = ref(false);

function openModal() {
  isOpen.value = true;
  query.value = "";
}
function closeModal() {
  isOpen.value = false;
}

const heroStore = HeroStore();
const heroes = new HeroDataRepository().findAll();

let filteredHeroes = computed(() => heroes.filter(filterHero));
let query = ref("");

function filterHero(hero: HeroData) {
  if (heroStore.hasInCampaign(hero.id, props.campaignId) == false) {
    return false;
  }
  return hero.name.toLowerCase().replace(/\s+/g, "").includes(query.value.toLowerCase().replace(/\s+/g, ""));
}

function removeHeroFromCampaign(heroId: string) {
  heroStore.removeFromCampaign(heroId, props.campaignId);
  closeModal();
}
</script>

<template>
  <button
    id="campaign-remove-hero"
    class="float-right px-3 py-3 bg-neutral text-gray-200 uppercase font-semibold text-sm rounded-lg"
    @click="openModal"
  >
    Remove hero
  </button>
  <BaseModal :is-open="isOpen" @close-modal="closeModal">
    <template #header>
      <div class="grid grid-cols-2">
        <div class="w-full font-medium place-self-center">Remove hero</div>
        <div>
          <button
            id="close-modal"
            class="px-2 py-2 bg-neutral text-gray-200 uppercase font-semibold text-sm rounded-lg float-right"
            @click="closeModal"
          >
            <XMarkIcon class="h-5 bg-neutral text-gray-200 uppercase font-semibold text-sm rounded-lg" />
          </button>
        </div>
      </div>
    </template>
    <template #default>
      <BaseListSearch @search="query = $event" />
      <BaseList id="campaign-remove-heroes">
        <template v-for="hero in filteredHeroes" :key="hero.id">
          <BaseListItem :avatar="hero.images.avatar" @click="removeHeroFromCampaign(hero.id)">
            {{ hero.name }}
          </BaseListItem>
        </template>
      </BaseList>
    </template>
  </BaseModal>
</template>

<style scoped></style>
