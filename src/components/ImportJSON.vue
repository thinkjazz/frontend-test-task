<template>
    <div class="upload-btn-wrapper">
        <button
                type="button"
                class="btn waves-effect waves-light"
                >
            <input
                    @change="loadingJSON($event)"
                    type="file"
                    name="file"
                    id="file"
                    :accept="mimeType"

            />
            <i class="material-icons right">cloud_upload</i>Импорт JSON


        </button>


    </div>


</template>

<script lang="ts">

  import { Component, Vue } from 'vue-property-decorator';
  import { IApplication, ITaskItems } from '@/interfaces/IApplication';
  import { namespace } from 'vuex-class';
  const tasker = namespace('tasker');

  @Component
  export default class ImportJSON extends Vue {
    private readonly mimeType = 'application/json';
    private readonly fileFormat = 'json';
    @tasker.State
    public name!: string;

    @tasker.State
    private taskItems!: ITaskItems[];

    private async loadingJSON (event: any): Promise<void> {
      const json = event.target.files[0];
      const jsonData = {
        taskItems: this.taskItems,
      };
      if (!ImportJSON.checkFormatFile(json)) {
        this.errorMsg('Импортировать можно только JSON');
        this.fetchJson(this.name, jsonData);

        return;
      }
      const data = await this.parseJson(json);
      if (data == null) {
        this.fetchJson(this.name, jsonData);

        return;
      }
      const jsonName = json.name.replace(`.${this.fileFormat}`, '');
      this.fetchJson(jsonName, data as object);
    }

    private fetchJson (jsonName: IApplication['name'], data: object): void {
      this.$emit('fetchJson', {
        jsonName: jsonName,
        ...data,
      });
    }

    private async parseJson (file: Blob): Promise<{taskItems: ITaskItems[]} | null> {
      try {
        const strJSON = await this.readJson(file);
        const data = JSON.parse(strJSON);
        if (!this.isValidTaskItem(data)) {
          this.errorMsg('Ошибка структуры');

          return null;
        }
        if (this.isTasksNotEmpty(data.taskItems) && !this.isValidTasks(data.taskItems)) {
          this.errorMsg('Ошибка структуры');

          return null;
        }

        return {
          taskItems: data.taskItems,
        };
      } catch (error) {
        this.errorMsg(`Ошибка парсинга: ${error}`);

        return null;
      }
    }

    private static checkFormatFile (file: Blob): boolean {
      return file.type === 'application/json';
    }

    private isValidTasks (taskItems: []): boolean {
      for (let item of taskItems) {
        if (!this.checkingTask(item)) {

          return false;
        }
      }

      return true;
    }

    private errorMsg (message: string): void {
      this.$store.commit('onError', { show: true, message });
    }

    private readJson (file: Blob): Promise<string> {
      return new Promise((resolve, reject) => {
        const readJsonFile = new FileReader();
        readJsonFile.readAsText(file);
        readJsonFile.onload = () => {
          if (typeof readJsonFile.result === 'string') {
            resolve(readJsonFile.result);
          } else {
            resolve('');
          }
        };

        return readJsonFile.onerror = function (error) {
          return reject(error);
        };
      });
    }

    public isValidTaskItem (value: IApplication): boolean {
      return value && 'taskItems' in value && Array.isArray(value.taskItems);
    }

    public isTasksNotEmpty (items: []): boolean {
      return Object.keys(items).length > 0;
    }

    public checkingTask (item: ITaskItems): boolean {
      if (item) {

        return ('name' in item && true) && ('description' in item && true) && ('isChecked' in item && true) && ('enclosedTaskItem' in item && Array.isArray(item.enclosedTaskItem));
      }

      return false;
    }
  }
</script>

<style scoped>
    .upload-btn-wrapper {
        /*display: none;*/
        position: relative;
        overflow: hidden;
        display: contents;
    }

    .upload-btn-wrapper input[type=file] {
        font-size: 100px;
        position: absolute;
        left: 0;
        top: 0;
        opacity: 0;
    }
    .btn {
        background-color:#feca57;
        box-shadow: none;
        -webkit-box-shadow: none;
    }
    .btn:hover {
        background-color: #f1c054;
    }
</style>
