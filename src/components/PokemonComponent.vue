 <template>
  <div >
 

  <div class="pokemon-page-wrapper">
  <header id="header" class="fixed-top">
    <div class="container">

      <div class="logo " style="float: left;">
        <h1 class="text-light"><a href="#header"><span>Pokedox</span></a></h1>
      </div>

      <nav class="main-nav">
        <ul>
          <li ><a href="#intro">Home</a></li>
           <li><a href="#about">About Us</a></li>
           <li class="active"><a href="#about">Pokemons</a></li>
          
        </ul>
      </nav>

    </div>
</header> 

<section class="pokemon-wrapper">
 <div class="container-fluid">
  <div class="row">
    <div class="col-md-3"></div>
     <div class="col-md-4"> 
      <input type="text" class="form-control" placeholder="Search Pokemon..." v-model="filter_pokemon">
    </div>
     <div class="col-md-2">
       <select class="form-control" 
        v-model.number="search_pokemon">
         <option value="1" >Name</option>
         <option value="4" >Color</option>
        <option value="3">Gender</option> 
        <option value="2">Habitat</option> 
      </select>
     </div>
      <div class="col-md-12">
        <div id="poke_container" class="poke-container">
          <div class="pokemon" v-for="(v,i) in pokemon_filter_list" :key="v.id" :style="{ backgroundColor: v.background }"  
            @click="openModal(v.id,v.background)">
            <div class="img-container">
               <img :src="v.sprites">
            </div>
            <div class="info">
                <span class="number">#{{v.id}}</span>
                <h4 class="name text-capitalize" >{{v.name}}</h4>
            </div>     
          </div> 
        </div>
      </div>
    </div>
     <!-- Modal -->
    <div id="myModal" class="modal fade" role="dialog">
      <div class="modal-dialog" style="width: 35vw;"  >

        <!-- Modal content-->
        <div class="modal-content" id="pokemonModal" >
          <div v-if="pokemon_info.length != 0">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal">&times;</button>
            <div class="pokemon_image" style="float: ;">
                <img :src="pokemon_info.data.sprites.front_default" alt="">
              </div>
            <div class="modal-title"  style="float: ;">
              <p class=" text-uppercase bold " style="padding-left:7px;">{{pokemon_info.data.name}} &nbsp; &nbsp; #{{pokemon_info.data.id}}</p>
                <span v-for="t in pokemon_info.data.types" class="types" style="background: #5186c7;">{{t.type.name}}</span>
            </div>
             
          </div>
          <div class="modal-body" >
            <ul class="pokemon_info_list">
               <li  @click="activeList=5 " :class="activeList == 5 ?'active' :'' ">
                <a href="javascript:;" >About
                </a>
              </li>
              <li  @click="activeList=1 " :class="activeList == 1 ?'active' :'' ">
                <a href="javascript:;" >Profile
                </a>
              </li>
               <li @click="activeList=2"  :class="activeList == 2 ?'active' :''" >
                <a href="javascript:;"  >Base Stats</a>
              </li>
               <li @click="activeList=3"  :class="activeList == 3 ?'active' :''" >
                <a href="javascript:;">Evolution</a>
              </li>
               <li @click="activeList=4"  :class="activeList ==4 ?'active' :''" >
                <a href="javascript:;">Moves</a>
              </li>
            </ul>
            <div class="pokemon_info_list_tabs" >
              <div  v-if="activeList == 5" >
               <p >
                
                 {{pokemon_species.flavor_text_entries[1].flavor_text}}
               </p>
              </div>
              <div  v-if="activeList == 1" >
                <table width="100%"  style="margin-left:10px;">
                  <tr>
                    <td width="40%">Height </td>
                     <td width="15%">:</td>
                    <td width="45%">{{pokemon_info.data.height}}</td>
                  </tr>
                  <tr>
                    <td>Weight </td>
                    <td>:</td>
                    <td>{{pokemon_info.data.weight}}</td>
                  </tr>
                   <tr>
                    <td >Species </td>
                    <td>:</td>
                    <td ></td>
                  </tr>
                   <tr>
                    <td>Abilities</td>
                    <td>:</td>
                    <td>{{pokemon_info.abilities}}</td>
                  </tr> 
                  <tr>
                    <td>Base Experience</td>
                    <td>:</td>
                    <td>{{pokemon_info.data.base_experience}}</td>
                  </tr>
                </table>
              </div>
              <div  v-if="activeList == 2" >
                <div class="content-stat" >
                  <p>Base Stats Total <span style="padding: 10px 15px;
                      background: #6989b0;
                      margin: 5px;
                      color: white;
                      border-radius: 25px;
                      font-size: 12px;">{{base_stat_total}}</span></p>
                 
                </div>
               <table width="100%" style="margin-left:10px;">
                  <tbody>
                    <tr v-for="stats in pokemon_info.data.stats">
                      <td width="40%">{{stats.stat.name}}</td>
                      <td width="15%">:</td>
                      <td width="45%">
                       <span
                          v-for="circle in 6"
                          :class="{ 'is-filled': calculateCircles(stats.base_stat) >= circle }" 
                          class="content-stat-bar-item"
                        />
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
               <div  v-if="activeList == 3" >
                    <span v-for="m in pokemon_info.data.moves" style="">
                      {{m.move.name}}
                    </span>
              </div>

            </div>
          </div>
          </div>
        </div>
      </div>
    </div>
  </div>
 </section>
  </div>
  </div>
