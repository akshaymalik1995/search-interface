<script lang="ts">
  // Importing a custom icon component
  import CloseIcon from "../icons/CloseIcon.svelte";
  import LoadingIcon from "../icons/LoadingIcon.svelte";
  import data from "../data";

  // Interface definitions for attributes, queries, and search state
  interface IATTRIBUTE {
    name: string;
  }

  interface IRESPONSE_ITEM {
    id: string;
    text: string;
    filename: string;
  }

  interface IQUERY {
    [attribute: string]: string[];
  }

  interface ISEARCH {
    input: string;
    error: string | null;
    attrs: IATTRIBUTE[];
    query: IQUERY;
    activeAttributeName: string;
    results: IRESPONSE_ITEM[];
    addKeywords: (value: string) => void;
    removeKeyword: (attr: string, keyword: string) => void;
    editKeyword: (attr: string, keword: string) => void;
    prepareQuery: () => string;
    sendQuery: () => void;
    status: "idle" | "loading" | "done";
  }

  // Initial search state
  const search: ISEARCH = {
    input: "",
    status: "idle",
    error: null,
    attrs: [
      {
        name: "Text",
      },
      {
        name: "Filename",
      },
    ],
    // Make sure you set the activeAttributeName while setting attrs
    activeAttributeName: "",
    query: {},
    addKeywords(keyword: string) {
      // Find the active attribute based on its name
      const activeAttribute: IATTRIBUTE | undefined = search.attrs.find(
        (attr) => attr.name === search.activeAttributeName
      );
      if (activeAttribute) {
        // Check if the keyword already exists in the query for the active attribute
        const attrName: string = activeAttribute.name;
        if (attrName in search.query) {
          if (search.query[attrName].includes(keyword)) {
            // If the keyword is already present, do nothing
            return;
          }
          // Add the keyword to the query array for the active attribute
          search.query[attrName] = [...search.query[attrName], keyword];
        } else {
          // Initialize the query array for the active attribute with the keyword
          search.query[attrName] = [keyword];
        }
      }
      // Clear the input field after adding keywords
      search.input = "";
    },
    removeKeyword: (attr: string, keyword: string) => {
      // Remove the specified keyword from the query for the given attribute
      search.query[attr] = search.query[attr].filter(
        (item) => item !== keyword
      );
    },
    editKeyword: (attr: string, keyword: string) => {
      search.activeAttributeName = attr;
      search.input = keyword;
      search.removeKeyword(attr, keyword);
    },
    // Method to construct the final query string from the search state
    prepareQuery() {
      let q = ""; // Initialize the query string
      const attrsSep = "&"; // Separator for attributes in the URL query
      const keywordSep = "-"; // Separator for keywords within an attribute
      const attrKeywordSep = "="; // Separator between attribute names and their values

      // Iterate over each attribute in the search query
      Object.keys(search.query).forEach((attrName, index) => {
        const keywords = search.query[attrName]; // Get all keywords for the current attribute
        const attr = search.attrs.find((item) => item.name === attrName); // Find the attribute object by name
        if (attr && keywords.length) {
          // Append the attribute name and equals sign to the query string
          q += attr.name + attrKeywordSep;

          // Iterate over each keyword for the current attribute
          keywords.forEach((keyword, i) => {
            // Determine if the current keyword is the last one for this attribute
            if (i === keywords.length - 1) {
              // If so, append the keyword without a separator
              q += keyword;
            } else {
              // Otherwise, append the keyword followed by the keyword separator
              q += keyword + keywordSep;
            }
          });

          // Append the attribute separator if it's not the last attribute in the query
          if (index !== Object.keys(search.query).length - 1) {
            q += attrsSep;
          }
        }
      });

      console.log(q); // Debugging: log the constructed query string
      return q; // Return the final query string
    },

    results: [],

    sendQuery() {
      // Change it as per your needs
      if (search.input){
        search.addKeywords(search.input)
      }

      const q = search.prepareQuery();
      if (!q) {
        alert("Search query is empty.");
        return;
      }

      let textKeywords: string[] = search.query["Text"] || [];
      let fileKeywords: string[] = search.query["Filename"] || [];
      search.error = null;
      search.status = "loading";
      try {
        setTimeout(() => {
          search.results = data.filter((item) => {
            for (let keyword of textKeywords) {
              if (item.text.includes(keyword)) return item;
            }

            for (let keyword of fileKeywords) {
              if (item.text.includes(keyword)) return item;
            }
          });
          search.status = "done";
        }, 2000);
      } catch (err) {
        search.error = err as string;
        search.status = "done";
      }
    },
  };

  if (search.attrs.length) {
    search.activeAttributeName = search.attrs[0].name;
  }

  // Event listeners for handling user interactions
  function onInput(e: Event) {
    // Update the search input when the user types in the input field
    const target = e.target;
    if (target instanceof HTMLInputElement) {
      search.input = target.value;
    }
  }

  function onInputSubmit(e: Event) {
    // Prevent the default form submission behavior
    e.preventDefault();

    // Process the input when the submit button is clicked
    const keyword = search.input;
    if (!keyword) return;
    search.addKeywords(keyword);
    search.input = ""; // Clear the input field
    console.log(search.query); // Log the current query for debugging purposes
  }

  function onSelectChange(e: Event) {
    // Update the active attribute when the user selects a different option
    const target = e.target;
    if (target instanceof HTMLSelectElement) {
      const keyword = target.value;
      search.activeAttributeName = keyword;
    }
  }
