<template>
  <img alt="University of St Andrews Crest" src="./assets/crest.png" />
  <Converter
    :msg="'University of St Andrews degree classification to US GPA conversion'"
    :conversion="[
      {
        class: '1st class',
        stALB: 17.5,
        stAUB: 20,
        USLB: 4.0,
        USUB: 4.0,
      },
      {
        class: '1st class',
        stALB: 16,
        stAUB: 17.4,
        USLB: 3.7,
        USUB: 3.9,
      },
      {
        class: '2nd class (division one)',
        stALB: 13,
        stAUB: 16.4,
        USLB: 3,
        USUB: 3.6,
      },
      {
        class: '2nd class (division two)',
        stALB: 10,
        stAUB: 13.4,
        USLB: 2.3,
        USUB: 2.9,
      },
      {
        class: '3rd class',
        stALB: 7,
        stAUB: 10.4,
        USLB: 1.3,
        USUB: 2.2,
      },
    ]"
  />
</template>

<script>
import ConverterVue from "./components/Converter.vue";
export default {
  name: "App",
  components: {
    Converter: ConverterVue,
  },
  data: function () {
    return {
      conversion: [
        {
          hello: "first_name",
          showname: "First Name",
          type: "text",
          required: "false",
          fee: "0",
        },
      ],
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

img {
  max-width: 30%;
  min-width: 75px;
}
</style>
