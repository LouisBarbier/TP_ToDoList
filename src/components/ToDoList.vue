<script setup>
// -- import de la fonction permettant de déclarer
//   une variable comme une varianel d'ETAT
import { reactive } from "vue";
import { onMounted } from "vue";
// -- les 2 sous composants utilisés
import ToDoListItem from "./ToDoListItem.vue";
import ToDoForm from "./ToDoForm.vue";

// -- la classe Chose
import Chose from "../Chose";

// -- la liste des choses --> dans le state
// --> donnée réactive = l'affichage sera actualisée
//      automatiquement à chque cght dans la liste
const url = "https://webmmi.iut-tlse3.fr/~pecatte/todos/public/1/todos";
let listeC = reactive([]);

function getToDos(){
  const fetchOptions={method: "GET"}
  fetch(url,fetchOptions)
      .then((response) => {
        return response.json();
      })
      .then((dataJSON) => {
        console.log(dataJSON);
        listeC.splice(0,listeC.length)
        dataJSON.forEach((v) => listeC.push(new Chose (v.id,v.libelle,v.fait)));
      })
      .catch((error) =>
        console.log(error)
      )
}

onMounted(()=>{
  getToDos()
})

// -- handler pour 'faire/défaire' une chose à partir de l'index dans la liste
function handlerFaire(chose) {
  chose.faire();
  const head=new Headers()
  head.append("Content-Type", "application/json");
  const fetchOptions={method: "PUT",
    headers: head,
    body: JSON.stringify(chose)}
  fetch(url,fetchOptions)
      .then((response) => {
        return response.json();
      })
      .then((dataJSON) => {
        getToDos()
      })
      .catch((error) =>
          console.log(error)
      )
}
// -- handle pour supprimer une chose à partir de l'index dans la liste
function handlerDelete(idx) {
  const fetchOptions={method: "DELETE"}
  fetch(url+"/"+listeC[idx].id,fetchOptions)
      .then((response) => {
        return response.json();
      })
      .then((dataJSON) => {
        getToDos()
      })
      .catch((error) =>
          console.log(error)
      )
}
// -- handler pour ajouter une nouvelle chose à partir
//    du libelle saisi dans le formulaire
//     qui se retrouve en paramétre
function handlerAdd(libelle) {
  const head=new Headers()
  head.append("Content-Type", "application/json")
  const fetchOptions={method: "POST",
    headers: head,
    body: JSON.stringify({ libelle : libelle })}
  fetch(url,fetchOptions)
      .then((response) => {
        return response.json();
      })
      .then((dataJSON) => {
        getToDos()
      })
      .catch((error) =>
          console.log(error)
      )
}
</script>

<template>
  <h3>Liste des choses à faire</h3>
  <!-- Utiliser le composant "todoform" pour saisir le texte de la chose 
       il faut écouter l'event "addc" émis par la composant pour prévenir
      que la saisie est terminée  -->
  <ToDoForm @addc="handlerAdd"></ToDoForm>
  <ul>
    <!-- 
      le composant todoitem sert à afficher une chose ;
      en paramètre d'entrée, il a besoin de 2 elts appelés "props"
         - la chose à afficher et l'index dans le tableau listeC
      il comporte 2 boutons pour faire / supprimer
      il emet un event quand le bouton est cliqué
        "deletec" pour la supression, "fairec" pour faire/défaire
    -->
    <ToDoListItem
      v-for="(chose, index) of listeC"
      :key="chose.id"
      :chose="chose"
      :indexc="index"
      @deletec="handlerDelete"
      @fairec="handlerFaire"
    />

    <!-- afficher la liste sans le composant "todolistitem"
    <li v-for="(chose, index) of listeC" :key="chose.id">
      {{ chose.pourAfficher() }}
      <button @click="handlerDelete(index)">Supprimer</button>
      <button @click="handlerFaire(index)">Faire</button>
    </li>
    -->
  </ul>
</template>

<style scoped>
</style>
