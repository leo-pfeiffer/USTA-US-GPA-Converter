<template>
  <div class="hello">
    <h3>{{ msg }}</h3>
    <p>
      The conversion scale as provided by the University of St Andrews can be
      found
      <a
        href="https://www.st-andrews.ac.uk/administration/academicdatateam/assessmentandawards/undergraduateinformation/"
        target="_blank"
        rel="noopener"
        >here</a
      >.
    </p>
    <hr />
    <p>Please enter your St Andrews GPA:</p>
    <input
      v-model="staGPA"
      @input="validateInput"
      type="number"
      placeholder="St Andrews GPA"
    />
    <p v-if="state === 'show'">
      Your St Andrews GPA of {{ staGPA }} corresponds to a US GPA of
      {{ round(convertToUS(Number(staGPA))) }}.
    </p>
    <p v-if="state === 'nan'">Please enter a valid GPA between 7 and 20.</p>
    <p v-if="state === 'lowGPA'">
      Your GPA must be higher than 7 for the conversion.
    </p>
    <p v-if="state === 'hidden'"></p>
    <p v-if="state === 'highGPA'">Your GPA cannot exceed 20.</p>
    <hr />
    <table>
      <thead>
        <col />
        <colgroup span="2"></colgroup>
        <colgroup span="2"></colgroup>
        <tr>
          <th></th>
          <th colspan="2" scope="colgroup">St Andrews GPA range</th>
          <th colspan="2" scope="colgroup">US GPA range</th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <thead>
        <tr>
          <th>Classification</th>
          <th>from</th>
          <th>to</th>
          <th>from</th>
          <th>to</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="c in conversion" v-bind:key="c.class">
          <td>{{ c.class }}</td>
          <td>{{ c.stALB }}</td>
          <td>{{ c.stAUB }}</td>
          <td>{{ c.USLB }}</td>
          <td>{{ c.USUB }}</td>
        </tr>
      </tbody>
    </table>
    <hr />
  </div>
</template>

<script>
export default {
  name: "Converter",
  props: {
    msg: String,
    conversion: Array,
  },
  methods: {
    validateInput() {
      console.log(this.staGPA);
      if (this.staGPA === "") {
        this.state = "hidden";
      } else if (isNaN(this.staGPA)) {
        this.state = "nan";
      } else {
        this.staGPA = Number(this.staGPA);
        if (this.staGPA < 7) {
          this.state = "lowGPA";
        } else if (this.staGPA > 20) {
          this.state = "highGPA";
        } else {
          this.state = "show";
        }
      }
      return;
    },
    convertToUS(gpa) {
      if (isNaN(gpa)) {
        return;
      }
      let idx = this.getConversionIndex(gpa);
      if (idx === -1) {
        return;
      }
      let stARange = this.calcStAndrewsRange(idx);
      let usRange = this.calcUSRange(idx);
      let dist = gpa - this.conversion[idx].stALB;
      let quotient = dist / stARange;

      return this.conversion[idx].USLB + quotient * usRange;
    },
    getConversionIndex(gpa) {
      for (let c in this.conversion) {
        let cls = this.conversion[c];
        if (cls.stALB <= gpa && gpa <= cls.stAUB) {
          return c;
        }
      }
      return -1;
    },
    calcUSRange(idx) {
      let lb = this.conversion[idx].USLB;
      let ub = this.conversion[idx].USUB;
      return this.calcRange(lb, ub);
    },
    calcStAndrewsRange(idx) {
      console.log(idx);
      console.log(this.conversion[idx]);
      let lb = this.conversion[idx].stALB;
      let ub = this.conversion[idx].stAUB;
      return this.calcRange(lb, ub);
    },
    calcRange(lb, ub) {
      return ub - lb;
    },
    round(num) {
      return Math.round((num + Number.EPSILON) * 100) / 100;
    },
  },
  data: function () {
    return {
      staGPA: "",
      state: "hidden",
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
table {
  display: inline-block;
  margin: 0 10px;
  text-align: right;
  overflow-x: auto;
  max-width: 100%;
  white-space: nowrap;
}
th,
td {
  padding: 0 15px;
}
a {
  color: #42b983;
}
</style>
