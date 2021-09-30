<script>
	import Robot from "./Robot.svelte";
	let newRobotName = "sven";
	let selectedRobot = 0;
	let money = 10000;
	let currentMovementDifficulty = 1;
	let currentResource = "Coal";
	let inventoryValue = 0;
	let robots = [];
	$: robots;
	buyThing("robot", null);
	let upgrades = [
		{ name: "HP_Capacity", value: [10, 25, 50, 100, 200, 500] },
		{ name: "Storage", value: [20, 50, 100, 200, 400, 1000] },
		{ name: "Damage", value: [1, 2, 5, 10, 20, 50] },
		{ name: "Mining_Speed", value: [2, 5, 10, 15, 20, 40] },
		{
			name: "Mining_Level",
			value: ["Coal", "Iron", "Gem", "Gold", "Platin", "ElementX"],
		},
		{ name: "Energy_Capacity", value: [20, 30, 40, 60, 100, 200] },
		{ name: "Energy_Regen", value: [4, 6, 8, 10, 15, 20] },
	];
	let upgradePrices = {
		upgrades: [0, 100, 300, 900, 2700, 8100],
		restock: { HP: 1, Energy: 1 },
	};
	$: actions = [
		{
			name: "Blocking",
			cost: 2,
			multiplicator: robots[selectedRobot].stats.HP_Capacity / 10,
		},
		{
			name: "Mining",
			cost: 1,
			multiplicator: robots[selectedRobot].upgrades.Mining_Level + 1,
		},
		{ name: "Moving", cost: 1, multiplicator: currentMovementDifficulty },
		{
			name: "Attacking",
			cost: 1,
			multiplicator: robots[selectedRobot].upgrades.Damage + 1,
		},
	];
	let resources = [
		{ name: "Coal", price: 1.5 },
		{ name: "Iron", price: 3 },
		{ name: "Gem", price: 6 },
		{ name: "Gold", price: 12 },
		{ name: "Platin", price: 24 },
	];
	let movementDifficulties = [
		{name: 'Light', cost: 1},
		{name: 'Medium', cost: 2},
		{name: 'Hard', cost: 3}
	]

	function calculateUpgradePriceForTier(tier) {
		return (
			upgradePrices.upgrades[tier] +
			(upgradePrices.upgrades[tier] *
				Math.pow(1.05, robots[selectedRobot].upgrades.count) -
				100)
		);
	}
	function buyThing(type, name) {
		console.log("they want to buy an", name, type);
		switch (type) {
			case "upgrade": {
				let price = calculateUpgradePriceForTier(
					robots[selectedRobot].upgrades[name] + 1
				);
				if (money - price >= 0) {
					money -= price;
					robots[selectedRobot].upgrades[name]++;
					robots[selectedRobot].upgrades.count += 1;
					let meantUpgrade = upgrades.filter(
						(upgrade) => upgrade.name === name
					)[0];
					let upgradeValue =
						meantUpgrade.value[
							robots[selectedRobot].upgrades[name]
						];
					robots[selectedRobot].stats[name] = upgradeValue;
				}
				return null;
			}
			case "restock": {
				let missing = getMissingFromResource(name);
				let price = missing * upgradePrices.restock[name];
				console.log("missing", missing, "will cost", price);
				if (money - price >= 0) {
					money -= price;
					robots[selectedRobot].stats[name] += missing;
				}
				return null;
			}
			case "robot": {
				robots.push({
					name: newRobotName,
					stats: {
						HP: 10,
						HP_Capacity: 10,
						Energy: 20,
						Energy_Capacity: 20,
						Storage: 20,
						Damage: 1,
						Mining_Speed: 2,
						Mining_Level: "Coal",
						Energy_Regen: 4,
					},
					inventory: {
						Coal: 0,
						Iron: 0,
						Gem: 0,
						Gold: 0,
						Platin: 0,
						ElementX: 0,
						count: 0
					},
					upgrades: {
						HP_Capacity: 0,
						Storage: 0,
						Damage: 0,
						Mining_Speed: 0,
						Mining_Level: 0,
						Energy_Capacity: 0,
						Energy_Regen: 0,
						count: 0,
					},
				});
				robots = robots;
				newRobotName = "";
				return null;
			}
			case "specials": {
			}
		}
	}
	//Maybe its not bad that an action even if it doesnt do anything still uses up energy
	function doAction(type) {
		if (type === "Mining") {
			let diff = robots[selectedRobot].stats.Storage - (robots[selectedRobot].inventory.count + robots[selectedRobot].stats.Mining_Speed);

			if(diff < 0) {
				console.log('its more than i can carry',)
				robots[selectedRobot].inventory[currentResource] += robots[selectedRobot].stats.Mining_Speed + diff;
				robots[selectedRobot].inventory.count += robots[selectedRobot].stats.Mining_Speed + diff;
			} else {
				robots[selectedRobot].inventory[currentResource] += robots[selectedRobot].stats.Mining_Speed;
				robots[selectedRobot].inventory.count += robots[selectedRobot].stats.Mining_Speed;
			}
		}
		console.log("i want to do a ", type, "action");
		let action = actions.filter((action) => action.name === type)[0];
		robots[selectedRobot].stats.Energy -=
			action.cost * action.multiplicator;
	}
	function changeSelectedRobot(id) {
		selectedRobot = id;
	}
	function changeResource(resource) {
		currentResource = resource;
	}
	function changeMovementDifficulty(difficulty) {
		currentMovementDifficulty = difficulty;
	}
	function getMissingFromResource(resource) {
		console.log(robots[selectedRobot].stats[resource + "_Capacity"]);
		return (
			robots[selectedRobot].stats[resource + "_Capacity"] -
			robots[selectedRobot].stats[resource]
		);
	}
	function sellInventory() {
		money += countInventoryValue();
	}
	function countInventoryValue() {
		let sum = 0;
		resources.forEach(elem => {
			sum += robots[selectedRobot].inventory[elem.name] * elem.price
		});
		return sum;
	}
