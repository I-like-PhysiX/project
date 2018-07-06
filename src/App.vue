<template>
  <div id ="app">
    <b-navbar toggleable="md" variant="dark" sticky>
      <b-navbar-toggle target="nav_collapse" :disabled="csakkosar"></b-navbar-toggle>
      <template>
        <div id="myContainer">
           <div class="my-3">
             <!-- our triggering (target) element -->
             <b-btn
                    id="exPopoverReactive1"
                    ref="button"
                    :disabled="popoverShow"
                    @click="search='', csakkosar=true"
                    size="md"
                    variant="dark"
                    style="margin: 0 1em;">
                    Kosár
              </b-btn>
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
          <b-alert show class="small">
            <strong>A kosárba helyezett termékek:</strong>
            <br>
            <br>
            <p style="color: red" v-if="rendeles==0">A kosár üres!</p>
            <div v-else show v-for="elem in rendeles" :key="elem.id">
              <p><b-button @click="remove(elem)">DEL</b-button>&nbsp;&nbsp;&nbsp;{{elem.termek}}</p>
            </div>
          </b-alert>
          <strong>Összesen fizetendő:</strong> <b>{{this.osszeg}} Ft</b>
          <br>
          <br>
          <b-btn @click="onClose" size="sm" variant="danger">Kosár kiürítése</b-btn>
          &nbsp;
          <b-btn @click="onOk" size="sm" variant="success">Vásárlás</b-btn>
        </div>
      </b-popover>
    </div>
    </template>
    <b-collapse is-nav id="nav_collapse">
      <b-navbar-nav class="ml-auto" style="margin: 0 10px;">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item active" style="margin: 10px 5px;">
            <select id="selecttype" v-model="selected" v-on:click="szur()" class="form-control input-sm">
              <option value="">Termékfajták</option>
            </select>
          </li>
          <li class="nav-item active" style="margin: 10px 5px;">
            <select name="sortBy" id="sortBy" v-on:click="szur()" v-model="sortType" class="form-control input-sm">
              <option v-for="item in sortOptions" :value="item.value">{{item.text}}</option>
            </select>
          </li>
          <li class="nav-item active" style="margin: 10px 5px;">
            <input
                     class="mr-sm-2 form-control input-sm"
                     type="text"
                     v-model="search"
                     @input="szur()"
                     placeholder="Keresés"/>
          </li>
        </ul>
       </b-navbar-nav>
     </b-collapse>
   </b-navbar>
    <br>
    <p style="text-align:center;">Elérhető termékek</p>
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
    <b-card-group columns fluid id="shapp">
      <b-card
              bg-variant="light"
              style="max-width: 40rem;"
              id="card"
              footer-bg-variant="secondary"
              show v-for="elem in szurttomb" :key="elem.id"
              :img-src="elem.url"
              fluid alt="Responsive image"
              img-alt="A termék képe"
              img-top>
       <p class="card-text">{{elem.termek}}, {{elem.egysar}} Ft/{{elem.egys}}</p>
       <p class="card-text" v-if="elem.ossz > 0">Raktáron</p>
       <p class="card-text" v-else>Elfogyott</p>
       <p class="card-text"><b> Mennyiség: {{elem.alap}} {{elem.egys}} Ár: {{Math.round(elem.alap*elem.egysar)}} Ft</b></p>
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
                style="padding: 20px 0; margin-top: 10px;"/>
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
                style="padding: 20px 0; margin-top: 10px;"/>
      </b-card>
    </b-card-group>

    <div class="row" v-for="i in Math.ceil(this.szurttomb.length / itemsPerRow)">
    <span v-for="elem in szurttomb.slice((i - 1) * itemsPerRow, i * itemsPerRow)">
      {{elem.id}}
    </span>
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
          { text: 'Rendezés', value: '' },
          { text: 'abc szerint', value: 'termek' },
          { text: 'ár szerint', value: 'egysar' }
       ],
      itemsPerRow: 4,
      tomb: [
        {id:1, type: "gyümölcs", termek: "Alma, gála", egysar:310, egys:'kg', alap:0, ossz:50, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:2, type: "gyümölcs", termek: "Alma, golden", egysar:350, egys:'kg', alap:0, ossz:0, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:3, type: "gyümölcs", termek: "Alma, jonatán", egysar:290, egys:'kg', alap:0, ossz:100, url: 'https://4.imimg.com/data4/QY/GN/MY-24065638/fresh-apple-500x500.jpg'},
        {id:4, type: "gyümölcs", termek: "Körte, vilmos", egysar:550, egys:'kg', alap:0, ossz:100, url: 'https://3.imimg.com/data3/WQ/FT/MY-7265137/fresh-pear-500x500.jpg'},
        {id:5, type: "gyümölcs", termek: "Narancs, lédig", egysar:350, egys:'kg', alap:0, ossz:0, url: 'https://4.imimg.com/data4/YM/OY/MY-23950624/fresh-oranges-250x250.jpg'},
        {id:6, type: "gyümölcs", termek: "Banán, lédig", egysar:350, egys:'kg', alap:0, ossz:50, url: 'https://5.imimg.com/data5/CT/TI/MY-8900429/ripened-organic-banana-500x500.jpg'},
        {id:7, type: "gyümölcs", termek: "Eper, magyar", egysar:600, egys:'kg', alap:0, ossz:100, url: 'http://iconshow.me/media/images/food/Fruit-Salad-icons/png/256/strawberry.png'},
        {id:8, type: "tejtermék", termek: "Tejföl, kunsági, 250 g", egysar:250, egys:'doboz', alap:0, ossz:100, url: 'https://s11284.pcdn.co/wp-content/uploads/2017/06/Breakstones-sour-cream.jpg'},
        {id:9, type: "tejtermék", termek: "Tejföl, Riska, 250g", egysar:350, egys:'doboz', alap:0, ossz:0, url: 'https://s11284.pcdn.co/wp-content/uploads/2017/06/Breakstones-sour-cream.jpg'}
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
     }
   },
  methods: {
    create_selection () {
      let tomb2=[];
      this.tomb.forEach(v => tomb2.push(v.type));
      let myArray = Array.from(new Set(tomb2));
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
    },
    onCancel(){
      this.search='';
      this.csakkosar=false;
      this.popoverShow = false;
      console.log('Quit cart');
    },
    onClose () {
      this.onReset();
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
      let szurttomb = this.tomb.filter(v => v.type==this.selected).filter(v => RegExp(this.search,'i').test(v.termek)).slice(0,9);
      switch (this.sortType) {
          case 'termek':
          szurttomb=szurttomb.sort((a,b) => a.termek > b.termek);
              break;
          case 'egysar':
          szurttomb=szurttomb.sort((a,b) => a.egysar > b.egysar);
              break;
            };
        this.szurttomb=szurttomb;
      },
      szur2(){
        let rendeles=this.tomb.filter(v => v.alap!=0)
        this.rendeles=rendeles;
      }
  },
  mounted() {
    this.create_selection();
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
 #shapp {
   margin:30px;
   font-family: courier;
   display: block;
 }
 .form-control {
    display: inline-block;
}
#card:hover{
  border-width:2px;
  border-style:ridge;
  border-color:#4285F4
}

.row {
  border: solid 1px #404040;
  padding: 10px;
  margin:30px;
}

</style>
