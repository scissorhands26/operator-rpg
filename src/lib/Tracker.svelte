<script lang="ts">
  import { each } from "svelte/internal";
  import { writable } from "svelte/store";
  import Player from "./Player.svelte";

  let dataStore: any = writable({});

  let storeVisible = false;
  let skillsVisible = true;

  function changeSkill(skillName: string, amount: number) {
    dataStore.update((data) => {
      const skill = data.user.skills.find((s) => s.name === skillName);
      if (skill) {
        skill.currentXP += amount;

        if (skill.currentXP === 10) {
          skill.currentLevel = skill.currentLevel + 1;
          skill.currentXP = 0;
        }
      }
      return data;
    });
  }

  function decreaseHealth() {
    if ($dataStore.user.info.currentHealth > 0) {
      $dataStore.user.info.currentHealth--;
    }
  }

  function respawn() {
    alert("Needs to be implemented!");
  }

  function saveData(data: any) {
    const a = document.createElement("a");
    const timestamp = new Date().getTime();
    a.href = URL.createObjectURL(
      new Blob([JSON.stringify(data, null, 2)], {
        type: "application/json",
      })
    );
    a.download = `data-${timestamp}.json`;
    a.click();
  }

  async function loadData() {
    const fileInput = document.createElement("input");
    fileInput.type = "file";
    fileInput.accept = ".json";
    fileInput.addEventListener("change", async (event) => {
      const file = (event.target as HTMLInputElement).files?.[0];
      if (!file) return;
      const text = await file.text();
      const data = JSON.parse(text);
      dataStore.set(data);
      console.log($dataStore);
      $dataStore.user.items.forEach((item) => {
        if (item.type === "armor") {
          $dataStore.user.info.maxHealth =
            $dataStore.user.info.maxHealth + item.value;
          $dataStore.user.info.currentHealth = $dataStore.user.info.maxHealth;
        }
      });
    });
    fileInput.click();
  }

  function buyItem(item) {
    console.log(item.cost);
    if ($dataStore.user.info.gold >= item.cost) {
      $dataStore.user.info.gold -= item.cost;
      $dataStore.user.items.push(item);
    } else {
      alert("You're broke!");
    }
  }

  function showStore() {
    skillsVisible = false;
    storeVisible = true;
  }
  function showSkills() {
    storeVisible = false;
    skillsVisible = true;
  }
</script>

