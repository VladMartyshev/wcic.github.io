<template>
  <div id="app">
    <!-- <img src="./assets/logo.png">
    <router-view/> -->
    <h1 class = "title">Візуальний квантифікатор важливості критеріїв (ВКВК)</h1>
    <div v-if="screen==0" class="screen-1">
      <div class="buttons-wrap">
        <button class="button add-criteria" @click="$refs.addCriterionMenu.open();">Додати критерії</button>
        <button class="button clear" @click="clear">Очистити</button>
        <button class="button result" @click="result">Результат</button>
      </div>
      <div class="criteria-columns">
        <div class="criteria-column" v-for="criterion in criteria" >
          <div class="weight-column" :style="{height:columnHeight+'px'}" @click="changeWeight($event, criterion)" @dblclick="toggleLock(criterion)" @contextmenu.prevent="contextMenu($event, criterion)" @wheel="scrollWeight($event, criterion)" :title="criterion.name">
            <div class="weight-bar" :style="{height: criterion.height*100+'%', background: criterion.locked?'#ff254b':'#3f90ea'}">
              <div class="criterion-weight">{{(criterion.height/1).toFixed(3)}}</div>
              <div class="criterion-id">Q<sub>{{criterion.id}}</sub></div>
            </div>
          </div>
          <!-- <div class="lock-btn" @click="toggleLock(criterion)">{{criterion.locked?"L":"U"}}</div> -->
          <!-- <i class="lock-btn handle"></i> -->
        </div>
      </div>

  <!--     <vddl-draggable v-for="(criterion, index) in criteria" :key="criterion.id"
            :draggable="criterion"
            :index="index"
            :wrapper="criteria"
            effect-allowed="move">
              <vddl-nodrag class="nodrag" >
                <div>{{criterion.name}}</div>
                  <vddl-handle
                  :handle-left="24"
                  :handle-top="24"
                  class="handle">
                </vddl-handle>
              </vddl-nodrag>
       </vddl-draggable> -->

  <!--     <ul class="list-group" v-sortable="{ handle: '.handle' }">
        <li class="list-group-item">Foo <i class="handle"></i></li>
        <li class="list-group-item">Bar <i class="handle"></i></li>
        <li class="list-group-item">Baz <i class="handle"></i></li>
      </ul> -->

<!--       <div class="criteria-form">
        <div class="form-group" v-for="criterion in criteria">
          <label>Q<sub>{{criterion.id}}</sub></label>
          <input type="text" v-model="criterion.name" class="criterion-name" :placeholder="criterion.defaultName" maxlength="30">
        </div>
        
      </div> -->

      <context-menu ref="addCriterionMenu">
        <template slot-scope="child">
          <ul class="options">
            <!-- <li @click="onClick('A')">Option A</li>
            <li @click="onClick('B')">Option B</li> -->
            <div class="form-group">
                <label>Q<sub>{{addingCriterion.id}}</sub></label>
                <input type="text" v-model="addingCriterion.name" class="criterion-name" :placeholder="addingCriterion.defaultName" maxlength="30">
            </div>
            <button class="addButton" @click="addCriterion()" >Додати</button>
            <li>

            </li>
          </ul>
        </template>
      </context-menu>
     
      <context-menu ref="criterionMenu">
        <template slot-scope="child">
          <ul class="options">
            <!-- <li @click="onClick('A')">Option A</li>
            <li @click="onClick('B')">Option B</li> -->
            <div class="form-group">
                <label>Q<sub>{{currentCriterion?currentCriterion.id:0}}</sub></label>
                <input type="text" v-model="currentCriterion.name" class="criterion-name" :placeholder="currentCriterion.defaultName" maxlength="30">
            </div>
            <div class="heightForm">
                <label>Ваговий коефіцієнт</label>
                <input type="number" step="0.001" min="0" max="1" v-model="currentCriterion.heightString" @input="heightInput(currentCriterion)" class="criterion-name" placeholder="Ваговий коефіцієнт">
            </div>
            <button class="lockButton" @click="toggleLock(currentCriterion)" :style="{background: currentCriterion.locked?'#3f90ea':'#ff254b'}">{{currentCriterion.locked?'Розблокувати':'Заблокувати'}}</button>
            <li>

            </li>
          </ul>
        </template>
      </context-menu>
    </div>
    <div v-if="screen==1">
      <table class="result-table">
        <tr v-for="criterion in sortResult">
          <td style="width:30%; text-align:right; margin-right: 10px;">Q<sub>{{criterion.id}}</sub> {{criterion.name == ""?criterion.defaultName:criterion.name}}</td>
          <td style="width:50%">
            <div class="result-bar":style="{width: criterion.height*100+'%'}"></div>
          </td>
          <td style="width:20%">
             {{criterion.height.toFixed(3)}}
          </td>

        </tr>
      </table>
       <button class="button back" @click="screen = 0">Повернутись</button>
    </div>

  </div>
