<script setup>
import { computed, onMounted, provide, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { useModal } from '@/composables/useModal';
import { useI18n } from '@/composables/useI18n';
import { useDomain } from '@/composables/useDomain';
import { useApp } from '@/composables/useApp';
import { useWeb3 } from '@/composables/useWeb3';
import { useNotifications } from '@/composables/useNotifications';

const { domain } = useDomain();
const { loadLocale } = useI18n();
const route = useRoute();
const { modalOpen } = useModal();
const { init, spaces, app } = useApp();
const { web3 } = useWeb3();
const { notify } = useNotifications();

provide('web3', web3);
provide('notify', notify);

const space = computed(() => {
  const key = domain || route.params.key;
  return spaces.value[key] ? spaces.value[key] : {};
});

const isOpenTermAndCondition = ref(false);

const onClickAccept = () => {
  isOpenTermAndCondition.value = false;
  localStorage.setItem('ACCEPT_CONDITION', 'true');
};

onMounted(async () => {
  await loadLocale();
  init();
  const accept = localStorage.getItem('ACCEPT_CONDITION');
  isOpenTermAndCondition.value = accept != 'true';
});

watch(modalOpen, val => {
  const el = document.body;
  el.classList[val ? 'add' : 'remove']('overflow-hidden');
});
</script>

<template>
  <div :class="space?.skin" id="app" class="overflow-hidden pb-4">
    <UiLoading v-if="app.loading || !app.init" class="overlay big" />
    <div v-else>
      <Topnav />
      <div class="pb-6">
        <router-view :key="$route.path" class="flex-auto" />
      </div>
    </div>
    <div id="modal" />
    <Notifications />
  </div>
  <div
    v-show="isOpenTermAndCondition && !(app.loading || !app.init)"
    style="
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
      z-index: 100;
    "
  >
    <Block
      class="text-center extra-icon-container"
      style="
        margin-bottom: 24px !important;
        background-color: #fff !important;
        width: 420px;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      "
    >
      <div>
        <h1>{{ $t('settings.terms') }}</h1>
        <div>{{ $t('metaverse.condition') }}</div>
        <UiButton @click="onClickAccept">{{ $t('agree') }}</UiButton>
      </div>
    </Block>
  </div>
</template>
