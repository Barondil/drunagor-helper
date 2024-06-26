<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { Combobox, ComboboxButton, ComboboxInput, ComboboxOption, ComboboxOptions } from "@headlessui/vue";
import { CheckIcon, ChevronUpDownIcon, XMarkIcon } from "@heroicons/vue/20/solid";
import type { FollowerRepository } from "@/data/repository/campaign/FollowerRepository";
import { CampaignStore } from "@/store/CampaignStore";
import type { Follower } from "@/data/repository/campaign/Follower";
import { useI18n } from "vue-i18n";

const props = defineProps<{
  campaignId: string;
  repository: FollowerRepository;
}>();

const campaignStore = CampaignStore();
const { t } = useI18n();

const followers = props.repository.findAll();

const followerIds = ref([] as string[]);
followerIds.value = campaignStore.find(props.campaignId).followerIds ?? [];

let filteredFollowers = computed(() =>
  query.value === ""
    ? followers
    : followers.filter((follower) =>
        follower.name.toLowerCase().replace(/\s+/g, "").includes(query.value.toLowerCase().replace(/\s+/g, ""))
      )
);

let query = ref("");

function clearSelection() {
  followerIds.value = [];
  query.value = "";
}

function findFollowers(followerIds: string[]): Follower[] {
  const followers: Follower[] = [];
  followerIds.forEach((followerId) => {
    let follower = props.repository.find(followerId);
    if (follower) {
      followers.push(follower);
    }
  });

  return followers;
}

watch(followerIds, (newFollowerIds) => {
  campaignStore.find(props.campaignId).followerIds = newFollowerIds;
});
</script>

<template>
  <Combobox v-model="followerIds" multiple>
    <div class="relative mt-1">
      <div class="relative w-full cursor-default overflow-hidden rounded-lg text-left">
        <ComboboxButton as="div" class="flex">
          <ComboboxInput
            class="w-full bg-base-100 py-2 pl-3 pr-16 leading-5 focus:ring-0 rounded-lg"
            :placeholder="t('text.add-or-remove-follower')"
            @change="query = $event.target.value"
            id="story-record-follower"
          />
        </ComboboxButton>
        <button
          id="story-record-follower-clear"
          @click="clearSelection"
          v-if="followerIds.length > 0"
          class="absolute inset-y-0 right-7 flex items-center pr-2"
        >
          <XMarkIcon class="h-5 w-5 text-white" />
        </button>
        <ComboboxButton class="absolute inset-y-0 right-0 flex items-center pr-2">
          <ChevronUpDownIcon class="h-5 w-5 text-white" />
        </ComboboxButton>
      </div>

      <ComboboxOptions
        class="bg-base-100 -top-2 transform -translate-y-full absolute mt-1 max-h-60 w-full overflow-auto rounded-md py-1"
        id="story-record-follower-options"
      >
        <div v-if="filteredFollowers.length === 0" class="relative cursor-default select-none py-2 px-4 text-white">
          Nothing found.
        </div>

        <ComboboxOption
          v-for="follower in filteredFollowers"
          as="template"
          :key="follower.id"
          :value="follower.id"
          v-slot="{ selected, active }"
        >
          <li
            class="relative cursor-pointer select-none py-2 pl-10 pr-4"
            :class="{
              'bg-emerald-500 text-white': active,
              'text-white': !active,
            }"
          >
            <span class="block truncate">
              {{ follower.name }}
            </span>
            <span
              v-if="selected"
              class="absolute inset-y-0 left-0 flex items-center pl-3"
              :class="{ 'text-white': active, 'text-emerald-500': !active }"
            >
              <CheckIcon class="h-5 w-5" />
            </span>
          </li>
        </ComboboxOption>
      </ComboboxOptions>
    </div>
  </Combobox>
  <template v-if="followerIds.length > 0">
    <template v-for="follower in findFollowers(followerIds)" :key="follower.id">
      <ul id="story-record-follower-display" class="pt-2 list-disc list-inside">
        <li>
          {{ follower.name }}
        </li>
      </ul>
    </template>
  </template>
</template>

<style scoped></style>