</template>

<script>
class Criterion {
  constructor(id, name, defaultName, height, heightString) {
    this.id = id;
    this.name = name;
    this.defaultName = defaultName;
    this.height = height;
    this.heightString = height.toFixed(3) + "";
    this.beforeHeight = 0;
    this.locked = false;
  }
}
import contextMenu from './components/ContextMenu'
export default {
  name: 'App',
  components: {
    'context-menu': contextMenu
  },
  data(){
    return {
      criteria: [],
      currentCriterion: {id: 0, name: "", defaultName: "", height: 0},
      addingCriterion: {id: 0, name: "", defaultName: "", height: 0},
      columnHeight: 500,
      allUnlocked: false,
      screen: 0,
      lockedCount: 0,
    }
  },
  methods: {
    addCriterion(){
      if(this.criteria.length < 9){
        const criterion = new Criterion(this.addingCriterion.id, this.addingCriterion.name, this.addingCriterion.defaultName, 1/(this.criteria.length+1));
        this.criteria.push(criterion);
        this.updateWeights(criterion);
        this.addingCriterion = {id: this.criteria.length+1, name: "", defaultName: `Критерій ${this.criteria.length+1}`, height: 0}
      }
      //this.$refs.addCriterionMenu.open();

    },
    clear(){
      this.criteria = [];
      this.allUnlocked = false;
      this.init();
    },
    init(){
      for(let i = 0; i < 2; i++){
        //this.addCriterion();
        const criterion = new Criterion(this.criteria.length+1, "", `Критерій ${this.criteria.length+1}`, 1/(this.criteria.length+1));
        this.criteria.push(criterion);
        this.updateWeights(criterion);
      }
      this.addingCriterion = {id: this.criteria.length+1, name: "", defaultName: `Критерій ${this.criteria.length+1}`, height: 0}
      //console.log("init");
    },
    changeWeight(e, criterion){
      let lockedHeight = 0;
      
      //criterion.locked = true;

      if(!criterion.locked && this.lockedCount != this.criteria.length - 1){
        const barCoords = e.currentTarget.getBoundingClientRect();
        let height = (e.currentTarget.clientHeight - (e.clientY - barCoords.top + e.currentTarget.clientTop))/e.currentTarget.clientHeight;
        height = height > 1?1:height;
        height = height < 0?0:height;

        this.criteria.forEach(function(item, i, arr){
          if(item.locked && item != criterion){
            lockedHeight+=item.height;
          }
        });

        height = height <= (1 - lockedHeight) ? height : 1 - lockedHeight;
        //console.log(lockedHeight);

        criterion.height = height;
        criterion.heightString = height.toFixed(3) + "";
        if(!this.allUnlocked ){
          //criterion.locked = true;
        }
        this.updateWeights(criterion);
      }
    },
    scrollWeight(e, criterion){
      e.preventDefault();
      if(!criterion.locked && this.lockedCount != this.criteria.length - 1){
        let lockedHeight = 0;
        this.criteria.forEach(function(item, i, arr){
          if(item.locked && item != criterion){
            lockedHeight+=item.height;
          }
        });
        let height = criterion.height -= (e.deltaY/4000).toFixed(3);
        height = height > 1?1:height;
        height = height < 0?0:height;
        height = height <= (1 - lockedHeight) ? height : 1 - lockedHeight;
        criterion.height = height;
        criterion.heightString = height.toFixed(3) + "";
        this.updateWeights(criterion);
        //console.log(e);
      }
    },
    updateWeights(changedCriterion) {
      let lockedHeight = 0;
      let unlockedHeight = 0;
      let unlockedCount = 0;
      const that = this;
      this.criteria.forEach(function(item, i, arr){
        if(item.locked || item == changedCriterion){
          //console.log("locked found")
          lockedHeight+=item.height;
        } else {
          // unlockedCount++;
          unlockedHeight+=item.height;
        }
        if(!item.locked){
          unlockedCount++;
        }
      });
      if(unlockedCount == 0){
        this.allUnlocked = true;
        this.criteria.forEach(function(item, i, arr){
          item.locked = false;
        })
      }
      let updated = 0;
      let updatedHeight = 0;
      //console.log(lockedHeight);
      //console.log(unlockedHeight);
      // this.criteria.forEach(function(item, i, arr){
      //   if(!item.locked && item != changedCriterion){
      //     item.height -= (100 - lockedHeight) == 0 ? 0 : ((changedCriterion.height - changedCriterion.beforeHeight)*item.height)/(100 - lockedHeight);
      //     item.height = item.height > 100?100:item.height;
      //     item.height = item.height < 0?0:item.height;
      //     updated++;
      //     updatedHeight += item.height;
      //     item.beforeHeight = item.height;
      //   } else {
      //     if(that.allUnlocked){
      //       item.locked = false;
      //     }
      //   }
      // });
      this.criteria.forEach(function(item, i, arr){
        if(!item.locked && item != changedCriterion){
          item.height = unlockedHeight == 0 ? (unlockedCount == 0 ? item.height : (1-lockedHeight)/unlockedCount) : (1 - lockedHeight)/(unlockedHeight)*item.height;
          item.height = item.height > 1?1:item.height;
          item.height = item.height < 0?0:item.height;
          item.heightString = item.height.toFixed(3) + "";
          updated++;
          updatedHeight += item.height;
          item.beforeHeight = item.height;
        } else {
          // if(that.allUnlocked){
          //   item.locked = false;
          // }
        }
      });
      changedCriterion.beforeHeight = changedCriterion.height;

      const maxUnlockedCriterion = this.criteria.filter(function(elem, index, arr){
        if(elem.locked || elem == changedCriterion){
          return false;
        }
        return true;
      }).sort(function(a, b){
        if(a.height > b.height){
          return 1;
        }
        if(a.height < b.height){
          return -1;
        }
        return 0;
      })[0];
      //console.log("max", maxUnlockedCriterion.height);
      //maxUnlockedCriterion.height = 0.34;
      if(typeof maxUnlockedCriterion != "undefined"){
        let totalHeight = 0;
        this.criteria.forEach(function(item, i, arr){
          if(item != maxUnlockedCriterion){
            totalHeight += parseFloat(item.height.toFixed(3));
          }
        });
        maxUnlockedCriterion.height = 1 - totalHeight.toFixed(3);
        //console.log(totalHeight);

      }
    },
    toggleLock(criterion){
      criterion.locked = !criterion.locked;
      this.lockedCount += criterion.locked?1:-1;
    },
    contextMenu(e, criterion){
      this.currentCriterion = criterion;
      this.$refs.criterionMenu.open(e)
    },
    heightInput(criterion){
      //console.log(criterion)
      if(parseFloat(criterion.heightString) > 1){
        criterion.heightString = "1";
      }
      if(parseFloat(criterion.heightString) < 0){
        criterion.heightString = "0";
      }
      if(criterion.heightString != "0." || criterion.heightString != "0,") criterion.heightString = parseFloat(criterion.heightString);
      let lockedHeight = 0;
      this.criteria.forEach(function(item, i, arr){
        if(item.locked && item != criterion){
          lockedHeight+=item.height;
        }
      });
      let height = parseFloat(criterion.heightString == "" ? 0 : criterion.heightString);
      height = height > 1?1:height;
      height = height < 0?0:height;
      height = height <= (1 - lockedHeight) ? height : 1 - lockedHeight;
      criterion.height = height;
      this.updateWeights(criterion);
    },
    onClick (opt) {
      //console.log('Clicked', opt, this.$refs.menu.locals.id)
    },
    result(){
      this.screen = 1;
      //console.log(this.sortResult())
    },
  },
  created() {
    this.init();
    
  },
  computed: {
    sortResult(){
      return [...this.criteria].sort(function(a, b){
        if(a.height > b.height){
          return -1;
        }
        if(a.height < b.height){
          return 1;
        }
        return 0;
      });
    },
  },
}
</script>

