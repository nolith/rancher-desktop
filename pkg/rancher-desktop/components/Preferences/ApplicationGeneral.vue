<script lang="ts">
import { Checkbox } from '@rancher/components';
import Vue from 'vue';
import { mapGetters } from 'vuex';

import RdFieldset from '@pkg/components/form/RdFieldset.vue';
import { Settings } from '@pkg/config/settings';
import { RecursiveTypes } from '@pkg/utils/typeUtils';

import type { PropType } from 'vue';

export default Vue.extend({
  name:       'preferences-application-general',
  components: { Checkbox, RdFieldset },
  props:      {
    preferences: {
      type:     Object as PropType<Settings>,
      required: true,
    },
  },
  data() {
    return {
      sudoAllowedTooltip: `
        If checked, Rancher Desktop will attempt to acquire administrative
        credentials ("sudo access") when starting for some operations.  This
        allows for enhanced functionality, including bridged networking and
        default docker socket support.  Changes will only be applied next time
        Rancher Desktop starts.
      `,
      automaticUpdates: true,
      statistics:       false,
    };
  },
  computed: {
    ...mapGetters('preferences', ['isPlatformWindows']),
    isSudoAllowed(): boolean {
      return this.preferences?.application?.adminAccess ?? false;
    },
    canAutoUpdate(): boolean {
      return this.preferences?.application.updater.enabled ?? false;
    },
  },
  methods: {
    onChange<P extends keyof RecursiveTypes<Settings>>(property: P, value: RecursiveTypes<Settings>[P]) {
      this.$store.dispatch('preferences/updatePreferencesData', { property, value });
    },
  },
});
</script>

<template>
  <div class="application-general">
    <rd-fieldset
      v-if="!isPlatformWindows"
      data-test="administrativeAccess"
      legend-text="Administrative Access"
      :legend-tooltip="sudoAllowedTooltip"
    >
      <checkbox
        label="Allow to acquire administrative credentials (sudo access)"
        :value="isSudoAllowed"
        @input="onChange('application.adminAccess', $event)"
      />
    </rd-fieldset>
    <rd-fieldset
      data-test="automaticUpdates"
      legend-text="Automatic Updates"
    >
      <checkbox
        data-test="automaticUpdatesCheckbox"
        label="Check for updates automatically"
        :value="canAutoUpdate"
        @input="onChange('application.updater.enabled', $event)"
      />
    </rd-fieldset>
    <rd-fieldset
      data-test="statistics"
      legend-text="Statistics"
    >
      <checkbox
        label="Allow collection of anonymous statistics to help us improve Rancher Desktop"
        :value="preferences.application.telemetry.enabled"
        @input="onChange('application.telemetry.enabled', $event)"
      />
    </rd-fieldset>
  </div>
</template>

<style lang="scss" scoped>
  .application-general {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
</style>