</script>

<div class="flex gap-4 text-sm">
  <div class="flex flex-col gap-2">
    <label for="search-attrs" class="">Options</label>
    <select
      on:change={(e) => onSelectChange(e)}
      id="search-attrs"
      class="shadow p-2 focus:outline-none"
      name=""
    >
      {#each search.attrs as attr}
        <option
          selected={search.activeAttributeName === attr.name}
          value={attr.name}>{attr.name}</option
        >
      {/each}
    </select>
  </div>

  <form class="flex flex-col grow gap-2" on:submit={(e) => onInputSubmit(e)}>
    <label class="" for="search-keyword">Keywords</label>
    <input
      value={search.input}
      on:input={(e) => onInput(e)}
      class="p-2 shadow focus:outline-none"
      id="search-keyword"
      type="text"
    />
    <button type="submit"></button>
  </form>
</div>

<div class="flex my-2 flex-wrap text-sm gap-4">
  {#each Object.keys(search.query) as attr}
    {#if search.query[attr].length}
      <div class="flex items-center shadow">
        <div class="bg-blue-100 p-2">{attr}</div>
        <div class="flex text-sm px-2 items-center gap-2">
          {#each search.query[attr] as keyword}
            <div class="flex items-center gap-2">
              <!-- svelte-ignore a11y-click-events-have-key-events -->
              <!-- svelte-ignore a11y-no-static-element-interactions -->
              <div
                class="cursor-pointer"
                on:click={() => search.editKeyword(attr, keyword)}
              >
                {keyword}
              </div>
              <!-- svelte-ignore a11y-click-events-have-key-events -->
              <!-- svelte-ignore a11y-no-static-element-interactions -->
              <div
                on:click={() => search.removeKeyword(attr, keyword)}
                class="cursor-pointer text-xs"
              >
                <CloseIcon width={16} height={20} />
              </div>
            </div>
          {/each}
        </div>
      </div>
    {/if}
  {/each}
</div>

<button
  disabled={search.status === "loading"}
  on:click={() => search.sendQuery()}
  class="bg-blue-400 my-2 hover:bg-blue-300 py-1 rounded w-full">Search</button
>

<!-- RESULTS -->

<div class="flex w-full my-4 flex-col gap-4">
  {#if search.status === "done" && !search.error}
    {#each search.results as item}
      <div class="rounded border">
        <div class="bg-gray-200 p-2">{item.filename}</div>
        <div class=" p-2">{item.text}</div>
      </div>
    {:else}
      <div class="text-sm" >No Results Found</div>
    {/each}
  {/if}
  {#if search.status === "done" && search.error}
    <div>Failed!</div>
  {/if}
  {#if search.status === "loading"}
    <div class="flex justify-center w-full">
      <div class="w-16 h-16"><LoadingIcon width={8} height={8}  /></div>
    </div>
  {/if}
</div>
