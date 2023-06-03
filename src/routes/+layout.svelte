<script lang="ts">
  // The ordering of these imports is critical to your app working properly
  import '../theme.postcss';
  // If you have source.organizeImports set to true in VSCode, then it will auto change this ordering
  import '@skeletonlabs/skeleton/styles/all.css';
  // Most of your app wide CSS should be put in this file
  import '../app.postcss';
  import { AppShell, AppBar, TableOfContents } from '@skeletonlabs/skeleton';
  import type { ModalSettings, ModalComponent } from '@skeletonlabs/skeleton';
  import { Drawer, drawerStore } from '@skeletonlabs/skeleton';
  import { Modal, modalStore } from '@skeletonlabs/skeleton';

  import Navigation from '$lib/navigation/Navigation.svelte';
  import { page } from '$app/stores';
  import { beforeNavigate, afterNavigate } from '$app/navigation';
  import DocsSearch from './DocsSearch.svelte';

  import logo from '$lib/assets/logo.svg';
  import SearchButton from './SearchButton.svelte';
  import { tick } from 'svelte';
  import { writable } from 'svelte/store';
  import copy from 'clipboard-copy';

  async function drawerOpen() {
    drawerStore.open({});
    await tick();
    if (document.activeElement?.id === 'search-button') {
      (document.activeElement as HTMLButtonElement).blur();
    }
  }

  const modalRegistry: Record<string, ModalComponent> = {
    search: {
      ref: DocsSearch,
      props: {
        results: writable({ term: '', results: [] })
      }
    }
  };

  function triggerSearch(): void {
    const d: ModalSettings = {
      type: 'component',
      component: 'search',
      position: 'item-start'
    };
    modalStore.trigger(d);
  }

  // Keyboard Shortcut (CTRL/⌘+K) to Focus Search
  function onWindowKeydown(e: KeyboardEvent): void {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
      // Prevent default browser behavior of focusing URL bar
      e.preventDefault();
      // If modal currently open, close modal (allows to open/close search with CTRL/⌘+K)
      $modalStore.length ? modalStore.close() : triggerSearch();
    }
  }

  const noToC = ['/'];
  $: displayToC = !noToC.includes($page.url.pathname);

  beforeNavigate((nav) => {
    if (nav.type == 'form') return;
    document
      .querySelectorAll('.copy-content')
      .forEach((el) => el.removeEventListener('click', copyContent));
  });

  afterNavigate((nav) => {
    if (nav.type == 'link') {
      // If linked to a page, sometimes it won't scroll to the top.
      const id = nav.to?.url.hash ? nav.to.url.hash.slice(1) : 'page';
      document.getElementById(id)?.scrollTo(0, 0);
    } else if (nav.type == 'enter' && $page.url.hash) {
      const el = document.getElementById($page.url.hash.substring(1));
      if (el) el.scrollIntoView();
    }

    if (nav.type != 'form') {
      copyBoxes();
    }
  });

  const checkedIcon =
    '<path fill="currentColor" d="m10.6 16.2l7.05-7.05l-1.4-1.4l-5.65 5.65l-2.85-2.85l-1.4 1.4l4.25 4.25ZM5 21q-.825 0-1.413-.588T3 19V5q0-.825.588-1.413T5 3h14q.825 0 1.413.588T21 5v14q0 .825-.588 1.413T19 21H5Zm0-2h14V5H5v14ZM5 5v14V5Z"/>';

  async function copyContent(e: Event) {
    if (!e.target) return;
    const parent = (e.target as HTMLElement).closest('pre');
    if (!parent) return;

    const codeEl = parent.querySelector('code');
    const svg = parent.querySelector('.copy-content svg');

    if (codeEl) {
      await copy(codeEl.innerText);
      if (svg) {
        const oldContent = svg.innerHTML;
        svg.innerHTML = checkedIcon;
        setTimeout(() => (svg.innerHTML = oldContent), 800);
      }
    }
  }

  function copyBoxes() {
    document
      .querySelectorAll<HTMLElement>('pre:not(.super-debug--pre) > code')
      .forEach((el) => {
        const pre = el.parentElement as HTMLPreElement;
        var copy = document.createElement('div');
        copy.classList.add('copy-content');
        copy.addEventListener('click', copyContent);

        copy.innerHTML =
          '<button type="button"><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path fill="currentColor" d="M5 22q-.825 0-1.413-.588T3 20V6h2v14h11v2H5Zm4-4q-.825 0-1.413-.588T7 16V4q0-.825.588-1.413T9 2h9q.825 0 1.413.588T20 4v12q0 .825-.588 1.413T18 18H9Zm0-2h9V4H9v12Zm0 0V4v12Z"/></svg></button>';

        pre.prepend(copy);
      });
  }
</script>

<svelte:head>
  <title>Wealth Advisors</title>
</svelte:head>

<svelte:window on:keydown|stopPropagation={onWindowKeydown} />

<Modal components={modalRegistry} />

<Drawer width="w-70">
  <h3 class="p-4">Navigation</h3>
  <hr />
  <Navigation />
</Drawer>
<!-- App Shell -->
<AppShell regionPage="shrink-0" slotSidebarLeft="bg-surface-500/5 w-0 lg:w-56">
  <svelte:fragment slot="header">
    <!-- App Bar -->
    <AppBar>
      <svelte:fragment slot="lead">
        <div class="flex shrink-0 items-center">
          <button class="lg:hidden btn btn-sm mr-4" on:click={drawerOpen}>
            <span>
              <svg viewBox="0 0 100 80" class="hamburger w-4 h-4">
                <rect width="100" height="20" />
                <rect y="30" width="100" height="20" />
                <rect y="60" width="100" height="20" />
              </svg>
            </span>
          </button>
          <img
            class="logo mr-3 hidden lg:block"
            src={logo}
            alt="wealth advisors logo" />
          <strong class="text-lg md:text-xl truncate">Wealth Advisors</strong>
        </div>
      </svelte:fragment>
      <svelte:fragment slot="trail">
        <SearchButton cls="hidden md:inline" />
      </svelte:fragment>
    </AppBar>
  </svelte:fragment>
  <svelte:fragment slot="sidebarLeft">
    <Navigation />
  </svelte:fragment>
  <!-- Page Route Content -->
  <slot />
  <svelte:fragment slot="sidebarRight">
    {#key $page.url.pathname}
      <TableOfContents
        width="w-56"
        class="{displayToC ? 'hidden md:block' : 'hidden'} p-4"
        target="#page" />
    {/key}
  </svelte:fragment>
</AppShell>

<style lang="scss">
  .hamburger {
    fill: #b7a73f;
  }

  svg {
    color: #b7a73f;
    width: 24px;
    height: 24px;
  }

  img.logo {
    width: 36px;
    height: 36px;
  }

  .sponsor.btn {
    z-index: 4;
  }

  .sponsor.card {
    top: 10px;
    right: 5px;
  }

  :global(pre:hover .copy-content) {
    visibility: visible;
  }

  :global(.copy-content) {
    float: right;
    color: #84792e;
    visibility: hidden;
  }
</style>