</template>

<script>
import axios from "axios";


export default {
  data() {
    return {
      pokemon: [],
      filter_pokemon:'',
      search_pokemon:1,
      loading: true,
      errored: false,
      currentstep:1,
      pokemon_info:[],
      pokemon_species:[],
      activeList:1,

    };
  },


 created() {
   this.getApi();
  },
 
  methods: {
    getApi(){
      const pokemon_number =150;
      var arr =[];
      for(let i=1; i<=pokemon_number; i++){
        let urlOne = `https://pokeapi.co/api/v2/pokemon/${i}/`;
        let urlTwo = `https://pokeapi.co/api/v2/pokemon-species/${i}`;
         let urlThree = `https://pokeapi.co/api/v2/gender/${i}/`;
        const requestOne = axios.get(urlOne);
        const requestTwo = axios.get(urlTwo);
        const requestThree = axios.get(urlThree);

        axios.all([requestOne, requestTwo,requestThree]).then(axios.spread((...responses) => {
            const responseOne = responses[0].data;
            const responseTwo = responses[1].data;
            let obje_to_push ={
              name :responseOne.name,
              id :responseOne.id,
              sprites:responseOne.sprites.front_default, 
              color :responseTwo.color.name,
              background :responseTwo.color.name,
              habitat :responseTwo.habitat.name,
            }
            this.pokemon.push(obje_to_push);

          $(this.pokemon).each(function(i, pokemon){
            switch (pokemon.background) {
              case 'green':
                pokemon.background = "rgb(152, 215, 165)";
                break;
                  case 'brown':
                pokemon.background = "rgba(165, 42, 42, 0.52)";
                break;
                case 'purple':
                pokemon.background = "rgba(128, 0, 128, 0.37)";
                break;
               case 'blue':
                pokemon.background = "rgb(222, 243, 253)";
                break; 
                case 'white':
                pokemon.background = "rgb(245, 245, 245)";
                break;
                case 'yellow':
                pokemon.background = "rgb(252, 247, 222)";
                break;
                case 'red':
                pokemon.background = "rgba(226, 18, 14, 0.4)";
                break;
                case 'gray':
                pokemon.background = "rgba(128, 128, 128, 0.44)";
                break;
                case 'black':
                pokemon.background = "rgba(0, 0, 0, 0.2901960784313726)";
                break;
                 case 'pink':
                pokemon.background = "rgb(244, 231, 218)";
                break;
              
              default:
             
              }


          });
              console.log(results);

          })).catch(errors => {

        });
      }
    },

    openModal(id,color){

      $('#myModal').modal('show');
       document.getElementById("pokemonModal").style.backgroundColor = color;
      // $('.modal-content').css('background-color',color);
      // document.getElementById("pokemon_modal").style.backgroundColor = " +color+ " ;


        let urlOne = `https://pokeapi.co/api/v2/pokemon/${id}/`;
        let urlTwo = `https://pokeapi.co/api/v2/pokemon-species/${id}`;
        let urlThree = `https://pokeapi.co/api/v2/gender/${id}/`;
        const requestOne = axios.get(urlOne);
        const requestTwo = axios.get(urlTwo);
        const requestThree = axios.get(urlThree);
        var array1 =[];
        var array2 =[];

       axios.all([requestOne, requestTwo,requestThree]).then(axios.spread((...responses) => {

        const responseOne = { 
              ...responses[0], 
               type: responses[0].data.types.map(t => t.type.name).join(", "),
               abilities: responses[0].data.abilities.map(a => a.ability.name).join(", "),
        };

      const responseTwo = { 
              ...responses[1].data, 
               egg_group: responses[1].data.egg_groups.map(e => e.name).join(", "),
        };

         array1.push(responseOne);
         array2.push(responseTwo);

          this.pokemon_info = array1[0];
          this.pokemon_species =array2[0];
     ;

          })).catch(errors => {
        });
       
    },
    calculateCircles(val){
      console.log(val);
      const maxStat = 255;
      const numberOfCircles = 6
      return Math.round(val / maxStat * numberOfCircles);

    },

    searchPokemon(type){



    },

  },

  computed:{
    pokemon_filter_list(){
      return this.pokemon.filter(o => {
        if (this.search_pokemon==1){
            
             return o.name.toLowerCase().includes(this.filter_pokemon.toLowerCase());
          }
          if (this.search_pokemon==2){
           
             return o.habitat.toLowerCase().includes(this.filter_pokemon.toLowerCase());
          }
           if (this.search_pokemon==3){
             return o.gender.toLowerCase().includes(this.filter_pokemon.toLowerCase());
          }
           if (this.search_pokemon==4){
             return o.color.toLowerCase().includes(this.filter_pokemon.toLowerCase());
          }
      })
    },
    base_stat_total(){
      return this.pokemon_info.data.stats.reduce((acc,val)=>{
        return acc + val.base_stat;

      },0);
    }
  },


   watch: {
    // info(newValue, oldValue) {
    //   if (oldValue.length === 0 && newValue.length > 0) {
    //     this.loadDetails();
    //   }
    // }
  },

  mounted() {
  }
};
</script>

