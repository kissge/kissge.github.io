<script lang="ts">
  import { onMount } from 'svelte';
  import type { Endpoints } from '@octokit/types';

  type ListUserReposResponse = Endpoints['GET /users/{username}/repos']['response']['data'];
  type Repository = ListUserReposResponse[0];

  let userId = 'kissge';
  let repositories: Repository[] = [];
  let loading = true;

  onMount(async () => {
    getUserId();
    await listRepositories();
  });

  function getUserId() {
    const match = location.href.match(/([^\/.]+)\.github\.io/);
    if (match) {
      userId = match[1];
    }
  }

  async function listRepositories() {
    const allRepos: Repository[] = [];
    let url: string | null = `https://api.github.com/users/${userId}/repos?per_page=100`;

    do {
      console.debug('Fetching', url);
      const response = await fetch(url);
      console.debug(response);
      const repositories = (await response.json()) as ListUserReposResponse;
      url = getNextPageURL(response.headers);
      allRepos.push(...repositories);
    } while (url);

    repositories = allRepos
      .filter(({ has_pages, archived, disabled }) => has_pages && !archived && !disabled)
      .sort(
        (a, b) =>
          (b.stargazers_count || 0) - (a.stargazers_count || 0) ||
          (b.updated_at || '').localeCompare(a.updated_at || ''),
      );
    loading = false;
  }

  function getNextPageURL(headers: Headers) {
    const link = headers.get('Link');
    if (!link) {
      return null;
    }

    const match = link.match(/<([^>]+)>; rel="next"/);
    if (!match) {
      return null;
    }

    return match[1];
  }

  function isURL(str: string) {
    try {
      new URL(str);
      return true;
    } catch (e) {
      return false;
    }
  }

  function formatDate(date: string | null | undefined) {
    return date ? new Date(date).toLocaleString(undefined, { dateStyle: 'long', timeStyle: 'full' }) : '';
  }
</script>

<main>
  <h1>{userId}’s GitHub Pages contents</h1>

  <p>
    This is an automatically generated list using GitHub API. You can set up your own list page easily by <a
      href="https://github.com/kissge/kissge.github.io/generate">using this page as a template</a
    >.
  </p>

  {#if loading}
    <img src="https://cdnjs.cloudflare.com/ajax/libs/galleriffic/2.0.1/css/loader.gif" alt="Loading" />
  {:else}
    <ol>
      {#each repositories as repo}
        <li>
          {repo.language}<br />
          <a href={repo.homepage || `//${userId}.github.io/${repo.name}`} class="title">{repo.full_name}</a>
          {#if repo.description}
            {#if isURL(repo.description)}
              <a href={repo.description}>{repo.description}</a>
            {:else}
              {repo.description}
            {/if}
          {/if}
          <br />
          <time datetime={repo.updated_at}>Last updated at: {formatDate(repo.updated_at)}</time><br />
          {#if repo.topics}
            {#each repo.topics as topic}
              <span class="topic">{topic}</span>
            {/each}
          {/if}

          <div class="right">
            <a href={repo.html_url} class="github">Source on GitHub</a>
          </div>
        </li>
      {/each}
    </ol>
  {/if}
</main>
