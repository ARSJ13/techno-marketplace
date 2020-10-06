<template>    
  <div>
    <div @click="carActive=true">
      <Header :priceTotal="priceTotal | precoReal" :count="carrinho.length"></Header>
    </div>
    <section class="modal" v-if="produto" @click="closeModal">
      <div class="modal-container">
        <div class="modal-img">
          <img :src="produto.img" :alt="produto.nome">
        </div>
        <div class="modal-body">
          <button @click="produto = false" class="modal-button-close">X</button>
          <span class="modal-price">{{ produto.preco | precoReal }}</span>
          <h2 class="modal-title">{{ produto.nome }}</h2>
          <p class="modal-description">{{ produto.descricao }}</p>
          <button v-if="produto.estoque > 0" class="modal-btn" @click="adicionarItem">Adicionar Item</button>
          <button v-else class="modal-btn esgotado" disabled>Estoque esgotado!</button>
        </div>
        <div class="modal-review">
          <h2 class="modal-review-subtitle">Avaliações</h2>
          <ul>
            <li v-for="(review, id) in produto.avaliacoes" :key="id" class="review-item">
              <p class="review-description">{{ review.descricao }}</p>
              <p class="review-user">{{ review.nome }} | {{ review.estrelas }} estrelas</p>
            </li>
          </ul>
        </div>
      </div>
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

    <div class="alert-container" :class="{active : alertActive}">
      <p class="alert-msg">{{ alertMsg }}</p>
    </div>

    <section class="car-modal" :class="{ativo: carActive}" @click="closeCarModal">
      <div class="car-container">
        <button class="car-close" @click="carActive = false">X</button>
        <h2 class="car-title">Carrinho</h2>
        <div v-if="carrinho.length>=1">
          <ul class="car-list">
            <li v-for="(item, index) in carrinho" :key="index" class="car-item">
              <p> {{ item.nome }} </p>
              <p class="car-price"> {{ item.preco | precoReal }} </p>
              <button class="car-remove" @click="deleteItem(index)">X</button>
            </li>
          </ul>
          <p class="car-total">{{ priceTotal | precoReal }}</p>
          <button class="car-submit">Finalizar Compra</button>
        </div>
        <p v-else>O carrinho está vazio!</p>
      </div>
    </section>

  </div>
</template>

<script>

import Header from './Header'

export default {
  name: "Home",
  components:{
    Header
  },
  data(){
    return{
      produtos: [],
      produto: "",
      count: 0,
      carrinho: [],
      carActive: false,
      alertMsg: "Item adicionado",
      alertActive: false
    }
  },
  filters:{
    precoReal(valor){
      return valor.toLocaleString("pt-BR", {style: "currency", currency: "BRL"})
    }
  },
  computed:{
    priceTotal(){
      let total = 0
      if(this.carrinho.length){
        this.carrinho.forEach( item =>{
          total += item.preco;
        })
      }
      return total
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
      window.scrollTo({
        top: 0
      })
    },
    closeModal({target, currentTarget}){
      if (target === currentTarget) {
        this.produto = false;
      }
    },
    closeCarModal({target, currentTarget}){
      if (target === currentTarget) {
        this.carActive = false;
      }
    },
    adicionarItem(){
      this.produto.estoque--;
      const {id, nome, preco} = this.produto;
      this.carrinho.push({id, nome, preco});
      this.alert(`${nome} adicionado ao carrinho`);
    },
    deleteItem(id){
      this.carrinho.splice(id, 1);
    },
    checkLocalStorage(){
      if(window.localStorage.carrinho){
        this.carrinho = JSON.parse(window.localStorage.carrinho);
      }
    },
    compararEstoque(){
      const items = this.carrinho.filter(item => {
        if (item.id === this.produto.id) {
          return true;
        }
      })
      this.produto.estoque = this.produto.estoque - items.length;
    },
    alert(msg){
      this.alertActive = true
      this.alertMsg = msg;
      setTimeout(()=>{
        this.alertActive = false
      }, 2000)
    },
    router(){
      const hash = document.location.hash;
      if(hash){
        this.fetchProduto(hash.replace("#",""));
      }
    }
  },
  watch:{
    produto(){
      document.title = this.produto.nome || "Techno";
      const hash = this.produto.id || "";
      history.pushState(null, null, `#${hash}`);
      if (this.produto) {
        this.compararEstoque()
      }
    },
    carrinho(){
      window.localStorage.carrinho = JSON.stringify(this.carrinho);
    }
  },
  created(){
    this.fetchProdutos(),
    this.checkLocalStorage()
  }
}
</script>