<style  scoped lang="scss">
  .pokemon-page-wrapper{
 background:#ECEFF1;
 height: 100vh;
  /*background: linear-gradient(to right, #D4D3DD, #EFEFBB);*/
  }
  .poke-container {
    overflow-y: auto;
    max-height: 420px;
	
}

.pokemon {

  background: #FFFFFFc9;
    width: 160px;
    height: 240px;
    margin: 20px ;
    display: inline-block;
    border-radius: 12px;
    box-shadow: 0px 5px 15px 1px rgba(81, 77, 92, 0.15);
    box-sizing: border-box;
    padding: 20px;
    text-align: center;
    overflow: hidden;
    transition: 0.2s all ease-in-out;

}

.pokemon .img-container {
  /*border: 1px solid;*/
	background-color: rgba(255, 255, 255, 0.6);
	border-radius: 50%;
	width: 120px;
	height: 120px;
	text-align: center;
}

.pokemon .img-container img {
	margin-top: 20px;
	max-width: 90%;
}

.pokemon .info {
	margin-top: 20px;
}

.pokemon .number {
	background-color: rgba(0, 0, 0, 0.1);
	border-radius: 10px;
	font-size: 0.8em;
	padding: 5px 10px;
}

.pokemon .name {
	margin: 15px 0 7px;
}

#myInput {
  background-image: url('/css/searchicon.png'); /* Add a search icon to input */
  background-position: 10px 12px; /* Position the search icon */
  background-repeat: no-repeat; /* Do not repeat the icon image */
  width: 100%; /* Full-width */
  font-size: 16px; /* Increase font-size */
  padding: 12px 20px 12px 40px; /* Add some padding */
  border: 1px solid #ddd; /* Add a grey border */
  margin-bottom: 12px; /* Add some space below the input */
}

</style>

