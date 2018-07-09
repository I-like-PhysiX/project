<template>
  <div id ="app">
    <b-navbar toggleable="md" variant="dark" sticky>
      <b-navbar-toggle target="nav_collapse" :disabled="csakkosar"></b-navbar-toggle>
      <template>
        <div id="myContainer">
           <div class="my-3">
             <b-button-group>
             <b-btn
                     class="exPopoverReactive1"
                     size="md"
                     variant="dark"
                     style="text-align: center">
                     Fiókom
             </b-btn>
             <!-- our triggering (target) element -->
             <b-btn
                    class="exPopoverReactive1"
                    id="exPopoverReactive1"
                    ref="button"
                    :disabled="popoverShow"
                    @click="search='', csakkosar=true"
                    size="md"
                    variant="dark"
                    style="text-align: center">
                    {{this.rendeles.length}} tétel
              </b-btn>
              </b-button-group>
           </div>
          <!-- Our popover title and content render container -->
          <!-- We use placement 'auto' so popover fits in the best spot on viewport -->
          <!-- We specify the same container as the trigger button, so that popover is close to button -->
          <b-popover
                     target="exPopoverReactive1"
                     triggers="click"
                     :show.sync="popoverShow"
                     placement="bottomright"
                     container="myContainer"
                     ref="popover"
                     @show="onShow"
                     @shown="onShown"
                     @hidden="onHidden">
          <template slot="title">
          <b-btn @click="onCancel" class="close" aria-label="Close">
            <span class="d-inline-block" aria-hidden="true">&times;</span>
          </b-btn>
          Kosár
          </template>
          <div>
          <b-form-group
                        label="Név*"
                        label-for="pop1"
                        horizontal class="mb-1"
                        :state="input1state"
                        invalid-feedback="Kötelező kitölteni!">
            <b-form-input
                          ref="input1"
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
                          ref="input3"
                          type="text"
                          id="pop3"
                          size="sm"
                          v-model="input3"/>
          </b-form-group>
          <b-form-group
                        label="Tel.*"
                        label-for="pop4"
                        horizontal class="mb-1"
                        :state="input4state"
                        invalid-feedback="Kötelező kitölteni!">
            <b-form-input
                          ref="input4"
                          type="tel"
                          id="pop4"
                          size="sm"
                          v-model="input4"/>
          </b-form-group>
          <b-form-group
                        label="E-mail*"
                        label-for="pop5"
                        horizontal class="mb-1"
                        :state="input5state"
                        invalid-feedback="Kötelező kitölteni!">
            <b-form-input
                          ref="input5"
                          placeholder="példa@email.com"
                          type="email"
                          id="pop5"
                          size="sm"
                          v-model="input5"/>
          </b-form-group>
          <b-alert show class="small" style="overflow: auto; height: 188px; width: 215px;">
            <h3 style="color: red; text-align: center; margin: 45px 0px;" v-if="rendeles==0">A kosár üres!</h3>
            <div v-else show v-for="elem in rendeles">
              <p><b-button @click="remove(elem)" style="margin-right: 15px;">x</b-button>{{elem.termek}}</p>
            </div>
          </b-alert>
          <strong>Összesen fizetendő:</strong> <b>{{this.osszeg}} Ft</b>
          <br>
          <br>
          <b-btn @click="onClose" size="sm" variant="danger" style="margin-right: 35px;">Kosár kiürítése</b-btn>
          <b-btn @click="onOk" size="sm" variant="success">Vásárlás</b-btn>
        </div>
      </b-popover>
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
                            ref="search"
                            placeholder="Keresés"
                            type="text"
                            id="search"
                            v-model="search"/>
            </b-form-group>
          </li>
          <li class="nav-item active">
            <b-btn
                          class="exPopoverReactive1"
                          @click="szur3(), search=''"
                          size="md"
                          variant="dark"
                          style="text-align: center">
                          Keresés
            </b-btn>
          </li>
        </ul>
       </b-navbar-nav>
     </b-collapse>
   </b-navbar>
    <div style="text-align:center; margin: 15px auto;">Elérhető termékek</div>
    <b-modal ref="myModalRef" hide-footer title="Bevásárló alkalmazás">
      <div class="d-block text-center">
        <h3>Köszönjük a vásárlást!</h3>
      </div>
      <b-btn
             class="mt-3"
             variant="outline-danger"
             block
             @click="hideModal">Bezárás</b-btn>
    </b-modal>
    <div class="container">
    <div class="row" v-for="i in Math.ceil(this.szurttomb.length / itemsPerRow)">
      <b-card
              bg-variant="light"
              id="card"
              footer-bg-variant="secondary"
              show v-for="elem in szurttomb.slice((i - 1) * itemsPerRow, i * itemsPerRow)"
              :img-src="elem.url"
              style="max-width: 274px;"
              img-alt="A termék képe"
              img-top
              class="image">
       <div class="top-right"><h3 style="color: red;">{{elem.info}}</h3></div>
       <p class="card-text"><p>{{elem.termek}}</p> <p>{{elem.egysar}} Ft/{{elem.egys}}</p></p>
       <p class="card-text" v-if="elem.ossz > 0">Raktáron</p>
       <p class="card-text" v-else>Elfogyott</p>
       <p class="card-text"><b> <p>Mennyiség: {{elem.alap}} {{elem.egys}}</p> <p>Ár: {{Math.round(elem.alap*elem.egysar)}} Ft</p></b></p>
       <b-input
                class="form-control input-md text-center"
                :disabled="elem.ossz === 0 ? true : false"
                v-model="elem.alap"
                @input="szur2()"
                min="0"
                :max="elem.ossz"
                v-if="elem.egys=='kg'"
                step="0.2"
                size="md"
                type="range"
                id="myNumber"
                style="padding: 20px 0; margin-top: 0px;"/>
        <b-input
                class="form-control input-md text-center"
                :disabled="elem.ossz === 0 ? true : false"
                v-if="elem.egys!='kg'"
                v-model="elem.alap"
                @input="szur2()"
                min="0"
                :max="elem.ossz"
                step="1"
                type="range"
                size="md"
                style="padding: 20px 0; margin-top: 0px;"/>
      </b-card>
    </div>
  </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      csakkosar: false,
      name: 'shlist',
      show: true,
      search:'',
      searchstate: null,
      mid: 1,
      input1: '',
      input1state: null,
      input3: '',
      input3state: null,
      input4: '',
      input4state: null,
      input5: '',
      input5state: null,
      popoverShow: false,
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
    init(){
      this.itemsPerRow=this.szurttomb.length;
      let szurttomb = this.tomb.filter(v => v.info=="Akció!");
      this.szurttomb=szurttomb;
      this.itemsPerRow=this.szurttomb.length;
      console.log("init");
    },
    clear(){
      this.rendeles=[];
    },
    create_selection () {
      let mySet = new Set();
      this.tomb.forEach(v => mySet.add(v.type));
      let myArray = Array.from(mySet);
      let dropdown = document.getElementById("selecttype");
      for (var i = 0; i < myArray.length; ++i) {
        dropdown[dropdown.length] = new Option(myArray[i], myArray[i])};
    },
    remove(elem) {
      elem.alap=0;
      this.rendeles.splice(this.rendeles.indexOf(elem), 1);
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
      this.popoverShow = false;
      console.log('Quit cart');
    },
    onClose () {
      this.rendeles.forEach(v => v.alap=0);
      this.clear();
      console.log('Reset cart');
    },
    onOk () {
      if (!this.input1) { this.input1state = false; }
      if (!this.input3) { this.input3state = false; }
      if (!this.input4) { this.input4state = false; }
      if (!this.input5) { this.input5state = false; }
      if (this.input1 && this.input3 && this.input4 && this.input5 && this.rendeles!=0) {
        console.log('Make API request');
        this.onReset();
        this.showModal();
      }
    },
    onShow () {
      /* This is called just before the popover is shown */
      /* Reset our popover "form" variables */
      this.input1 = '';
      this.input3 = '';
      this.input4 = '';
      this.input5 = '';
      this.input1state = null;
      this.input3state = null;
      this.input4state = null;
      this.input5state = null;
    },
    onShown () {
      /* Called just after the popover has been shown */
      /* Transfer focus to the first input */
      this.focusRef(this.$refs.input1);
    },
    onHidden () {
      /* Called just after the popover has finished hiding */
      /* Bring focus back to the button */
      this.focusRef(this.$refs.button);
    },
    focusRef (ref) {
      /* Some references may be a component, functional component, or plain element */
      /* This handles that check before focusing, assuming a focus() method exists */
      /* We do this in a double nextTick to ensure components have updated & popover positioned first */
      this.$nextTick(() => {
        this.$nextTick(() => { (ref.$el || ref).focus() });
      });
  },
    onReset () {
      Object.assign(this.$data, this.$options.data());
      console.log('Reset values');
    },
    szur () {
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
        let rendeles=this.tomb.filter(v => v.alap!=0);
        this.rendeles=rendeles;
      },
      szur3(){
        if (!this.search) { this.input5state = false; }
        if (this.search){
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
</style>