<style scoped>
/* MODAL CAR */
.car-modal::before{
  content: "";
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
}
.car-modal{
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  padding: 20px;
  display: none;
}
.car-modal.ativo{
  display: block;
}
.car-close{
  border-radius: 50%;
  border: 2px solid #000;
  width: 40px;
  height: 40px;
  position: absolute;
  top: -10px;
  right: -10px;
  font-size: 1rem;
  background: #fff;
  font-family: Arial, Helvetica, sans-serif;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.7);  
}
.car-close:hover{
  background: #000;
  color: #fff;
  cursor: pointer;
  outline: none;
  transition: ease-in-out 200ms;
}
.car-container{
  position: relative;
  margin: 80px auto;
  background: #fff;
  padding: 40px;
  max-width: 800px;
  animation: fadeInDown 300ms forwards;
}
.car-item{
  display: grid;
  grid-template-columns: 1fr 1fr 50px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  padding: 15px 0;
}
.car-title{
  margin-bottom: 10px;
  border-bottom: 2px solid #000;
  font-size: 1.5rem;
  padding-bottom: 10px;
}
.car-remove{
  border: none;
  font-size: 1rem;
  cursor: pointer;
  outline: none;
  background: #fff;
  font-family: Arial, Helvetica, sans-serif;
}
.car-price{
  display: flex;
  justify-content: flex-end;
  align-items: center;
  font-family: Arial, Helvetica, sans-serif;
}
.car-total{
  display: flex;
  justify-content: flex-end;
  padding-right: 50px;
  margin: 20px 0;
  font-family: Arial, Helvetica, sans-serif;
}
.car-submit{
  display: block;
  margin-left: auto;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
  outline: none;
}


/* MODAL PRODUTO */
  .modal::before{
    content: "";
    position: fixed;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.5);
  }
  .modal{
    position: absolute;
    top: 0px;
    left: 0px;
    padding: 80px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .modal-container{
    position: relative;
    background: linear-gradient(to right, transparent 250px, #fff 250px);
    z-index: 1;
    display: grid;
    align-items: end;
    gap: 2rem;
    padding: 50px 50px 50px 0px;
    animation: fadeIn 300ms ease-out;
  }

  @keyframes fadeIn{
    from{
      opacity: 0;
      transform: translate3d(50px, 0, 0);
    }
    to{
      opacity: 1;
      transform: translate3d(0px, 0, 0);
    }
  }

  .modal-button-close{
    border-radius: 50%;
    border: 2px solid #000;
    width: 40px;
    height: 40px;
    position: absolute;
    top: -10px;
    right: -10px;
    font-size: 1rem;
    background: #fff;
    font-family: Arial, Helvetica, sans-serif;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.7);
  }
  .modal-button-close:hover{
    background: #000;
    color: #fff;
    cursor: pointer;
    outline: none;
    transition: ease-in-out 200ms;
  }
  .modal-img{
    grid-column: 1;
    box-shadow: 0px 3px 4px rgba(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);
  }
  .modal-img>img{
    max-width: 300px;
    display: block;
  }
  .modal-body{
    max-width: 600px;
    grid-column: 2;
  }
  .modal-title{
    font-size: 3rem;
  }
  .modal-btn{
    margin-top: 80px;
    background: #000;
    cursor: pointer;
    color: #fff;
    border: none;
    font-size: 1rem;
    padding: 0.8rem 1.5rem;
  }
  .modal-btn:hover{
    background: rgba(0, 0, 0, 0.9);
  }
  .modal-btn:active{
    background: #808080;
  }
  .esgotado{
    opacity: 0.6;
  }
  .esgotado:hover{
    background: #000;
  }
  .modal-review{
    grid-column: 2;
  }
  .modal-review-subtitle{
    font-size: 1.75rem;
  }
  .review-item{
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    padding-bottom: 20px;
  }
  .review-description{
    margin: 20px 0px 5px 0px;
    color: rgba(0, 0, 0, 0.7);
  }
  .review-user{
    font-weight: bold;
  }

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

  /* ALERTA */

  .alert-container{
    position: absolute;
    top: 20px;
    left: 0px;
    z-index: 10;
    width: 100%;
    text-align: center;
    display: none;
  }

  .alert-container.active{
    display: block;
    animation: fadeInDown 300ms forwards;
  }

  @keyframes fadeInDown {
    from {
      opacity: 0;
      transform: translate3d(0, -30px, 0);
    }
    to {
      opacity: 1;
      transform: translate3d(0, 0, 0);
    }
  }

  .alert-msg{
    background: #fff;
    display: inline-block;
    padding: 20px 40px;
    border: 2px solid #000;
    font-weight: bold;
    box-shadow: 0px 3px 4px rgba(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);

  }
</style>