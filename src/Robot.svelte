<script>
    export let robot;
  
    export let selected;
  </script>
  
  <div class='robotinfo' class:selected="{selected === true}">
    <p>{robot.name}</p>
    <p>HP: {robot.stats.HP}/{robot.stats.HP_Capacity}</p>
    <p>Energy: {robot.stats.Energy}/{robot.stats.Energy_Capacity}</p>
    <p>Storage: {robot.inventory.count}/{robot.stats.Storage}</p>
    <p>Damage: {robot.stats.Damage}</p>
    <p>Mining up to {robot.stats.Mining_Level} at {robot.stats.Mining_Speed}/round
    <p>Regenerates {robot.stats.Energy_Regen} per action</p>
    <p>Upgrades: {robot.upgrades.count}</p>
  </div>
  
  <style>
    .robotinfo {
      transition: background 0.2s ease-in-out;
      border-radius: 1rem;
      background: rgba(0, 0, 0, 0);
      margin: 1rem;
      padding: 0.5rem;
      -webkit-box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
      box-shadow: 5px 5px 23px -1px rgba(0, 0, 0, 0.42);
    }
    .robotinfo:hover {
      transition: background 0.2s ease-in-out;
      background: rgba(0, 0, 0, 0.2);
    }
    .selected {
      background: rgba(0, 0, 0, 0.2);
    }
  </style>