<template>
    <button type="button" class="btn waves-effect waves-light" @click="exportJSON">Export JSON
        <i class="material-icons right">cloud_download</i>
    </button>
</template>

<script lang="ts">
  import { Component, Prop, Vue } from 'vue-property-decorator';

  @Component
  export default class ExportJSON extends Vue {

    @Prop({ required: true, type: String })
    private jsonName!: string;

    @Prop({ required: true, type: String })
    private formatJson!: string;


    @Prop({ required: true, type: String })
    private data!: string;

    public exportJSON () {
      const data = this.data;
      const json = `${this.jsonName}.${this.formatJson}`;
      const blob = new Blob([data]);
      if (!window.navigator.msSaveOrOpenBlob) {
        const a = window.document.createElement('a');
        a.href = window.URL.createObjectURL(blob);
        a.download = json;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
      } else {
        window.navigator.msSaveBlob(blob, json);
      }
    }
  }
</script>

<style scoped>
    .btn {
        background-color:#54a0ff;
        box-shadow: none;
        -webkit-box-shadow: none;
    }
    .btn:hover {
        background-color: #adcfff;
    }
</style>
