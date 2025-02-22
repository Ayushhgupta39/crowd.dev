<template>
  <div class="panel !bg-purple-50">
    <div class="flex justify-between items-center">
      <div class="flex gap-2">
        <app-svg
          name="enrichment"
          class="w-5 h-5 enrichment-icon"
          color="#111827"
        />
        <span class="text-gray-900 font-semibold text-sm">Contact enrichment</span>
      </div>
      <el-tooltip placement="top" content="Learn more">
        <a
          aria-label="Learn more"
          class="btn btn--transparent !h-8 !w-8 !text-gray-400 hover:!text-gray-600"
          href="https://docs.crowd.dev/docs/member-enrichment"
          target="_blank"
          rel="noopener noreferrer"
        ><i class="ri-question-line text-lg" /></a>
      </el-tooltip>
    </div>

    <div class="mt-4 mb-5 text-2xs text-gray-600">
      Get more insights about this contact by enriching it with attributes such
      as emails, seniority, OSS contributions and much more.
    </div>

    <el-tooltip
      placement="top"
      content="Contact enrichment requires an associated GitHub profile or Email"
      :disabled="!isEnrichmentDisabled || !isEnrichmentFeatureEnabled()"
      popper-class="max-w-[260px]"
    >
      <span v-if="isFindGitHubFeatureEnabled">
        <el-button
          v-if="!isEnrichmentDisabled"
          class="btn btn--primary btn--full !h-8"
          :disabled="isEditLockedForSampleData"
          @click="onEnrichmentClick"
        >Enrich member</el-button>
        <el-button
          v-else
          class="btn btn--primary btn--full !h-8"
          :disabled="isEditLockedForSampleData"
          @click="onFindGithubClick"
        >
          <i class="ri-github-fill pr-2" /> Find GitHub
        </el-button>
      </span>
      <span v-else>
        <el-button
          class="btn btn--primary btn--full !h-8"
          :disabled="isEnrichmentActionDisabled"
          @click="onEnrichmentClick"
        >
          <span v-if="isEnrichmentFeatureEnabled()">Enrich contact</span>
          <span v-else>Upgrade plan</span>
        </el-button>
      </span>
    </el-tooltip>

    <div class="w-full text-center italic text-gray-500 text-3xs mt-2">
      * requires a GitHub profile or Email
    </div>
    <app-member-find-github-drawer
      v-if="openFindGitHubDrawer"
      v-model="openFindGitHubDrawer"
      :member="member"
    />
  </div>
</template>

<script setup>
import {
  computed, defineProps, onMounted, ref,
} from 'vue';
import { mapActions, mapGetters } from '@/shared/vuex/vuex.helpers';
import AppSvg from '@/shared/svg/svg.vue';
import { isEnrichmentFeatureEnabled } from '@/modules/member/member-enrichment';
import { useRouter } from 'vue-router';
import { FeatureFlag, FEATURE_FLAGS } from '@/utils/featureFlag';
import { MemberPermissions } from '../member-permissions';
import AppMemberFindGithubDrawer from './member-find-github-drawer.vue';

const router = useRouter();
const props = defineProps({
  member: {
    type: Object,
    default: () => {},
  },
});

const { doEnrich } = mapActions('member');
const { doRefreshCurrentUser } = mapActions('auth');
const { currentTenant, currentUser } = mapGetters('auth');

const isFindGitHubFeatureEnabled = FeatureFlag.isFlagEnabled(
  FEATURE_FLAGS.findGitHub,
);

const isEnrichmentDisabled = computed(
  () => !props.member.username?.github?.length && !props.member.emails?.length,
);

const openFindGitHubDrawer = ref(false);

const isEditLockedForSampleData = computed(
  () => new MemberPermissions(currentTenant.value, currentUser.value)
    .editLockedForSampleData,
);

const isEnrichmentActionDisabled = computed(
  () => isEnrichmentDisabled.value || isEditLockedForSampleData.value,
);

onMounted(() => {
  doRefreshCurrentUser({});
});

const onEnrichmentClick = async () => {
  if (!isEnrichmentFeatureEnabled()) {
    router.push('/settings?activeTab=plans');
    return;
  }
  await doEnrich(props.member.id);
};

const onFindGithubClick = () => {
  openFindGitHubDrawer.value = props.member;
};
</script>

<style lang="scss">
.enrichment-icon svg use {
  transform: scale(1.25);
}
</style>
