<script setup lang="ts">
import { CampaignStore } from "@/store/CampaignStore";
import { ref, watch } from "vue";
import { useI18n } from "vue-i18n";

const props = defineProps<{
  campaignId: string;
}>();
const { t } = useI18n();

const campaignStore = CampaignStore();
const campaign = campaignStore.find(props.campaignId);
const name = ref(campaign.name);

watch(name, async (newName) => {
  campaignStore.find(props.campaignId).name = newName;
});
</script>

<template>
  <div class="p-4 grid gap-2">
    <span class="w-48">{{ t("text.party-name") }}</span>
    <input type="text" id="campaign-log-name" v-model="name" class="w-full bg-base-100 focus:ring-0 rounded-lg" />
  </div>
</template>

<style scoped></style>