<style>
 * {
  margin: 0;
  padding: 0;
 }
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
/*  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;*/
  margin-top: 40px;
  -moz-user-select: none; /* Mozilla Firefox */
  -ms-user-select: none; /* Internet Explorer (не поддерживается) */
  -o-user-select: none; /* Opera Presto (не поддерживается) */
  -webkit-user-select: none; /* Google Chrome, Opera Next, Safari */
}
.title {
  text-align: center;
}
.screen-1 {
  display: flex;
  width: 960px;
  margin: 0 auto;
  margin-top: 50px;
  align-items: center;
}
.buttons-wrap {
  /*display: flex;*/
  /*align-items: center;*/
  /*flex-direction: column;*/
  /*height: fit-content;*/
}
.criteria-columns {
  display: flex;
  margin: 0 auto;
  /*margin-top: 100px;*/
  width: fit-content;
}
.criteria-column {
  margin: 0 7px;
  position: relative;
}
.weight-column {
  display: flex;
  /*height: 500px;*/
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  background: #ececec;
  cursor: pointer;
  position: relative;
}
.weight-bar {
  /*background: #3f90ea;*/
  width: 30px;
  position: relative;
}
.criterion-weight {
  /*width: 100%;*/
  position: absolute;
  top: 0;
  left: 100%;
  transform: translate(-88%, -100%);
  /*text-align: center;*/
}
.criterion-id {
  width: 100%;
  position: absolute;
  bottom: 0;
  /*left: 100%;*/
  transform: translate(0, 100%);
  text-align: center;
}
.lock-btn {
  height: 30px;
  width: 30px;
  position: absolute;
  bottom: -60px;
  /*transform: translate(0, 100%;);*/
  background: #ececec;
}
.criteria-form {
  padding: 30px;
  margin: 0 auto;
  margin-top: 50px;
  /*width: fit-content;*/
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}
.form-group {
  padding: 3px;
  width: 30%;
}
.criterion-name {
  height: 30px;
  width: 90%;
  border: solid #86add8 1px;
  border-radius: 5px;
  font-size: 20px;
}
.button {
  display: block;
  margin: 0 auto;
  margin-top: 50px;
  width: 300px;
  height: 40px;
  background: #3f90ea;
  color: #FFF;
  border: none;
  border-radius: 5px;
  font-size: 20px;
  cursor: pointer;
}

.button.result, .button.back {
  background: #9273ca;
}
.button.clear {
  background: #ffa52b;
}
.vddl-list, .vddl-draggable {
  position: relative;
}

.vddl-list {
  min-height: 44px;
}

.vddl-dragging{
  opacity: 0.7;
}

.vddl-dragging-source {
  display: none;
}

.options {
  text-align: center;
  width: 350px;
  /*border: 1px black solid;*/
  list-style: none;
}
.options .form-group {
  width: 100%;
}
.options .form-group input {
  width: calc(100% - 40px);
}
.options .heightForm input{
  width: calc(100% - 150px);
}
.lockButton {
  width: 90%;
  height: 30px;
  color: #FFF;
  margin: 0 auto;
  margin-top: 20px;
}
.addButton {
  width: 90%;
  height: 30px;
  color: #FFF;
  margin: 0 auto;
  margin-top: 20px;
  background: #3c63c5;
}
.result-table {
  width: 70%;
  margin: 0 auto;
  margin-top: 50px;
}
/*.result-bar-wrap {
  width: 80%;
}*/
.result-bar {
  height: 20px;
  background: #3f90ea;
}
</style>
