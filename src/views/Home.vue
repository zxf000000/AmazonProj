<template>
  <div class="home">
    <v-file-input
      truncate-length="15"
      label="选择文件"
      @input="fileInput"
      @change="change"
    ></v-file-input>
    <v-btn
      @click="exportData"
    >
      Export
    </v-btn>
<!--    <v-simple-table>-->
<!--      <template v-slot:default>-->
<!--        <thead>-->
<!--        <tr>-->
<!--          <th class="text-left">-->
<!--            Name-->
<!--          </th>-->
<!--          <th class="text-left">-->
<!--            Count-->
<!--          </th>-->
<!--        </tr>-->
<!--        </thead>-->
<!--        <tbody>-->
<!--        <tr-->
<!--          v-for="item in result"-->
<!--          :key="item.name"-->
<!--        >-->
<!--          <td>{{ item.name }}</td>-->
<!--          <td>{{ item.value }}</td>-->
<!--        </tr>-->
<!--        </tbody>-->
<!--      </template>-->
<!--    </v-simple-table>-->
    <v-data-table
      :headers="headers"
      :items="result"
      :items-per-page="5"
      class="elevation-1"
    ></v-data-table>
  </div>
</template>

<script>
// @ is an alias to /src
import ExcelJS from 'exceljs';
import { saveAs } from 'file-saver';

export default {
  name: 'Home',
  components: {
  },
  data() {
    return {
      file: null,
      headers: [
        {
          text: 'Word',
          align: 'start',
          sortable: false,
          value: 'name',
        },
        { text: 'Count', value: 'value' },
      ],
      result: [],
    };
  },
  methods: {
    exportData() {
      const workbook = new ExcelJS.Workbook();
      const worksheet = workbook.addWorksheet('sheet1');
      worksheet.columns = [
        { header: 'Word', key: 'name', width: 10 },
        { header: 'Count', key: 'value', width: 32 },
      ];
      // eslint-disable-next-line no-plusplus
      for (let i = 0; i < this.result.length; i++) {
        worksheet.addRow(this.result[i]);
      }
      workbook.xlsx.writeBuffer().then((buffer) => {
        // eslint-disable-next-line no-undef
        saveAs(new Blob([buffer], {
          type: 'application/octet-stream',
        }), 'result.xlsx');
      });
    },
    async change(file) {
      const res = {};
      this.file = file;
      const workbook = new ExcelJS.Workbook();
      await workbook.xlsx.load(file);
      const worksheet = workbook.getWorksheet(1);
      const col3 = worksheet.getColumn(3).values;
      const col4 = worksheet.getColumn(4).values;
      // eslint-disable-next-line no-plusplus
      for (let i = 1; i < col3.length; ++i) {
        const texts = col3[i];
        const count = col4[i];
        const arr = texts.split(' ');
        // eslint-disable-next-line no-plusplus
        for (let j = 0; j < arr.length; j++) {
          const arr1 = arr[j].replace(' ', '').split(',');
          // eslint-disable-next-line no-plusplus
          for (let x = 0; x < arr1.length; x++) {
            const word = arr1[x].toLowerCase();
            if (word.length > 0) {
              if (typeof res[word] === 'undefined') {
                // eslint-disable-next-line radix
                res[word] = parseInt(count);
              } else {
                // eslint-disable-next-line radix
                res[word] += parseInt(count);
              }
            }
          }
        }
      }
      const arr = [];
      Object.keys(res).forEach((key) => {
        arr.push({ name: key, value: res[key] });
      });
      this.result = arr;
    },
    fileInput() {
      console.log(this.file);
    },
  },
};
</script>
