<script>
  export let intensity = 0;
</script>

<div class="fire-bar">
  <div class="fire" style="width: {intensity}%"></div>
</div>

<style>
.fire-bar {
  width: 250px;
  height: 25px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: inset 0 0 5px rgba(255,255,255,0.2);
  margin-top: 1rem;
}

.fire {
  height: 100%;
  background: linear-gradient(90deg, #f1c40f, #e67e22, #e74c3c);
  transition: width 0.1s linear;
  box-shadow: 0 0 10px rgba(231,76,60,0.6);
  border-radius: 12px;
}
</style>
