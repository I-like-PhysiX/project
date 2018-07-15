<template>
  <div id ="app">
    <b-navbar toggleable="md" variant="dark" sticky>
      <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
      <template>
        <div id="myContainer">
           <div class="my-3">
             <b-button-group>
               <b-btn
                       class="exPopoverReactive1"
                       size="md"
                       variant="dark"
                       style="text-align: center"
                       v-on:click="init(), search='', sortType='', selected=''">
                       Webshop
               </b-btn>
             <b-btn
                    class="exPopoverReactive1"
                    v-on:click="search='', csakkosar=true, csaktermekek=false"
                    v-if="csaktermekek"
                    size="md"
                    variant="dark"
                    style="text-align: center">
                    {{this.rendeles.length}} tétel
              </b-btn>
              <b-btn
                     class="exPopoverReactive1"
                     v-on:click="onCancel()"
                     v-if="csakkosar"
                     size="md"
                     variant="dark"
                     style="text-align: center">
                     Vissza a termékekhez
               </b-btn>
              </b-button-group>
           </div>
    </div>
    </template>
    <b-collapse is-nav id="nav_collapse">
      <b-navbar-nav class="ml-auto">
        <ul class="navbar-nav mr-auto" style="margin-top: 15px;">
          <li class="nav-item active">
            <select id="selecttype" v-model="selected" v-on:change="szur()" class="form-control input-sm">
              <option value="" disabled>Termékfajták</option>
            </select>
          </li>
          <li class="nav-item active">
            <select name="sortBy" id="sortBy" v-on:change="szur()" v-model="sortType" class="form-control input-sm">
              <option v-for="item in sortOptions" :value="item.value" :disabled="item.isdisabled">{{item.text}}</option>
            </select>
          </li>
          <li class="nav-item active">
            <b-form-group
                          label-for="search"
                          :state="searchstate">
              <b-form-input
                            class="form-control"
                            size="md"
                            placeholder="Keresés"
                            type="text"
                            id="search"
                            v-model="search"/>
            </b-form-group>
          </li>
          <li class="nav-item active">
            <b-btn
                          class="exPopoverReactive1"
                          v-on:click="szur2(), search=''"
                          size="md"
                          variant="dark"
                          style="text-align: center"
                          :disabled="this.search === '' ? true : false">
                          Keresés
            </b-btn>
          </li>
        </ul>
       </b-navbar-nav>
     </b-collapse>
   </b-navbar>
    <div style="text-align:center; margin: 15px auto;" v-if="csaktermekek">Elérhető termékek</div>
    <div style="text-align:center; margin: 15px auto;" v-if="csakkosar">Kosár</div>
    <div style="text-align:center; margin: 15px auto;" v-if="csakadatok">Adatok megadása és vásárlás</div>
    <div class="main">
    <b-modal ref="myModalRef" hide-footer title="Bevásárló alkalmazás">
      <div class="d-block text-center">
        <h3>Köszönjük a vásárlást!</h3>
      </div>
      <b-btn
             class="mt-3"
             variant="outline-danger"
             block
             v-on:click="hideModal">Bezárás</b-btn>
    </b-modal>
    <div class="container">
      <div v-if="csakkosar">
        <b-alert show class="small" style="background-color: #F5F5F5;">
          <h3 style="color: red; text-align: center; margin: 45px 0px;" v-if="rendeles==0">A kosár üres!</h3>
          <table v-else>
            <thead>
              <tr>
                <td><strong>Termék neve</strong></td>
                <td><strong>Cikkszám</strong></td>
                <td><strong>Egységár</strong></td>
                <td><strong>Mennyiség</strong></td>
                <td><strong>Részösszeg</strong></td>
              </tr>
            </thead>
            <tbody>
              <tr v-for="elem in rendeles" :key="elem.id">
                <td>
                  {{ elem.termek }}
                </td>
                <td>
                  {{ elem.id }}
                </td>
                <td>
                  {{ elem.egysar }}/{{ elem.egys }}
                </td>
                <td>
                  <div
                       :disabled="elem.ossz === 0 ? true : false"
                       v-if="elem.egys=='kg'"
                       v-model="elem.alap">
                       <b-button v-on:click="removefromcart(elem, 0.1)">-</b-button>
                       <b-button v-on:click="addtocart(elem, 0.1)">+</b-button>
                       {{elem.alap}} {{elem.egys}}
                  </div>
                  <div
                       :disabled="elem.ossz === 0 ? true : false"
                       v-if="elem.egys!='kg'"
                       v-model="elem.alap">
                       <b-button v-on:click="removefromcart(elem, 1)">-</b-button>
                       <b-button v-on:click="increment(elem, 1)">+</b-button>
                       {{elem.alap}} {{elem.egys}}
                  </div>
                </td>
                <td>
                  {{Math.round(elem.alap*elem.egysar)}} Ft
                  <b-button v-on:click="del(elem)">X</b-button>
                </td>
              </tr>
            </tbody>
          </table>
        </b-alert>
        <strong>Összesen fizetendő:</strong> <b>{{this.osszeg}} Ft</b>
        <br>
        <br>
        <b-button-group class="responsive">
          <b-btn v-on:click="onClose()" size="sm" variant="danger">Kosár kiürítése</b-btn>
          <b-btn v-on:click="onShow(), csakkosar=false, csakadatok=true" size="sm" variant="success" :disabled="this.rendeles==0 ? true : false">Tovább az adatokhoz</b-btn>
        </b-button-group>
      </div>
    <div v-if="csakadatok">
      <b-form-group
                    label="Név*"
                    label-for="pop1"
                    horizontal class="mb-1"
                    :state="input1state"
                    invalid-feedback="Kötelező kitölteni!">
        <b-form-input
                      type="text"
                      id="pop1"
                      size="sm"
                      v-model="input1"/>
      </b-form-group>
      <b-form-group
                    label="Cím*"
                    label-for="pop3"
                    horizontal class="mb-1"
                    :state="input3state"
                    invalid-feedback="Kötelező kitölteni!">
        <b-form-input
                      type="text"
                      id="pop3"
                      size="sm"
                      v-model="input3"/>
      </b-form-group>
      <b-form-group
                    label="Telefonszám*"
                    label-for="pop4"
                    horizontal class="mb-1"
                    :state="input4state"
                    invalid-feedback="Kötelező kitölteni!">
        <b-form-input
                      type="tel"
                      id="pop4"
                      size="sm"
                      v-model="input4"/>
      </b-form-group>
      <b-form-group
                    label="E-mail cím*"
                    label-for="pop5"
                    horizontal class="mb-1"
                    :state="input5state"
                    invalid-feedback="Kötelező kitölteni!">
        <b-form-input
                      placeholder="példa@email.com"
                      type="email"
                      id="pop5"
                      size="sm"
                      v-model="input5"/>
      </b-form-group>
      <strong>Összesen fizetendő:</strong> <b>{{this.osszeg}} Ft</b>
      <br>
      <br>
      <b-button-group class="responsive">
        <b-btn v-on:click="csakkosar=true, csakadatok=false" size="sm" variant="danger">Vissza a kosárhoz</b-btn>
        <b-btn v-on:click="onOk()" size="sm" variant="success">Vásárlás</b-btn>
      </b-button-group>
    </div>
    <div class="row" v-for="i in Math.ceil(this.szurttomb.length / itemsPerRow)" v-if="csaktermekek">
      <b-card
              bg-variant="light"
              id="card"
              footer-bg-variant="secondary"
              v-for="elem in szurttomb.slice((i - 1) * itemsPerRow, i * itemsPerRow)"
              :key="elem.id"
              :img-src="elem.url"
              style="max-width: 274px;"
              img-alt="A termék képe"
              img-top
              class="image">
       <div class="top-right"><h3 style="color: red;">{{elem.info}}</h3></div>
       <p class="card-text"><p>{{elem.termek}}</p> <p>{{elem.egysar}} Ft/{{elem.egys}}</p></p>
       <p class="card-text" v-if="elem.ossz > 0">Raktáron</p>
       <p class="card-text" v-else>Elfogyott</p>
       <div
            :disabled="elem.ossz === 0 ? true : false"
            v-if="elem.egys=='kg'"
            v-model="elem.alap">
            <b-button v-on:click="addtocart(elem, 0.1)">Kosárba</b-button>
       </div>
       <div
            :disabled="elem.ossz === 0 ? true : false"
            v-if="elem.egys!='kg'"
            v-model="elem.alap">
            <b-button v-on:click="addtocart(elem, 1)">Kosárba</b-button>
       </div>
      </b-card>
    </div>
  </div>
