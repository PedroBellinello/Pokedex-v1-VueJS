<template>
  <v-app>
    <v-containter>
      <v-container>
        <v-text-field
        v-model="search"
        label='Pesquisar'
        placeholder="Charmander"
        solo>
        </v-text-field>
         <v-row>
          <v-col cols="4" v-for="pokemon in filtered_pokemons" :key="pokemon.name">

            <v-card @click="show_pokemon(get_id(pokemon))">

              <v-container>
                <v-row class="mx-0 d-flex justify-center">
                 <img :src='`http://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(pokemon)}.png`'
                 :alt="pokemon.name" width="80%"/>
                </v-row>
                <h2 class="text-center">{{ get_name(pokemon.name)}}</h2>
              </v-container>
            </v-card>
         </v-col>
         </v-row>
      </v-container>
   </v-containter>

   <v-dialog v-model="show_dialog" width="800">
      <v-card v-if="selected_pokemon" class="px-4">
        <v-container>             
          <v-row class="d-flex align-center ">
              <v-col cols="4"> 
                    <img :src="`http://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`"
                    :alt="selected_pokemon.name" width="80%"/>
              </v-col>
              <v-col cols="8"> 
                <h1>{{get_name(selected_pokemon.name)}}</h1>
                <v-chip  v-for="type in selected_pokemon.types" :key="type.slot" class="mr-2">{{type.type.name}} </v-chip>
                <v-divider class="my-4"></v-divider>
                <v-chip>
                  Altura: {{selected_pokemon.height * 2.54 }} Cm
                </v-chip>
                <v-chip class="ml-2">
                  Peso: {{ (selected_pokemon.weight * 0.45359237).toFixed(0)}} Kgs
                </v-chip>
              </v-col>
            </v-row>
            <h2>Stats</h2>
            <v-simple-table>
               <template>
                 <thead>
                   <tr class="text-left">
                     <td>
                      HP: 
                     {{this.selected_pokemon.stats[0].base_stat}}
                     </td> 
                     </tr>
                     <tr class="text-left">
                     <td>
                      ATK: 
                     {{this.selected_pokemon.stats[5].base_stat}}
                     </td> 
                     </tr>
                     <tr class="text-left">
                     <td>
                      DEF: 
                     {{this.selected_pokemon.stats[4].base_stat}}
                     </td> 
                     </tr>
                     <h2>Moves</h2>
                     <tr>
                     <th class="text-left">
                       Level
                     </th>
                     <th class="text-left">
                      Name
                     </th>
                   </tr>
                 </thead>
                 
                 <tbody>
                   <tr v-for="item in filter_moves(selected_pokemon)" :key="item.move.name">
                     <td>{{get_move_level(item)}}</td>
                     <td>{{ get_name(item.move.name)}}</td>
                   </tr>
                 </tbody>
               </template>
             </v-simple-table>
        </v-container>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from 'axios'
//import version from 'os'

export default {
  name: 'App',

  components: {},

//se
 data ()  {
  return {
    pokemon: [],
    search: "",
    show_dialog: false,
    selected_pokemon: null,
  }
 },



//ao ser aberto no browser  esse código executa
  mounted () {
    axios
      .get("https://pokeapi.co/api/v2/pokemon?limit=459")
      .then((response) => {
        this.pokemon =  response.data.results;
      });
  },
  
  methods: {
    //pegar id
    get_id(pokemon){
      return Number(pokemon.url.split("/")[6]);
    },
    //pegar nome
    get_name(text) {
      return text.charAt(0).toUpperCase() + text.slice(1).replace("-", " ")
    },
    //mostrar pokemon por id
    show_pokemon(id){
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`)
      .then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = !this.show_dialog
      })
    },
    //filtrar level de habilidades
    get_move_level(move){
     for(let version of move.version_group_details){
      //version.version_group.name == "sword-shield" 
       if(version.move_learn_method.name == "level-up"){
        return version.level_learned_at;
      }
     }
    },
    //filtrar habilidades
    filter_moves(pokemon){
      return pokemon.moves.filter((item) =>{
        var include =  false;
        for (let version of item.version_group_details){
          //version.version_group.name == "sword-shield" &&
          if( version.move_learn_method.name == "level-up"){
            include = true
          }
        }
         return include;
      })
    },
  },

  //filtrar pokemons
  computed: {
    filtered_pokemons() {
     return this.pokemon.filter((item) => {
        return item.name.includes(this.search)
      })
    }
  }

}
</script>

<style>
#app {
  background: linear-gradient(
      to bottom right,
      rgb(134, 116, 116, 1),
      rgb(54, 57, 59, 1)
    )
    no-repeat center center fixed !important;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;
}
</style>
