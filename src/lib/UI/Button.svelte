<script>
  export let text = "Click me";
  export let onClick = () => {};
</script>

<button on:click={onClick}>
  {text}
</button>

<style>
button {
  background: linear-gradient(45deg, #e74c3c, #f1c40f);
  border-radius: 12px;
  padding: 0.8rem 2rem;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 5px 15px rgba(231, 76, 60, 0.6);
}
button:hover { 
  transform: scale(1.05); 
  box-shadow: 0 8px 20px rgba(231,76,60,0.8);
}
button:active {
  transform: scale(0.95);
}
</style>