{#if Object.keys($dataStore).length === 0}
  <button on:click={() => loadData()}>Load</button>
{:else if storeVisible === false}
  <div>
    <div class="load-save-buttons">
      <button on:click={() => loadData()}>Load</button>
      <button on:click={showStore}>Store</button>
      <button on:click={() => saveData($dataStore)}>Save</button>
    </div>
    <div class="container">
      <div class="traits-container">
        {#each $dataStore.user.skills as skill}
          <div class="trait-item">
            <div>{skill.name}</div>
            <div>LVL: {skill.currentLevel}</div>
            <div class="skill-buttons">
              <button on:click={() => changeSkill(skill.name, -1)}>-</button>
              <span
                class="skill-progress"
                style="--currentXP: {skill.currentXP}"
              />
              <button on:click={() => changeSkill(skill.name, 1)}>+</button>
            </div>
          </div>
        {/each}
      </div>
      <div class="player-container">
        <div>
          <h2>
            {$dataStore.user.info.rank}
            {$dataStore.user.info.name}, {$dataStore.user.info.cert}
          </h2>
          <div>
            <div>
              <span>
                Gold: {#each Array.from( { length: $dataStore.user.info.gold } ) as _, index}
                  <span class="gold-coins">🪙</span>
                {/each}</span
              >
            </div>
            <span>Health:</span>
            <span class="hearts">
              {#each Array.from( { length: $dataStore.user.info.maxHealth } ) as _, index}
                {#if index < $dataStore.user.info.currentHealth}
                  ❤️
                {:else}
                  🖤
                {/if}
              {/each}
            </span>
            <div>
              {#if $dataStore.user.info.currentHealth === 0}<button
                  on:click={respawn}>Respawn</button
                >{:else}<button on:click={decreaseHealth}>Take Damage</button
                >{/if}
            </div>
          </div>
        </div>
        <div class="player-object">
          <Player />
        </div>
        <div>
          <span>Inventory</span>
          <div class="inventory-container">
            {#each $dataStore.user.items as item}
              <div class="inventory-item">{item.name}</div>
            {/each}
          </div>
        </div>
        <div class="player-damage">
          <span>Completed Ops: {$dataStore.user.info.ops}</span>
        </div>
      </div>
    </div>
  </div>
{:else}
  <div>
    <div class="load-save-buttons">
      <button on:click={() => loadData()}>Load</button>
      <button on:click={showSkills}>Skills</button>
      <button on:click={() => saveData($dataStore)}>Save</button>
    </div>

    <div class="container">
      <div class="store-container">
        {#each $dataStore.game.store as item}
          <div class="store-item">
            <div>{item.name}</div>
            <div>{item.type}</div>
            <div>
              Price: {item.cost}
            </div>
            <button on:click={() => buyItem(item)}>Buy</button>
          </div>
        {/each}
      </div>

      <div class="player-container">
        <div>
          <h2>
            {$dataStore.user.info.rank}
            {$dataStore.user.info.name}, {$dataStore.user.info.cert}
          </h2>
          <div>
            <div>
              <span>
                Gold: {#each Array.from( { length: $dataStore.user.info.gold } ) as _, index}
                  <span class="gold-coins">🪙</span>
                {/each}</span
              >
            </div>
            <span>Health:</span>
            <span class="hearts">
              {#each Array.from( { length: $dataStore.user.info.maxHealth } ) as _, index}
                {#if index < $dataStore.user.info.currentHealth}
                  ❤️
                {:else}
                  🖤
                {/if}
              {/each}
            </span>
            <div>
              {#if $dataStore.user.info.currentHealth === 0}<button
                  on:click={respawn}>Respawn</button
                >{:else}<button on:click={decreaseHealth}>Take Damage</button
                >{/if}
            </div>
          </div>
        </div>
        <div class="player-object">
          <Player />
        </div>

        <div>
          <span>Inventory</span>
          <div class="inventory-container">
            {#each $dataStore.user.items as item}
              <div class="inventory-item">{item.name}</div>
            {/each}
          </div>
        </div>

        <div class="player-damage">
          <span>Completed Ops: {$dataStore.user.info.ops}</span>
        </div>
      </div>
    </div>
  </div>
{/if}

<style>
  .container {
    display: flex;
    flex-direction: row;
  }
  .load-save-buttons {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }
  .gold-coins:not(:first-child) {
    margin-left: -20px;
  }
  .store-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    gap: 10px;
    border: 0.5em solid black;
    height: 75vh;
    padding: 10px;
  }
  .store-item {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 5px;
    text-align: center;
    border: 2px solid black;
  }
  .traits-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    gap: 10px;
    border: 0.5em solid black;
    height: 75vh;
    padding: 10px;
  }
  .trait-item {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 0;
    text-align: center;
    border: 2px solid black;
  }
  .player-container {
    display: flex;
    flex-direction: column;
    border: 0.5em solid black;
    justify-content: space-between;
    height: 75vh;
    padding: 10px;
    margin-left: -5px;
  }
  .inventory-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    gap: 10px;
    border: 0.5em solid black;
    padding: 10px;
  }
  .inventory-item {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 0;
    text-align: center;
    border: 2px solid black;
  }
  .player-object {
    display: flex;
    justify-content: center;
  }
  .hearts {
    width: 240px;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
  }
  .skill-buttons {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }
  .skill-buttons button {
    background-color: #fff;
    border: 2px solid #000;
    color: #000;
    font-size: 1.2em;
    padding: 0.2em 0.4em;
    cursor: pointer;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 5px;
  }
  .skill-buttons button:hover {
    background-color: #000;
    color: #fff;
  }
  .skill-progress {
    background-color: #c8e6c9;
    border: 2px solid #000;
    border-radius: 4px;
    height: 26px;
    position: relative;
    width: 100px;
  }
  .skill-progress::before {
    border-radius: 4px;
    background-color: #4caf50;
    content: "";
    display: block;
    height: 100%;
    left: 0;
    position: absolute;
    top: 0;
    transition: width 0.2s ease-in-out;
    width: calc((var(--currentXP) / 10) * 100%);
  }
</style>
