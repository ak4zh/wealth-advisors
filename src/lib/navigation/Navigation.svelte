<script lang="ts">
  import { onMount } from 'svelte';
  import { drawerStore } from '@skeletonlabs/skeleton';
  import A from './A.svelte';
  import Title from './Title.svelte';
  import { page } from '$app/stores';
  import SearchButton from '../../routes/SearchButton.svelte';

  function drawerClose(): void {
    drawerStore.close();
  }

  let nav: HTMLElement;

  onMount(() => {
    nav.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      if (target && target.tagName == 'A') {
        drawerClose();
      }
    });
  });
</script>

<nav class="list-nav p-4" bind:this={nav}>
  <SearchButton cls="md:hidden mx-4 mb-3" width="w-40" />
  <ul>
    <li><A href="/">Home</A></li>

    <Title id="concepts">Regulatory</Title>

    <li><A href="/tos">Terms of Service</A></li>
    <li><A href="/privacy">Privacy Policy</A></li>
    <li><A href="/contact-us">Contact Us</A></li>
  </ul>
</nav>