</script>

<main>
	<div class='upper'>

	
	<div class="section">
		<div class='buysection'>

			{#each upgrades as upgrade, id}
			<div class='option'>
				<button on:click={() => buyThing("upgrade", upgrade.name)}
					>Upgrade {upgrade.name} to {upgrade.value[
						robots[selectedRobot].upgrades[upgrade.name] + 1
					]} for {Math.round(
						calculateUpgradePriceForTier(
							robots[selectedRobot].upgrades[upgrade.name] + 1
							)
							)}
				</button>
			</div>
		{/each}
	</div>

		<div class="buysection">
			<div class='option'>
				<button on:click={() => buyThing("restock", "HP")}>
					Replenish {robots[selectedRobot].name}'s HP for {upgradePrices
					.restock.HP *
					robots[selectedRobot].stats.HP_Capacity -
					robots[selectedRobot].stats.HP}</button
				>
			</div>
			<div class='option'>
				<button on:click={() => buyThing("restock", "Energy")}>
					Replenish {robots[selectedRobot].name}'s Energy for {upgradePrices
						.restock.Energy *
						robots[selectedRobot].stats.Energy_Capacity -
						robots[selectedRobot].stats.Energy}</button
				>
			</div>
		</div>
		<div class="buysection">
			<div class='option'>
				<input bind:value={newRobotName} placeholder="New Robot name" />
				<button on:click={() => buyThing("robot", null)}
					>Buy Robot {newRobotName}</button
				>
				<button on:click={sellInventory}>Sell Inventory for {inventoryValue}</button>
			</div>
		</div>
	</div>
	<div class='section'>
		<div class="buysection">
			{#each actions as action}
				<div class='option'>
					<button on:click={() => doAction(action.name)}>
						Do a {action.name} Action for {action.cost * action.multiplicator}
					</button>
				</div>
			{/each}
		</div>
		<div class="buysection">
			{#each resources as resource}
				<div class='option'>
					<button on:click={() => changeResource(resource.name)}>
						Change Resource to {resource.name}
					</button>
				</div>
			{/each}
		</div>
		<div class='buysection'>
			{#each movementDifficulties as difficulty}
				<div class='option'>
					<button on:click={() => {changeMovementDifficulty(difficulty.cost)}}>Change Tile to {difficulty.name}</button>
				</div>
			{/each}
		</div>
	</div>
	<div class='section'>
		<p>Inventory</p>
		<div class='inventory'>
			<div class='option'>
				Money: {money}
			</div>
			{#each resources as resource}
				<div class='option'>
					{resource.name}: {robots[selectedRobot].inventory[resource.name]}
				</div>
			{/each}
		</div>
	</div>
</div>
	<div class="robotinfowrapper">
		{#each robots as robot, id}
			<div class="robotwrapper" on:click={() => changeSelectedRobot(id)}>
				<Robot selected={selectedRobot === id ? true : false} {robot} />
			</div>
		{/each}
	</div>
	{money}
</main>

<style>
	main {
		overscroll-behavior-y: contain;
		background: url("images/background.jpg");
		background-size: cover;
		left: 0;
		top: 0;
		position: absolute;
		min-height: 100%;

		font-family: sans-serif;
		text-align: center;
		color: white;
	}
	.upper {
		display: flex;
		flex-direction: row;
		height: 50%;
		
	}
	@media(max-width: 960px) {
		.upper {
			flex-direction: column;
		}
	}
	.robotinfowrapper {
		display: flex;
		flex-direction: row;
		margin: 2rem;
		flex-wrap: wrap;
	}
	.robotwrapper {
		cursor: pointer;
	}
	.buysection {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: flex-start;
		margin: 2rem 0rem 2rem 0rem;
	}
	.inventory {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		margin: 2rem 0rem 2rem 0rem;
	}
	button {
		all: unset;
		margin: 0.5rem;
		border: solid white;
		border-width: 0px 4px 0px 4px;
		padding: 0.5rem;
		padding-top: 0.7rem;
		cursor: pointer;
		text-transform: uppercase;
  	}
	input {
		all: unset;
		border-bottom: 4px solid white;
  }
	button:hover {
    	transition: background 0.2s ease-in-out;
    	background: rgba(0, 0, 0, 0.2);
  	}
	.section {
		display: flex;
		flex-direction: column;
		-webkit-box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
		box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
		margin: 1rem;
		border-radius: 1rem;
		justify-content: center;
		flex-grow: 1;
  	}
	  .option {
		display: flex;
		flex-direction: column;
		margin: 1rem;
		-webkit-box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
		box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
		border-radius: 1rem;
		padding: 1rem;
		background: rgba(0, 0, 0, 0.2);
	  }
</style>
