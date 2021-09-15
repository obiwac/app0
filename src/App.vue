<template>
  <div id="app">
    <div class="rows">
      <div class="cols" v-for="(row, indexX) in board" :key="indexX">
        <a class="case-container" href="#" v-for="(item, indexY) in row" :key="indexY" v-on:click="pressed(indexX, indexY)">
          <Case :content="item" :direction="get_direction()"/>
        </a>
      </div>
    </div>
    <div class="cols">
      <button v-on:click="togglePotter">
        <span v-if="potterPos == undefined & currentAction == 'potter'">Placement de Harry Potter (h)</span>
        <span v-else-if="potterPos == undefined">Placer Harry Potter (h)</span>
        <span v-else>Retirer Harry Potter (h)</span>
      </button>
      <button v-on:click="toggleEvil">
        <span v-if="evilPos == undefined & currentAction == 'evil'">Placement Le Mal (e)</span>
        <span v-else-if="evilPos == undefined">Placer Le Mal (e)</span>
        <span v-else>Retirer Le Mal (e)</span>
      </button>
      <button v-on:click="toggleBuilding">
        <span v-if="currentAction == 'building'">Placement d'un batiment (b)</span>
        <span v-else>Placer un batiment (b)</span>
      </button>
      <button v-on:click="toggleTroll">
        <span v-if="currentAction == 'troll'">Placement d'un troll (t)</span>
        <span v-else>Placer un troll (t)</span>
      </button>
      <button v-on:click="toggleRemove">
        <span v-if="currentAction == 'remove'">Suppression une case (r)</span>
        <span v-else>Supprimer d'une case (r)</span>
      </button>
    </div>
    <div class="cols">
      <button v-on:click="turn_left">turn left (q)</button>
      <button v-on:click="turn_right">turn right (d)</button>
      <button v-on:click="destroy_dark_force">destroy dark force (a)</button>
      <button v-on:click="move">move (z)</button>
    </div>
    <div class="cols">
      <button v-on:click="resolve">Resolve</button>
      <button v-on:click="simulate">Simulate</button>
      <select name="map" id="map" v-on:change="chooseMap">
        <option value="clear">Vide</option>
        <option value="map1">Map 1</option>
        <option value="map2">Map 2</option>
        <option value="map3">Map 3</option>
        <option value="map4">Obstacle 1</option>
        <option value="map5">Obstacle 2</option>
        <option value="map6">Obstacle 3</option>
        <option value="map7">Obstacle 4</option>
      </select>
    </div>
    <textarea id="src-input" auto-grow>
	</textarea>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import Case from './components/Case.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld,
    Case
  },
  data() {
    return {
      timeout_ms : 500,
      board : [],
      potterPos : undefined,
      evilPos : undefined,
      currentAction : undefined,
      currentDirection : 1,
      directions : ['north', 'east','south', 'west'],
      mouvementDirection : {
        north : {x : -1,y : 0},
        east : {x : 0, y : 1},
        south : {x : 1, y : 0},
        west : {x : 0, y : -1}
      },
      nOp : 1
    }
  },
  methods : {
    pressed(x, y) {
      switch (this.currentAction) {
        case 'potter':
          if (this.board[x][y] != 'blank') alert("Il y a déjà quelque chose sur la case")
          else {
            this.potterPos = {'x' : x, 'y' : y};
            this.board[x][y] = "potter";
            this.currentAction = undefined;
          }
          break;
        case 'evil':
          if (this.board[x][y] != 'blank') alert("Il y a déjà quelque chose sur la case")
          else {
            this.evilPos = {'x' : x, 'y' : y};
            this.board[x][y] = "evil";
            this.currentAction = undefined;
          }
          break;
        case 'building':
          if (this.board[x][y] != 'blank') alert("Il y a déjà quelque chose sur la case")
          else {
            this.board[x][y] = "building";
            this.currentAction = undefined;
          }
          break;

        case 'troll':
          if (this.board[x][y] != 'blank') alert("Il y a déjà quelque chose sur la case")
          else {
            this.board[x][y] = "troll";
            this.currentAction = undefined;
          }
          break;
        case 'remove':
          switch (this.board[x][y]) {
            case 'potter':
              this.potterPos = undefined
              break;
            case 'evil':
              this.evilPos = undefined
              break;
            default:
              break;
          }
          this.board[x][y] = 'blank';
          this.currentAction = undefined
          break;
        default:
          break;
      }
    },
    togglePotter() {
      if (this.currentAction == "potter") this.currentAction = undefined
      else if (this.potterPos == undefined) this.currentAction = 'potter'
      else {
        this.board[this.get_x()][this.get_y()] = "blank";
        this.potterPos = undefined
      }
    },
    toggleEvil() {
      if (this.currentAction == "evil") this.currentAction = undefined
      else if (this.evilPos == undefined) this.currentAction = 'evil'
      else {
        this.board[this.get_target_x()][this.get_target_y()] = "blank";
        this.evilPos = undefined
      }
    },
    toggleBuilding() {
      this.currentAction = this.currentAction == undefined ? 'building' : undefined
    },
    toggleTroll() {
      this.currentAction = this.currentAction == undefined ? 'troll' : undefined
    },
    toggleRemove() {
      this.currentAction = this.currentAction == undefined ? 'remove' : undefined
    },
    chooseMap(map) {
      switch (map.srcElement.value) {
        case 'clear':
          this.clearMap();
          break;
        case 'map1':
          this.setupMap1();
          break;
        case 'map2':
          this.setupMap2();
          break;
        case 'map3':
          this.setupMap3();
          break;
        case 'map4':
          this.setupMap4();
          break;
        case 'map5':
          this.setupMap5();
          break;
        case 'map6':
          this.setupMap6();
          break;
        case 'map7':
          this.setupMap7();
          break;
        default:
          break;
      }
    },
    // from game
    turn_left() {
      this.currentDirection = (this.currentDirection + 3) % 4
    },
    turn_right() {
      this.currentDirection = (this.currentDirection + 1) % 4
    },
    get_direction() {
      return this.directions[this.currentDirection];
    },
    get_mouvementDirection () {
      return this.mouvementDirection[this.get_direction()]
    },
    setRandomDirection() {
      this.currentDirection =  Math.floor(Math.random() * 4)
    },
    get_x() {
      if (this.potterPos == undefined) throw Error("Potter not placed")
      return this.potterPos.x
    },
    get_y() {
      if (this.potterPos == undefined) throw Error("Potter not placed")
      return this.potterPos.y
    },
    get_target_x() {
      if (this.evilPos == undefined) throw Error("Evil not placed")
      return this.evilPos.x
    },
    get_target_y() {
      if (this.evilPos == undefined) throw Error("Evil not placed")
      return this.evilPos.y
    },
    is_on_target() {
      return this.get_x() == this.get_target_x() && this.get_y() == this.get_target_y()
    },
    destroy_dark_force() {
      if (this.is_on_target()) {
        alert("Bravo, vous avez vaincu le mal")
        this.evilPos = undefined
      }
      else alert("Vous n'avez pas toucher le mal")
    },
    get_new_positions() {
      return {x : this.get_x() + this.get_mouvementDirection().x, y:this.get_y() + this.get_mouvementDirection().y}
    },
    on_next_case() {
      let newPos = this.get_new_positions();
      if (newPos.x < 0 || newPos.x > 11 || newPos.y < 0 || newPos.y > 15) return 'void'
      return this.board[newPos.x][newPos.y]
    },
    can_move() {
      let next = this.on_next_case();
      if (next == 'building' || next == 'void') return false
      return true;
    },
    is_in_front_of_enemy() {
      if (this.on_next_case() == 'troll') return true
      return false
    },
    kill_potter() {
      this.board[this.get_x()][this.get_y()] = 'blank';
      this.potterPos = undefined;
    },
    move() {
      switch (this.on_next_case()) {
        case 'building':
          alert('Vous ne pouvez pas vous déplacer sur un batiment')
          return;
        case 'troll':
          this.kill_potter()
          alert('Vous êtes mort');
          return;
        case 'void':
          this.kill_potter()
          alert('Vous êtes tombé dans le vide intersidéral');
          return;
        default: // evil or blank
          if (this.is_on_target()) { // case potter walk on the evil and left it
            this.board[this.get_x()][this.get_y()] = 'evil';
          } else this.board[this.get_x()][this.get_y()] = 'blank';
          this.potterPos = this.get_new_positions();
          this.board[this.get_x()][this.get_y()] = 'potter';
          break;
      }
    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    // Simulation done by students
    // async simulate() {
      // this.nOp = 0
      // simplify the names
      // let time = 500
      // let sleep = this.sleep
      // mouvement
      // let turn_left = setTimeout(() => {this.turn_left}, time)
      // let turn_right = setTimeout(() => {this.turn_right}, time)
      // let move = this.move //this.doTimeout(this.move, time)
      // booleans
      // let can_move = this.can_move
      // let is_in_front_of_enemy = this.is_in_front_of_enemy
      // let is_on_target = this.is_on_target
      // getter
      // let get_x = this.get_x
      // let get_y = this.get_y
      // let get_target_x = this.get_target_x
      // let get_target_y = this.get_target_y
      // let get_direction = this.get_direction
      // feeling of power
      // let destroy_dark_force = this.destroy_dark_force
      
      // TODO by students
      // while(!is_on_target()) {
      //   if(!this.is_in_front_of_enemy()) {
      //     move()
      //     await sleep(1000)
      //   }
      // }
      // destroy_dark_force()
      // END OF THE CODE
    // },
    async simulate() {
      // define a bunch of stuff locally for the call to 'eval' later on
      // we need to shut the linter up temporarily though
      
      /* eslint-disable no-unused-vars */

      let sleep = this.sleep
      let timeout_ms = this.timeout_ms

      let move = this.move
      let turn_left = this.turn_left
      let turn_right = this.turn_right

      let can_move = this.can_move
      let is_on_target = this.is_on_target
      let get_direction = this.get_direction
      let get_x = this.get_x
      let get_y = this.get_y
      let get_target_x = this.get_target_x
      let get_target_y = this.get_target_y
      let destroy_dark_force = this.destroy_dark_force

      /* eslint-enable no-unused-vars */

      // translate python source to javashit

      let python_source = document.getElementById("src-input").value
      python_source = python_source.replace(/ {4}/g, '\t')

      let lines = python_source.split("\n")
      let js_src = ""

      let prev_tabcount = 0;

      function translate_line(line) {
        let tabcount = 0

        for (let chr of line) {
          if (chr == '\t') tabcount++
          else break
        }

        line = line.substring(tabcount, line.length)

        if (line == "") {
          return
        }

        if (tabcount < prev_tabcount) {
          js_src += '}'.repeat(prev_tabcount - tabcount)
        }

        prev_tabcount = tabcount

        const VALID_STATEMENTS = [ "while", "if" ]
        let is_statement = false

        for (let statement of VALID_STATEMENTS) {
          if (line.search(statement) >= 0) {
            is_statement = true

            const DICTIONARY = { "elif": "else if", "and": "&&", "or": "||", "not": '!', "EAST": '"east"', "WEST": '"west"', "NORTH": '"north"', "SOUTH": '"south"' }
            let expression = line.replace(':', "").split(' ').map(token => DICTIONARY[token] || token)

            expression.splice(0, 1)
            expression = expression.join(' ')
        
            js_src += statement + '(' + expression + "){"
          }
        }

        if (!is_statement) {
          js_src += line
          
          if (js_src.length && js_src[js_src.length - 1]) {
            js_src += ';'
          }
        }
      }

      for (let line of lines) {
        translate_line(line)
      }

      js_src += '}'.repeat(prev_tabcount) // in case we don't end the python source by a newline...

      js_src = js_src.replace("can_move()", "lmfao()")
      js_src = js_src.replace("move()", "move();await sleep(timeout_ms)")
      js_src = js_src.replace("turn_left()", "turn_left();await sleep(timeout_ms)")
      js_src = js_src.replace("turn_right()", "turn_right();await sleep(timeout_ms)")
      js_src = js_src.replace("lmfao()", "can_move()")

      console.log(js_src)
      eval("(async() => {" + js_src + "})()") // a bit hacky but eh
    },
    async resolve() {
      if (this.potterPos == undefined || this.evilPos == undefined) return
      // do a BFS
      let visited = Array(12).fill(null).map(() => {return Array(16).fill(false)})  // Array(12).fill(Array(16).fill(false))
      let pathTo = Array(12).fill(null).map(() => {return Array(16).fill(undefined)}) // Array(12).fill(Array(16).fill(undefined))
      let queue = []
      queue.push({
        x : this.get_x(),
        y : this.get_y()
      })
      while (queue.length > 0) {
        let c = queue.shift()
        if (c.x == this.get_target_x() && c.y == this.get_target_y()) break; // evil found
        for (const direction in this.mouvementDirection) {
          let mouv = this.mouvementDirection[direction]
          let newX = mouv.x + c.x
          let newY = mouv.y + c.y
          if (!(newX < 0 || newX > 11 || newY < 0 || newY > 15) && (this.board[newX][newY] == 'blank' || this.board[newX][newY] == 'evil') && !visited[newX][newY]) {
            queue.push({
              x : newX,
              y : newY
            })
            visited[newX][newY] = true
            pathTo[newX][newY] = c
          }
        }
      }
      // remake the path
      let path = []
      let c = {
        x : this.get_target_x(),
        y : this.get_target_y()
      }
      while (pathTo[c.x][c.y] != undefined) {
        path.unshift(c)
        c = pathTo[c.x][c.y]
      }
      // move potter to the dark force
      for (let index = 0; index < path.length; index++) {
        const newCoord = path[index];
        // Update the direction
        let delta = {
          x : newCoord.x - this.get_x(),
          y : newCoord.y - this.get_y()
        }
        if (delta.x == this.get_mouvementDirection().x && delta.y == this.get_mouvementDirection().y)
          console.log("same pos");
        else if (Math.abs(delta.x) == Math.abs(this.get_mouvementDirection().x) && Math.abs(delta.y) == Math.abs(this.get_mouvementDirection().y)) {
          // 180 turn
          this.turn_right()
          await this.sleep(500)
          this.turn_right()
          await this.sleep(500)
        } else {
          // David Goodenough
          while (delta.x != this.get_mouvementDirection().x || delta.y != this.get_mouvementDirection().y) {
            this.turn_right()
          }
          await this.sleep(500)
        }
        
        this.setOnBoard(this.get_x(), this.get_y(), 'blank')
        this.setOnBoard(newCoord.x, newCoord.y, 'potter')
        await this.sleep(500)
      }
      this.destroy_dark_force()
    },
    // tools for building map
    setOnBoard(x, y, value) {
      if (value == 'potter') this.potterPos = {'x' : x, 'y' : y};
      if (value == 'evil') this.evilPos = {'x' : x, 'y' : y};
      this.board[x][y] = value;
    },
    setRectangleOnBoard(x_from, y_from, x_to, y_to, value) {
      if (x_from > x_to || y_from > y_to) throw Error('Avoid infinite loop')
      for (let x = x_from; x <= x_to; x++) {
        for (let y = y_from; y <= y_to; y++) {
          this.setOnBoard(x, y, value)
        }
      }
    },
    // Map's
    clearMap() {
      this.board = Array(12).fill(null).map(() => {return Array(16).fill('blank')})
    },
    setupMap1() {
      this.clearMap();

      this.currentDirection = 1;
      
      this.setOnBoard(8, 10, 'potter');
      this.setOnBoard(4, 3, 'evil');

      this.setOnBoard(2, 1, 'building')
      this.setOnBoard(7, 1, 'building')
      this.setOnBoard(3, 5, 'building')
      this.setOnBoard(9, 5, 'building')
      this.setOnBoard(3, 8, 'building')
      this.setOnBoard(5, 8, 'building')
      this.setOnBoard(10, 8, 'building')
      this.setOnBoard(1, 12, 'building')
      this.setOnBoard(10, 12, 'building')
      this.setOnBoard(4, 13, 'building')
      this.setOnBoard(8, 14, 'building')

      this.setOnBoard(10, 2, 'troll')
      this.setOnBoard(6, 4, 'troll')
      this.setOnBoard(1, 6, 'troll')
      this.setOnBoard(7, 6, 'troll')
      this.setOnBoard(1, 9, 'troll')
      this.setOnBoard(5, 11, 'troll')
      this.setOnBoard(2, 14, 'troll')
    },
    setupMap2() {
      this.clearMap();

      this.setRandomDirection();
      this.setOnBoard(6, 0, 'potter');
      this.setOnBoard(6, 15, 'evil');

      this.setOnBoard(10, 1, 'building')
      this.setOnBoard(6, 2, 'building')
      this.setOnBoard(1, 3, 'building')
      this.setOnBoard(8, 3, 'building')
      this.setOnBoard(4, 4, 'building')
      this.setOnBoard(3, 7, 'building')
      this.setOnBoard(9, 7, 'building')
      this.setOnBoard(6, 8, 'building')
      this.setOnBoard(4, 10, 'building')
      this.setOnBoard(2, 11, 'building')
      this.setOnBoard(8, 12, 'building')
      this.setOnBoard(5, 13, 'building')
      this.setOnBoard(1, 14, 'building')
      this.setOnBoard(10, 14, 'building')
    },
    setupMap3() {
      this.clearMap()
      this.currentDirection = 1

      this.setOnBoard(4, 3, 'potter')
      this.setOnBoard(9, 11, 'evil')

      this.setOnBoard(5, 1, 'building')
      this.setOnBoard(9, 1, 'building')
      this.setOnBoard(10, 1, 'building')
      this.setOnBoard(1, 2, 'building')
      this.setOnBoard(3, 2, 'building')
      this.setOnBoard(3, 3, 'building')
      this.setOnBoard(5, 3, 'building')
      this.setRectangleOnBoard(7, 3, 9, 3, 'building')
      this.setOnBoard(2, 5, 'building')
      this.setRectangleOnBoard(4, 5, 6, 5, 'building')
      this.setRectangleOnBoard(3, 7, 3, 9, 'building')
      this.setOnBoard(5, 7, 'building')
      this.setRectangleOnBoard(7, 7, 10, 7, 'building')
      this.setOnBoard(1, 8, 'building')
      this.setRectangleOnBoard(6, 9, 6, 12, 'building')
      this.setOnBoard(8, 9, 'building')
      this.setOnBoard(10, 10, 'building')
      this.setRectangleOnBoard(1, 11, 4, 11, 'building')
      this.setOnBoard(10, 11, 'building')
      this.setOnBoard(2, 13, 'building')
      this.setOnBoard(8, 13, 'building')
      this.setOnBoard(9, 13, 'building')
      this.setOnBoard(4, 14, 'building')
    },
    setupMap4() {
      this.clearMap()

      this.currentDirection = 1;
      this.setOnBoard(6, 6, 'potter')
      this.setOnBoard(6, 8, 'evil')

      this.setRectangleOnBoard(4, 7, 8, 7, 'building')
    },
    setupMap5() {
      this.clearMap()

      this.currentDirection = 1;
      this.setOnBoard(4, 6, 'potter')
      this.setOnBoard(6, 10, 'evil')

      this.setOnBoard(6, 6, 'building')
      this.setOnBoard(6, 8, 'building')
      this.setOnBoard(4, 10, 'building')
      this.setOnBoard(8, 10, 'building')
    },
    setupMap6() {
      this.clearMap()

      this.currentDirection = 1;
      this.setOnBoard(6, 3, 'potter')

      this.setRectangleOnBoard(6, 6, 6, 10, 'building')
      this.setRectangleOnBoard(1, 8, 4, 8, 'building')

      this.setOnBoard(2, 8, 'evil')
    },
    setupMap7() {
      this.clearMap()

      this.currentDirection = 1;
      this.setOnBoard(6, 3, 'potter')

      this.setRectangleOnBoard(6, 4, 6, 10, 'building')
      this.setOnBoard(3, 7, 'building')
      this.setOnBoard(4, 7, 'building')
      this.setOnBoard(8, 7, 'building')
      this.setOnBoard(9, 7, 'building')

      this.setOnBoard(6, 7, 'evil')
    }
  },
  beforeMount() {
    this.clearMap()
  },
  mounted() {
    window.addEventListener("keypress", e => {
      switch (e.key) {
        case 'z':
          this.move()
          break;
        case 'q':
          this.turn_left()
          break;
        case 'd':
          this.turn_right()
          break;
        case 'a':
          this.destroy_dark_force()
          break;
        case 'b':
          this.toggleBuilding()
          break;
        case 't':
          this.toggleTroll()
          break;
        case 'h':
          this.togglePotter()
          break;
        case 'e':
          this.toggleEvil()
          break;
        case 'r':
          this.toggleRemove()
          break;
        default:
          break;
      }
    })
  }
}
</script>

<style>
#app {
  margin: auto;
}
.rows {
  display: flex;
  flex-direction: column; /* nique la logique */
}
.cols {
  display: flex;
  flex-direction: row; /* nique la logique */
}
.case-container {
  margin: 2px;
  padding: 0px;
}
</style>
