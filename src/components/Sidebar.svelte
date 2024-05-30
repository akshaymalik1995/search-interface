<script lang="ts">
  const routes = [
    {
      name: "Home",
      path: "/",
    },
    {
      name: "Search",
      path: "/search",
    },
  ];
</script>

<div class="absolute left-0 bottom-0 h-full w-64 shadow-lg overflow-hidden">
  <ul class="space-y-2 p-4">
    {#each routes as route}
      <li class="list-none ">
        <a class="block px-4 py-2 rounded-md hover:bg-gray-200 transition duration-150 ease-in-out {route.path === window.location.pathname ? 'bg-gray-200' : '' } " href={route.path} >
          {route.name}
        </a>
      </li>
    {/each}
  </ul>
</div>
