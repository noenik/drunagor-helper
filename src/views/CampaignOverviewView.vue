<script setup lang="ts">
import BaseButtonMenu from "@/components/BaseButtonMenu.vue";
import BaseDivider from "@/components/BaseDivider.vue";
import CampaignNew from "@/components/CampaignNew.vue";
import type { HeroData } from "@/data/repository/HeroData";
import { HeroDataRepository } from "@/data/repository/HeroDataRepository";
import { Campaign } from "@/store/Campaign";
import { CampaignStore } from "@/store/CampaignStore";
import { Hero } from "@/store/Hero";
import { HeroStore } from "@/store/HeroStore";
import { PartyStore } from "@/store/PartyStore";
import { customAlphabet } from "nanoid";
import CampaignImport from "@/components/CampaignImport.vue";

const partyStore = PartyStore();
const legacyCampaign = partyStore.findAll();
const campaignStore = CampaignStore();
const heroStore = HeroStore();
const nanoid = customAlphabet("1234567890", 5);

//backwards compatibility
if (legacyCampaign.length > 0) {
  let campaignId = nanoid();
  campaignStore.add(new Campaign(campaignId, "core"));

  legacyCampaign.forEach((hero) => {
    let newHero = new Hero(hero.heroId, campaignId);
    newHero.auraId = hero.auraId;
    newHero.outcomeIds = hero.outcomeIds;
    newHero.statusIds = hero.statusIds;

    heroStore.add(newHero);
    partyStore.removeMember(hero.heroId);
  });
}

const heroDataRepository = new HeroDataRepository();

function findHeroes(campaignId: string): HeroData[] {
  const heroes: HeroData[] = [];
  heroStore.findAllInCampaign(campaignId).forEach((hero) => {
    heroes.push(heroDataRepository.find(hero.heroId) ?? ({} as HeroData));
  });

  return heroes;
}
</script>

<template>
  <BaseDivider>Campaigns</BaseDivider>
  <BaseButtonMenu>
    <CampaignNew />
    <CampaignImport />
  </BaseButtonMenu>
  <div id="campaigns" class="grid gap-4 pt-4 place-items-center">
    <template v-for="campaign in campaignStore.findAll()" :key="campaign.campaignId">
      <router-link :to="{ name: 'Campaign', params: { id: campaign.campaignId } }" class="w-full">
        <div class="bg-neutral p-2 drop-shadow-lg rounded-lg w-full">
          <div class="pb-4">
            <p class="text-center">
              <span class="capitalize">{{ campaign.campaign }}</span>
              <template v-if="campaign.name"> - {{ campaign.name }} </template>
            </p>
          </div>
          <div class="flex flex-wrap justify-center min-h-16">
            <template v-for="hero in findHeroes(campaign.campaignId)" :key="hero.heroId">
              <div class="w-16 h-16">
                <img class="w-14 bg-base-100 rounded-full" :src="hero.images.avatar" />
              </div>
            </template>
          </div>
        </div>
      </router-link>
    </template>
  </div>
</template>

<style scoped></style>
