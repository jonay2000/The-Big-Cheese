<script context="module" lang="ts">
  import { enhance } from "$lib/form";
  import type { Load } from "@sveltejs/kit";

  export const load: Load = async ({ fetch }) => {
    const recipesUrl = "/recipes.json";
    const recipes = await fetch(recipesUrl);

    if (recipes.ok) {
      return {
        props: { recipes: await recipes.json() },
      };
    }

    const { message } = await recipes.json();

    return {
      status: recipes.status,
      error: new Error(message),
    };
  };
</script>

<script lang="ts">
  import { scale } from "svelte/transition";
  import { flip } from "svelte/animate";
  import { transition_in } from "svelte/internal";

  type Measurement = "Kilogram" | "Gram" | { Custom: string };

  type Quantity = {
    value: number;
    unit: Measurement;
  };

  type Ingredient = {
    title: string;
    note: string;
    quantity: Quantity;
  };

  type Recipe = {
    _id: string;
    title: string;
    description: string;
    servings: string;
    ingredients: Ingredient[];
    instructions: string[];
    tags: string[];
    categories: string[];
    prep_time: number;
    cook_time: number;
  };

  export let recipes: Recipe[];

  async function patch(res: Response) {
    const recipe: Recipe = await res.json();
    recipes = recipes.map((r) => {
      if (r._id === recipe._id) return recipe;
      return r;
    });
  }
</script>

<svelte:head>
  <title>Recipes</title>
</svelte:head>

<div class="recipes">
  <h1>Recipes</h1>

  <form
    class="new"
    action="/recipes.json"
    method="post"
    use:enhance={{
      result: async (res, form) => {
        const created = await res.json();
        recipes = [...recipes, created];

        form.reset();
      },
    }}
  >
    <input name="text" aria-label="Add recipe" placeholder="+ tap to add a recipe" />
  </form>
  {#each recipes as recipe (recipe._id)}
    <div class="recipe" transition:scale|local={{ start: 0.7 }} animate:flip={{ duration: 200 }}>
      <form
        class="text"
        action="/recipes/{recipe._id}.json?_method=patch"
        method="post"
        use:enhance={{
          result: patch,
        }}
      >
        <input
          aria-label="Edit recipe"
          type="description"
          name="text"
          value={`${recipe.title}: ${recipe.description}`}
        />
        <button class="save" aria-label="Save recipe" />
      </form>
    </div>
  {/each}
</div>

<style>
  .recipes {
    width: 100%;
    max-width: var(--column-width);
    margin: var(--column-margin-top) auto 0 auto;
    line-height: 1;
  }

  .new {
    margin: 0 0 0.5rem 0;
  }

  input {
    border: 1px solid transparent;
  }

  input:focus-visible {
    box-shadow: inset 1px 1px 6px rgba(0, 0, 0, 0.1);
    border: 1px solid #ff3e00 !important;
    outline: none;
  }

  .new input {
    font-size: 28px;
    width: 100%;
    padding: 0.5em 1em 0.3em 1em;
    box-sizing: border-box;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
    text-align: center;
  }

  .recipe {
    display: grid;
    grid-template-columns: 2rem 1fr 2rem;
    grid-gap: 0.5rem;
    align-items: center;
    margin: 0 0 0.5rem 0;
    padding: 0.5rem;
    background-color: white;
    border-radius: 8px;
    filter: drop-shadow(2px 4px 6px rgba(0, 0, 0, 0.1));
    transform: translate(-1px, -1px);
    transition: filter 0.2s, transform 0.2s;
  }

  form.text {
    position: relative;
    display: flex;
    align-items: center;
    flex: 1;
  }

  .recipe input {
    flex: 1;
    padding: 0.5em 2em 0.5em 0.8em;
    border-radius: 3px;
  }

  .recipe button {
    width: 2em;
    height: 2em;
    border: none;
    background-color: transparent;
    background-position: 50% 50%;
    background-repeat: no-repeat;
  }

  .save:focus {
    transition: opacity 0.2s;
    opacity: 1;
  }
</style>
