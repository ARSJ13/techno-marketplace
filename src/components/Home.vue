<template>    
  <div>
    <section class="modal" v-if="produto">
      <div class="modal-container">
        <img :src="produto.img" :alt="produto.nome">
      </div>
      {{produto}}
    </section>
    <section class="produtos">
      <div v-for="(item, index) in produtos" @click="fetchModal(item.id)" :key="index" class="produto">
        <img :src="item.img" :alt="item.name" class="produto-img">
        <div class="produto-body">
          <span class="produto-price">{{ item.price | precoReal }}</span>
          <h2 class="produto-name">{{ item.name }}</h2>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: "Home",
  data(){
    return{
      produtos: [],
      produto: ""
    }
  },
  filters:{
    precoReal(valor){
      return valor.toLocaleString("pt-BR", {style: "currency", currency: "BRL"})
    }
  },
  methods:{
    fetchProdutos(){
      fetch('./api/produtos.json')
      .then(res => res.json())
      .then( res => {
        this.produtos = res;
      })
    },
    fetchModal(id){
      fetch(`./api/produtos/${id}/dados.json`)
      .then( res => res.json())
      .then( res =>{
        this.produto = res;
      })
    }
  },
  created(){
    this.fetchProdutos()
  }
}
</script>

<style scoped>
  .produtos{
    max-width: 900px;
    margin: 100px auto 0 auto;
  }
  .produto{
    display: flex;
    margin-bottom: 3rem;
    align-items: center;
    background-color: #fff;
    box-shadow: 0 0 2rem rgba(0, 0, 0, 0.1);
    cursor: pointer;
  }
  .produto-img{
    max-width: 300px;
    margin-right: 2rem;
  }
  .produto-name{
    font-size: 2.5rem;
    line-height: 1;
  }
  .produto-price{
    color: rgba(0, 0, 0, 0.5);
  }
</style>