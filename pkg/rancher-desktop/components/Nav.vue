<template>
  <nav>
    <ul>
      <li v-for="item in items" :key="item.route" :item="item.route">
        <NuxtLink :to="item.route">
          {{ routes[item.route].name }}
          <badge-state
            v-if="item.error"
            color="bg-error"
            class="nav-badge"
            :label="item.error.toString()"
          />
        </NuxtLink>
      </li>
    </ul>
    <template v-if="featureExtensions">
      <hr>
      <nav-item>
        <template #before>
          <span class="icon icon-circle-plus"></span>
        </template>
        Extensions
      </nav-item>
      <template v-for="extension in extensions">
        <nuxt-link
          :key="extension.id"
          :to="extensionRoute(extension)"
        >
          <nav-item :id="`extension:${extension.id}`">
            <template #before>
              <img :src="imageUri(extension.id)">
            </template>
            {{ extension.metadata.ui['dashboard-tab'].title }}
          </nav-item>
        </nuxt-link>
      </template>
    </template>
  </nav>
</template>

<script lang="ts">
import os from 'os';

import { NuxtApp } from '@nuxt/types/app';
import { BadgeState } from '@rancher/components';
import { RouteRecordPublic } from 'vue-router';

import NavItem from './NavItem.vue';

import { hexEncode } from '@pkg/utils/string-encode';

export default {
  components: {
    BadgeState,
    NavItem,
  },
  props: {
    items: {
      type:      Array,
      required:  true,
      validator: (value: {route: string, error?: number}[]) => {
        const nuxt: NuxtApp = (global as any).$nuxt;
        const routes = nuxt.$router.getRoutes().reduce((paths: Record<string, RouteRecordPublic>, route) => {
          paths[route.path] = route;

          return paths;
        }, {});

        return value && (value.length > 0) && value.every(({ route }) => {
          const result = route in routes;

          if (!result) {
            console.error(`<Nav> error: path ${ JSON.stringify(route) } not found in routes ${ JSON.stringify(Object.keys(routes)) }`);
          }

          return result;
        });
      },
    },
    extensions: {
      type:     Array,
      required: true,
    },
  },
  data() {
    const nuxt: NuxtApp = (this as any).$nuxt;

    return {
      // Generate a route (path) to route entry mapping, so that we can pick out
      // their names based on the paths given.
      routes: nuxt.$router.getRoutes().reduce((paths: Record<string, RouteRecordPublic>, route) => {
        paths[route.path] = route;
        if (route.name === 'Supporting Utilities' && os.platform() === 'win32') {
          route.name = 'WSL Integrations';
        }

        return paths;
      }, {}),
    };
  },
  computed: {
    featureExtensions(): boolean {
      const nuxt: NuxtApp = (this as any).$nuxt;

      return !!nuxt.$config.featureExtensions;
    },
  },
  methods: {
    imageUri(id: string): string {
      return `x-rd-extension://${ hexEncode(id) }/icon.svg`;
    },
    extensionRoute({ id, metadata }: { id: string, metadata: any }) {
      const { ui: { 'dashboard-tab': { root, src } } } = metadata;

      return {
        name:   'rdx-root-src-id',
        params: {
          root,
          src,
          id: hexEncode(id),
        },
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

nav {
    background-color: var(--nav-bg);
    padding: 0;
    margin: 0;
    padding-top: 20px;
}

ul {
    margin: 0;
    padding: 0;
    list-style-type: none;

    li {
        padding: 0;

        a {
            color: var(--body-text);
            text-decoration: none;
            line-height: 24px;
            padding: 7.5px 10px;
            letter-spacing: 1.4px;
            display: block;
            outline: none;
        }

        a.nuxt-link-active {
            background-color: var(--nav-active);
        }
    }
}

a {
  &:hover {
    text-decoration: none;
  }

  &.nuxt-link-active::v-deep div {
    background-color: var(--nav-active);
  }
}
.nav-badge {
  line-height: initial;
  letter-spacing: initial;
  font-size: 0.75rem;
}

</style>
