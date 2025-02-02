<template>
  <div id="docsearch" />
</template>

<script setup lang="ts">
import { useRoute, useRouter } from "vitepress";
import { onMounted, watch } from "vue";

import docsearch from "@docsearch/js";
import "@docsearch/css/dist/style.css";
import useData from "../../composables/data";

const { theme } = useData();

const options = theme.value.algolia;

const route = useRoute();
const router = useRouter();

watch(
  () => options,
  (value) => {
    initialize(value);
  }
);

onMounted(() => {
  initialize(options);
});

function isSpecialClick(event: MouseEvent) {
  return event.button === 1 || event.altKey || event.ctrlKey || event.metaKey || event.shiftKey;
}

const docsearchProps: Partial<Parameters<typeof docsearch>[0]> = {
  container: "#docsearch",

  navigator: {
    navigate: ({ itemUrl }: { itemUrl: string }) => {
      const url = new URL(window.location.origin + itemUrl);
      // Router doesn't handle same-page navigation so we use the native
      // browser location API for anchor navigation
      if (route.path === url.pathname) {
        window.location.assign(url);
      } else {
        router.go(itemUrl);
      }
    },
  },
  transformItems: (items) => {
    const transformedItems: typeof items = [];

    for (const item of items) {
      const url = new URL(item.url);

      if (url.pathname.endsWith(".html")) continue;
      if (url.hash === "#app") url.hash = "";

      const transformedItem = {
        ...item,
        url: url.pathname + url.hash,
      };

      transformedItems.push(transformedItem);
    }

    return transformedItems;
  },
  hitComponent: ({ hit, children }: { hit: any; children: any }) => {
    return {
      type: "a",
      key: undefined,
      constructor: undefined,
      ref: undefined,
      props: {
        href: hit.url,
        onClick: (event: MouseEvent) => {
          // we rely on the native link scrolling when user is already on
          // the right anchor because Router doesn't support duplicated
          // history entries
          if (isSpecialClick(event) || route.path === hit.url) return;

          // if the hits goes to another page, we prevent the native link
          // behavior to leverage the Router loading feature
          event.preventDefault();
          router.go(hit.url);
        },
        children,
      },
      __v: null,
    };
  },
};

function initialize(userOptions: any) {
  docsearch({ ...userOptions, ...docsearchProps });
}
</script>

<style lang="scss">
.DocSearch {
  --docsearch-primary-color: var(--accent-color);
  --docsearch-text-color: #252525;
  --docsearch-spacing: 1em;
  --docsearch-icon-stroke-width: 1.4;
  --docsearch-highlight-color: var(--docsearch-primary-color);
  --docsearch-muted-color: rgb(150, 159, 175);
  --docsearch-container-background: rgba(0, 0, 0, 0.8);
  --docsearch-logo-color: var(--docsearch-primary-color);

  /* modal */
  --docsearch-modal-width: 600px;
  --docsearch-modal-height: 600px;
  --docsearch-modal-background: var(--bg-color);
  --docsearch-modal-shadow: none;

  /* searchbox */
  --docsearch-searchbox-height: 50px;
  --docsearch-searchbox-background: var(--bg-color);
  --docsearch-searchbox-focus-background: var(--light-bg-color);
  --docsearch-searchbox-shadow: none;

  /* hit */
  --docsearch-hit-height: 50px;
  --docsearch-hit-shadow: none;
  --docsearch-hit-background: var(--light-bg-color);
  --docsearch-hit-color: currentColor;

  /* key */
  --docsearch-key-gradient: none;
  --docsearch-key-shadow: none;

  /* footer */
  --docsearch-footer-background: var(--light-bg-color);
  --docsearch-footer-shadow: inset 0 1px 0 var(--border-color);
}

/* Darkmode */
.dark .DocSearch {
  --docsearch-text-color: rgb(245, 246, 247);
  --docsearch-logo-color: rgb(255, 255, 255);
}
.DocSearch-Modal {
  border-radius: var(--border-radius);
  border: var(--border);
}
.DocSearch-Form {
  border-radius: var(--border-radius);
  border: var(--border);
}
.DocSearch-Button {
  border-radius: var(--border-radius);
  border: var(--border);
  margin: 0;

  .DocSearch-Button-Keys {
    display: none;
  }
}
.DocSearch-Button-Placeholder {
  &::after {
    content: "…";
  }
}
.DocSearch-Button-Key {
  padding: 0;
  margin: 0;
}
.DocSearch-Logo svg * {
  fill: var(--accent-color);
}
</style>
