<script setup lang="ts">
import type { ItemData } from "@/data/repository/ItemData";
import { HeroStore } from "@/store/HeroStore";
import type { CoreItemDataRepository } from "@/data/repository/campaign/core/CoreItemDataRepository";
import ItemCardSelectCategorized from "@/components/ItemCardSelectCategorized.vue";

const heroStore = HeroStore();

const emit = defineEmits(["stash"]);
const props = defineProps<{
  heroId: string;
  campaignId: string;
  cardsDataRepository: CoreItemDataRepository;
  bagSlot: number;
}>();

const hero = heroStore.findInCampaign(props.heroId, props.campaignId);

const slotItem = props.bagSlot === 1 ? hero.equipment.bagOneId : hero.equipment.bagTwoId;
const itemId = slotItem ?? "";

const itemCards: ItemData[] = props.cardsDataRepository.findAll();
const itemCardCategories = [
  { name: "Treasure Deck", items: itemCards.filter((item) => item.cardType === "Chest") },
  { name: "Misc", items: itemCards.filter((item) => item.cardType !== "Chest") },
];

// eslint-disable-next-line @typescript-eslint/no-unused-vars
function subTypeList(item: ItemData) {
  return "";
}

function onClear() {
  if (props.bagSlot === 1) {
    hero.equipment.bagOneId = "";
    return;
  }
  hero.equipment.bagTwoId = "";
}

function onSelect(selectedId: string) {
  if (props.bagSlot === 1) {
    heroStore.findInCampaign(props.heroId, props.campaignId).equipment.bagOneId = selectedId;
    return;
  }
  heroStore.findInCampaign(props.heroId, props.campaignId).equipment.bagTwoId = selectedId;
}

function onStash() {
  const heroState = heroStore.findInCampaign(props.heroId, props.campaignId);
  if (props.bagSlot === 1) {
    if (heroState.equipment.bagOneId === "") return;
    heroStore.findInCampaign(props.heroId, props.campaignId).stashedCardIds.push(heroState.equipment.bagOneId!);
    emit("stash");
    return;
  }

  if (heroState.equipment.bagTwoId === "") return;
  heroStore.findInCampaign(props.heroId, props.campaignId).stashedCardIds.push(heroState.equipment.bagTwoId!);
  emit("stash");
}
</script>

<template>
  <ItemCardSelectCategorized
    @clear="onClear"
    :categories="itemCardCategories"
    :item-type="null"
    :sub-type-list="subTypeList"
    :value="itemId"
    @selected="onSelect"
    @stash="onStash"
  ></ItemCardSelectCategorized>
</template>

<style scoped></style>