</div>
<div class="text-mid">
  <div class="center">
  </div>
</div>
<b-navbar toggleable="md" variant="dark" class="bottom">
  <div class="footer" style="color:white;  margin: 0px auto;">
    © 2018. All Rights Reserved
  </div>
</b-navbar>
</div>
</template>

<script>
export default {
  data() {
    return {
      csakadatok: false,
      csakkosar: false,
      csakvasarlas: false,
      csaktermekek: true,
      name: 'shlist',
      show: true,
      search:'',
      searchstate: null,
      input1: '',
      input1state: null,
      input3: '',
      input3state: null,
      input4: '',
      input4state: null,
      input5: '',
      input5state: null,
      szurttomb: [],
      rendeles: [],
      selected: '',
      sortType: '',
      sortOptions: [
          { text: 'Rendezés', value: '', isdisabled: true},
          { text: 'abc szerint', value: 'termek', isdisabled: false},
          { text: 'ár szerint', value: 'egysar', isdisabled: false}
       ],
      itemsPerRow: 1,
      tomb: [
        {id:1, type: "gyümölcs", termek: "Alma, gála", info: "Akció!", egysar:310, egys:'kg', alap:0, ossz:10, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:2, type: "gyümölcs", termek: "Alma, golden", info: "", egysar:350, egys:'kg', alap:0, ossz:0, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:3, type: "gyümölcs", termek: "Alma, jonatán", info: "", egysar:290, egys:'kg', alap:0, ossz:10, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:4, type: "gyümölcs", termek: "Körte, vilmos", info: "", egysar:550, egys:'kg', alap:0, ossz:10, url: 'https://3.imimg.com/data3/WQ/FT/MY-7265137/fresh-pear-500x500.jpg'},
        {id:5, type: "gyümölcs", termek: "Narancs, lédig", info: "Akció!", egysar:350, egys:'kg', alap:0, ossz:0, url: 'http://images6.fanpop.com/image/photos/34500000/Orange-Fruit-orange-34512935-600-600.png'},
        {id:6, type: "gyümölcs", termek: "Banán, lédig", info: "", egysar:350, egys:'kg', alap:0, ossz:10, url: 'https://5.imimg.com/data5/CT/TI/MY-8900429/ripened-organic-banana-500x500.jpg'},
        {id:7, type: "gyümölcs", termek: "Eper, magyar", info: "Akció!", egysar:600, egys:'kg', alap:0, ossz:10, url: 'https://5.imimg.com/data5/FY/QK/MY-40752636/fresh-strawberry-500x500.jpg'},
        {id:8, type: "tejtermék", termek: "Tejföl, kunsági, 250 g", info: "", egysar:250, egys:'doboz', alap:0, ossz:10, url: 'https://i5.walmartimages.com/asr/278c6980-ff4c-4c6f-8bcc-c7a13bd4b987_1.9513a8277bd8464ff661e6ddf8113f8f.jpeg?odnHeight=450&odnWidth=450&odnBg=FFFFFF'},
        {id:9, type: "tejtermék", termek: "Tejföl, Riska, 250g", info: "Akció!", egysar:350, egys:'doboz', alap:0, ossz:0, url: 'https://i5.walmartimages.com/asr/278c6980-ff4c-4c6f-8bcc-c7a13bd4b987_1.9513a8277bd8464ff661e6ddf8113f8f.jpeg?odnHeight=450&odnWidth=450&odnBg=FFFFFF'}
      ]
    }
  },
  watch: {
     input1 (val) {
       if (val) {
         this.input1state = true;
       }
     },
     input3 (val) {
       if (val) {
         this.input3state = true;
       }
     },
     input4 (val) {
       if (val) {
         this.input4state = true;
       }
     },
     input5 (val) {
       if (val) {
         this.input5state = true;
       }
     },
     search (val) {
       if (val) {
         this.searchstate = true;
       }
     }
   },
  methods: {
    addtocart(elem, step){
      if (elem.alap < elem.ossz) {
        elem.alap = Math.round((elem.alap + step) * 10) / 10;
        this.rendeles.push(elem);
        let set = new Set(this.rendeles);
        this.rendeles=Array.from(set);
      }
    },
    removefromcart(elem, step){
      elem.alap = Math.round((elem.alap - step) * 10) / 10;
      if (elem.alap==0){
        this.del(elem);
      }
    },
    del(elem){
      elem.alap=0;
      this.rendeles.splice(this.rendeles.indexOf(elem), 1);
    },
    init(){
      this.itemsPerRow=this.szurttomb.length;
      let szurttomb = this.tomb.filter(v => v.info=="Akció!");
      this.szurttomb=szurttomb;
      this.itemsPerRow=this.szurttomb.length;
      this.onCancel();
      console.log("init");
    },
    create_selection () {
      let mySet = new Set();
      this.tomb.forEach(v => mySet.add(v.type));
      let myArray = Array.from(mySet);
      let dropdown = document.getElementById("selecttype");
      for (var i = 0; i < myArray.length; ++i) {
        dropdown[dropdown.length] = new Option(myArray[i], myArray[i])};
    },
    showModal () {
      this.$refs.myModalRef.show()
    },
    hideModal () {
      this.$refs.myModalRef.hide()
      this.init();
    },
    onCancel(){
      this.search='';
      this.csakkosar=false;
      this.csaktermekek=true;
      console.log('Quit cart');
    },
    onClose () {
      this.rendeles.forEach(v => v.alap=0);
      this.rendeles=[];
      console.log('Reset cart');
    },
    onOk () {
      if (!this.input1) { this.input1state = false; }
      if (!this.input3) { this.input3state = false; }
      if (!this.input4) { this.input4state = false; }
      if (!this.input5) { this.input5state = false; }
      if (this.input1 && this.input3 && this.input4 && this.input5) {
        console.log('Make API request');
        this.onReset();
        this.showModal();
      }
    },
    onShow () {
      /* This is called just before the basket is shown */
      /* Reset our "form" variables */
      this.input1 = '';
      this.input3 = '';
      this.input4 = '';
      this.input5 = '';
      this.input1state = null;
      this.input3state = null;
      this.input4state = null;
      this.input5state = null;
    },
    onReset () {
      Object.assign(this.$data, this.$options.data());
      console.log('Reset values');
    },
    szur () {
      this.csakadatok=false
      this.csakkosar=false;
      this.csaktermekek=true;
      let szurttomb = this.tomb.filter(v => v.type==this.selected);
      switch (this.sortType) {
          case 'termek':
          szurttomb=szurttomb.sort((a,b) => a.termek > b.termek);
              break;
          case 'egysar':
          szurttomb=szurttomb.sort((a,b) => a.egysar > b.egysar);
              break;
            };
        this.szurttomb=szurttomb;
        this.itemsPerRow=this.szurttomb.length;
      },
      szur2(){
        this.csakadatok=false
        this.csakkosar=false;
        this.csaktermekek=true;
        if (!this.search) {
          this.input5state = false;
        }else {
          let szurttomb =this.tomb.filter(v => RegExp(this.search,'i').test(v.termek)).slice(0,9);
          this.szurttomb=szurttomb;
          this.itemsPerRow=this.szurttomb.length;
        }
      }
  },
  mounted() {
    this.create_selection();
    this.init();
  },
  computed: {
    osszeg() {
      return Math.round(this.rendeles.reduce((o,v)=>o+v.egysar*v.alap,0)/5)*5;
    }
  }
}
</script>

<style scoped>
 #app{
  font-family: courier;
  box-sizing: border-box;
    display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -webkit-flex-direction: column;
    -ms-flex-direction: column;
    flex-direction: column;
    margin: 0 auto;
    min-height: 100vh;
 }
 .main{
  width:100%;
  overflow:hidden;
}
.bottom,.text-mid {
  padding:20px 10px;
  height:100%;
}
.text-mid {
  margin-top: auto;
  background:white;
}
 .form-control {
    display: inline-block;
}
#card:hover{
  border-width:1px;
  border-style:ridge;
  border-color:#4285F4;
}
.exPopoverReactive1:hover{
  border-width:1px;
  border-style:ridge;
  border-color:#FFFFFF;
}
.image {
    position: relative;
}
.top-right {
    position: absolute;
    top: 8px;
    right: 16px;
}
table {
    border-collapse: collapse;
    width: 100%;
}
th, td {
    padding: 8px;
    text-align: left;
    border-bottom: 1px solid #ddd;
}
</style>
