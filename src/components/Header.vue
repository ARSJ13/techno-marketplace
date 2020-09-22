<template>
  <header class="header">
    <img class="logo" src="../assets/techno.svg" alt="Techno">
    <div class="carrinho-menu">{{ count }} |</div>
  </header>
</template>

<script>

export default {
  name: "Header",
  props: ['count']
}
</script>

<style scoped>
  .header{
    display: flex;
    justify-content: space-between;
    max-width: 900px;
    margin: 0 auto;
    padding: 20px 0;
  }
  .logo{
    width: 5rem;
  }
  .carrinho-menu::after{
    content: "";
    display: inline-block;
    background: url("../assets/carrinho.svg");
    width: 25px;
    height: 25px;
    margin-left: 10px;
  }
  .carrinho-menu{
    display: flex;
    align-items: center;
    cursor: pointer;
    font-family: Arial, Helvetica, sans-serif;
  }
</style>
