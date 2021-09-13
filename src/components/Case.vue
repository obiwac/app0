<template>
  <div>
      <!-- <div :class="getClass"></div> -->
      <div v-if="content == 'blank'" class="content blank"></div>
      <div v-else-if="content == 'troll'" class="content troll"></div>
      <div v-else-if="content == 'building'" class="content building"></div>
      <div v-else-if="content == 'potter' & direction == 'east'" class="content potter"></div>
      <div v-else-if="content == 'potter' & direction == 'north'" class="content potter north"></div>
      <div v-else-if="content == 'potter' & direction == 'west'" class="content potter west"></div>
      <div v-else-if="content == 'potter' & direction == 'south'" class="content potter south"></div>
      <div v-else-if="content == 'evil'" class="content evil"></div>
  </div>
</template>

<script>
export default {
  name: 'Case',
  props: {
    content: String,
    direction : String
  }
}
</script>

<style scoped>
.content {
    width: 50px;
    height: 50px;
    margin: auto;
    background-size: 50px, 50px;
}
.blank {
    background-image: url("../assets/blank.png");
}
.troll {
    background-image: url("../assets/troll.png");
}
.building {
    background-image: url("../assets/building.png");
}
.potter {
    background-image: url("../assets/potter.png");
}
.north {
    transform: rotate(270deg);
}
.west {
    transform:scaleX(-1)
}
.south {
    transform: rotate(90deg);
}
.evil {
    background-image: url("../assets/evil.png");
}
</style>
