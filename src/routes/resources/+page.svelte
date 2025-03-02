<script lang="ts">
  import { base } from "$app/paths";
  import { github_url } from "$lib/constants";
  import type { PageData } from "./$types";
  export let data: PageData;

  let resources = data.payload.resources;
  let displayedResources = resources;
  let tagLogicAnd: boolean = true; // Whether all the selected tags must match the resource (vs any of the selected tags)
  // TODO: make this a user preference
  $: tagLogic = tagLogicAnd ? "and" : "or";

  let tags = data.payload.tags;
  let tags_count = data.payload.tags_count;
  // Creating filter object
  let filterObject: any = {};
  filterObject["tags"] = {};
  let tag: string;
  for (tag of tags) {
    filterObject.tags[tag] = false;
  }

  let removeWhitespace = (str: string) => {
    return str.replace(/\s/g, "");
  };

  function filterResources(event) {
    // Reset displayed resources
    displayedResources = [];

    let resource;
    let tag: string;

    // Tags of interest
    let filterTags: string[] | Set<string> = Object.keys(
      filterObject.tags
    ).filter((tag) => filterObject.tags[tag] === true);
    filterTags = new Set(filterTags);

    let minCommonTags = tagLogic ? filterTags.size : 1;
    let resourceTags: Set<string>;
    for (resource of resources) {
      // Resource tags
      resourceTags = new Set(resource.tags);

      if (setIntersection(filterTags, resourceTags).size >= minCommonTags) {
        displayedResources.push(resource);
      }
      console.log(resource.title, setIntersection(filterTags, resourceTags));
    }

    // Force svelte re-render
    console.log(filterTags);
    displayedResources = displayedResources;
  }

  function setIntersection(set1: Set<any>, set2: Set<any>) {
    let intersection = new Set();
    for (let element of set2) {
      if (set1.has(element)) {
        intersection.add(element);
      }
    }
    return intersection;
  }

  import ListItem from "./ListItem.svelte";
</script>

<h1>Resources</h1>
<div class="py-1">
  <i>{resources.length} resources and counting!!</i>
</div>
<div class="flex flex-wrap gap-2 pb-3">
  <a
    class="p-2 rounded-lg border-2 border-green-500 text-green-500"
    href="{github_url}/issues/new/choose"
    target="_blank"
    rel="noreferrer"
  >
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      stroke-width="1.5"
      stroke="currentColor"
      class="w-6 h-6 inline"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M19.5 14.25v-2.625a3.375 3.375 0 00-3.375-3.375h-1.5A1.125 1.125 0 0113.5 7.125v-1.5a3.375 3.375 0 00-3.375-3.375H8.25m3.75 9v6m3-3H9m1.5-12H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 00-9-9z"
      />
    </svg>
    Suggest resource
    <span class="sr-only">in a new tab</span>
  </a>

  <a
    class="p-2 rounded-lg border-2 border-green-500 text-green-500"
    href="{github_url}/edit/main/data/resources.yml"
    target="_blank"
    rel="noreferrer"
  >
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      stroke-width="1.5"
      stroke="currentColor"
      class="w-6 h-6 inline"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M16.862 4.487l1.687-1.688a1.875 1.875 0 112.652 2.652L10.582 16.07a4.5 4.5 0 01-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 011.13-1.897l8.932-8.931zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0115.75 21H5.25A2.25 2.25 0 013 18.75V8.25A2.25 2.25 0 015.25 6H10"
      />
    </svg>
    Edit
    <span class="sr-only">in a new tab</span>
  </a>

  <a
    class="p-2 rounded-lg border-2 border-green-500 text-green-500"
    href="{base}/ClimateTown-Knowledge-Hub-resources.csv"
    download
  >
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="16"
      height="16"
      fill="currentColor"
      class="w-6 h-6 inline"
      viewBox="0 0 16 16"
    >
      <path
        d="M.5 9.9a.5.5 0 0 1 .5.5v2.5a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-2.5a.5.5 0 0 1 1 0v2.5a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2v-2.5a.5.5 0 0 1 .5-.5z"
      />
      <path
        d="M7.646 11.854a.5.5 0 0 0 .708 0l3-3a.5.5 0 0 0-.708-.708L8.5 10.293V1.5a.5.5 0 0 0-1 0v8.793L5.354 8.146a.5.5 0 1 0-.708.708l3 3z"
      />
    </svg>
    Download resources
  </a>
</div>

<details class="rounded-lg border-2" open>
  <summary class="cursor-pointer text-2xl p-2">
    <svg
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      stroke-width="1.5"
      stroke="currentColor"
      class="w-6 h-6 inline transition-transform duration-100 ease-in-out"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        d="M19.5 8.25l-7.5 7.5-7.5-7.5"
      />
    </svg>

    Filter
  </summary>
  <div>
    <!-- begin form -->
    <form
      on:submit|preventDefault={filterResources}
      class="border-solid p-4 space-y-4"
    >
      <!-- <label for="search">Search</label> -->
      <!-- <input type="text" id="search" name="search" /> -->
      <div class="flex flex-row flex-wrap gap-2">
        {#each tags as tag}
          <!-- checkboxes -->
          <div class="flex justify-between gap-2 rounded-full bg-gray-300">
            <label
              class="cursor-pointer py-2 px-3 rounded-full flex items-center gap-2"
              for={removeWhitespace(tag)}
            >
              <input
                type="checkbox"
                class="appearance-none cursor-pointer w-6 h-6 bg-white rounded-full checked:bg-black transition duration-200"
                bind:checked={filterObject.tags[tag]}
                id={removeWhitespace(tag)}
                name={removeWhitespace(tag)}
              />
              <span>
                {tag}
                <span class="text-gray-500 italic">({tags_count[tag]})</span>
              </span>
            </label>
          </div>
        {/each}
      </div>
      <div class="flex justify-end">
        <button type="submit" class="p-2 rounded-lg bg-green-500 text-white">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-6 h-6 inline"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M12 3c2.755 0 5.455.232 8.083.678.533.09.917.556.917 1.096v1.044a2.25 2.25 0 01-.659 1.591l-5.432 5.432a2.25 2.25 0 00-.659 1.591v2.927a2.25 2.25 0 01-1.244 2.013L9.75 21v-6.568a2.25 2.25 0 00-.659-1.591L3.659 7.409A2.25 2.25 0 013 5.818V4.774c0-.54.384-1.006.917-1.096A48.32 48.32 0 0112 3z"
            />
          </svg>

          Filter
        </button>
      </div>
    </form>
  </div>
</details>

<div
  class="grid xl:grid-cols-3 md:grid-cols-2 sm:grid-cols-1 gap-x-4 gap-y-4 mt-3"
>
  {#each displayedResources as resource}
    <ListItem {...resource} />
  {:else}
    <div>No resources here!</div>
  {/each}
</div>
