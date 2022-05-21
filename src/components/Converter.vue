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
    <br>
    <input
        v-model="staMedian"
        @input="validateInput"
        type="number"
        placeholder="St Andrews Median"
    />
    <p v-if="state === 'show'">
      Your St Andrews GPA of {{ staGPA }} corresponds to a US GPA of
      {{ round(convertToUS(), 2) }}.
    </p>
    <p v-if="state === 'nan'">Please enter a valid GPA between 7 and 20.</p>
    <p v-if="state === 'lowGPA'">
      Your GPA must be higher than 7 for the conversion.
    </p>
    <p v-if="state === 'hidden'"></p>
    <p v-if="state === 'highGPA'">Your GPA cannot exceed 20.</p>
    <hr />
    <table>
      <colgroup>
        <col />
        <col span="2"/>
        <col span="2"/>
      </colgroup>
      <thead>
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
    <div style="justify-content: center; display: flex">
      <p style="width: 80%">
        ℹ️ If your GPA falls between two classifications (e.g. 13 is part of both 2:2 and 2:1),
        the classification is determined based on the median. The lower classification is chosen
        if the median falls into the lower classification, otherwise the higher one is chosen.
        If you don't specify the median, the higher classification is chosen.
      </p>
    </div>
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
      let hasMedian = (this.staMedian !== "")
      if ((this.staGPA === "")) {
        this.state = "hidden";
      } else if (isNaN(this.staGPA) || (hasMedian && isNaN(this.staMedian))) {
        this.state = "nan";
      } else {
        this.staGPA = this.round(Number(this.staGPA), 1);
        this.staMedian = hasMedian ? this.round(Number(this.staMedian), 1) : '';
        if (this.staGPA < 7 || (hasMedian && this.staMedian < 7)) {
          this.state = "lowGPA";
        } else if (this.staGPA > 20 || (hasMedian && this.staMedian > 20)) {
          this.state = "highGPA";
        } else {
          this.state = "show";
        }
      }
    },
    convertToUS() {
      let gpa = this.staGPA === '' ? Number.NaN : Number(this.staGPA)
      let median = this.staMedian === '' ? Number.NaN : Number(this.staMedian)
      if (isNaN(gpa)) {
        return;
      }
      let idx = this.getConversionIndex(gpa, median);
      if (idx === -1) {
        return;
      }
      let stARange = this.calcStAndrewsRange(idx);
      let usRange = this.calcUSRange(idx);
      let dist = gpa - this.conversion[idx].stALB;
      let quotient = dist / stARange;

      return this.conversion[idx].USLB + quotient * usRange;
    },
    getConversionIndices(gpa) {
      let indexes = []
      for (let c = 0; c < this.conversion.length; c++) {
        let cls = this.conversion[c];
        if (cls.stALB <= gpa && gpa <= cls.stAUB) {
          indexes.push(c)
        }
      }
      return indexes
    },
    getConversionIndex(gpa, median) {
      let gpaIdx = this.getConversionIndices(gpa)

      // unambiguous result based on GPA
      if (gpaIdx.length === 0) {
        return -1
      } else if (gpaIdx.length === 1) {
        return gpaIdx[0]
      } else if (gpaIdx.length > 2) {
        console.error("GPA falls in more than 2 classifications")
        return -1
      }

      // consider median
      if (median === '' || isNaN(median)) {
        return gpaIdx[0] // median not specified, use GPA result
      }
      let medIdx = this.getConversionIndices(median)
      if (medIdx.length === 0) {
        return -1
      } else if (medIdx.length > 2) {
        console.error("Median falls in none or more than 2 classifications")
        return -1
      }
      return medIdx[medIdx.length-1] > gpaIdx[0] ? gpaIdx[1] : gpaIdx[0]  // get best result from median
    },
    calcUSRange(idx) {
      let lb = this.conversion[idx].USLB;
      let ub = this.conversion[idx].USUB;
      return this.calcRange(lb, ub);
    },
    calcStAndrewsRange(idx) {
      let lb = this.conversion[idx].stALB;
      let ub = this.conversion[idx].stAUB;
      return this.calcRange(lb, ub);
    },
    calcRange(lb, ub) {
      return ub - lb;
    },
    round(num, d) {
      let exp = Math.pow(10, d);
      return Math.round((num + Number.EPSILON) * exp) / exp;
    },
  },
  data: function () {
    return {
      staGPA: "",
      staMedian: "",
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
