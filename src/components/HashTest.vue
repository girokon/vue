<template>
  <div>
    <div>
      <label for="source">Source</label>
      <input name="source" id="source" v-model="source" />
      <pre>
          {{ result.b.toString(16) }} - {{ currentGroup.one }} {{
          maxHash >= result.b ? "ok" : "bad"
        }}
          {{ result.c.toString(16) }} - {{ currentGroup.two }} {{
          maxHash >= result.c ? "ok" : "bad"
        }}
      </pre>
    </div>
    <div>
      <label for="groups">Groups</label>
      <input name="groups" id="groups" v-model="groups" />
    </div>
    <div>
      <label for="cycles">Cycles</label>
      <input name="cycles" id="cycles" v-model="cycles" />
      <button @click="research">make science</button>
    </div>
    <div v-if="stat">
      <ul>
        <li v-for="group in stat.all" :key="group">Group {{group}} - {{stat.groupsPercent[group]}}%</li>
      </ul>
    </div>
  </div>
</template>

<script>
import { getHashGroup, Jenkins } from "../lib/hash";

const MAX_HASH = 0xffffffff;

const makeIdFromTime = () => {
  return +new Date() + "" + (Math.floor(Math.random() * 8888888) + 1111111);
};

export function getRandomUInt64() {
  let uint64 = "";
  const possible = "0123456789";

  for (let i = 0; i < 19; i++) {
    uint64 += possible.charAt(Math.floor(Math.random() * possible.length));
  }

  return uint64;
}

export default {
  name: "HashTest",
  data() {
    return {
      source: "",
      groups: 2,
      cycles: 1000000,
      stat: null
    };
  },
  computed: {
    result() {
      return Jenkins.hashlittle2(this.source, 0, 0);
    },
    currentGroupSize() {
      return Math.trunc(MAX_HASH / parseInt(this.groups, 10));
    },
    currentGroup() {
      const hash1 = this.result.b;
      const hash2 = this.result.c;
      return {
        one: getHashGroup(hash1, parseInt(this.groups, 10)),
        two: getHashGroup(hash2, parseInt(this.groups, 10))
      };
    },
    maxHash() {
      return MAX_HASH;
    }
  },
  methods: {
    research() {
      const stat = {
        groups: {},
        groupsPercent: {}
      };
      const numGroups = parseInt(this.groups, 10);
      let cycles = parseInt(this.cycles, 10);
      for (let i = 0; i < cycles; i++) {
        const item = makeIdFromTime();
        const hash = Jenkins.hashlittle2(item, 0, 0);

        const group = getHashGroup(hash.b, numGroups);
        stat.groups[group] = (stat.groups[group] || 0) + 1;
      }

      Object.keys(stat.groups).forEach(group => {
        stat.groupsPercent[group] = Math.trunc(stat.groups[group] / cycles * 10000) / 100
      });

      stat.all = Object.keys(stat.groups);

      this.stat = stat;
    }
  }
};
</script>
