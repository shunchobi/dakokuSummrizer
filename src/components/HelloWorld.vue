<template>
  <h1>Dakoku Weaver 2</h1>
  <p>
    <a
      href="https://magictube365.sharepoint.com/:x:/r/sites/labor/_layouts/15/Doc.aspx?sourcedoc=%7B844A2BFE-7064-4CEA-B23B-6FF501CCD4A5%7D&file=%E5%87%BA%E9%80%80%E5%8B%A4%E6%99%82%E9%96%93%E8%A1%A8.xlsx&action=default&mobileredirect=true&DefaultItemOpen=1">paste
      table data here.</a>
  </p>
  <textarea @input="handleChangeTextarea"></textarea>
  <h2>Names</h2>
  <ul>
    <li v-for="name in names" :key="name">{{ name }}</li>
  </ul>

  <table>
    <thead>
      <tr>
        <th style="width: 150px;">date</th>
        <th v-for="name in names" colspan="2" style="width: 150px;" :key="name">{{ name }}</th>
      </tr>
    </thead>
    <tbody>
      <template v-for="date in Object.keys(data)" :key="date">
        <tr style="border-top: 1px solid #000;">
          <th rowspan="2">{{ date }}</th>
          <template v-for="name in names" :key="name">
            <!-- <td :style="{ backgroundColor: data[date][name] ? 'white' : 'red' }"> -->
            <td>
              <span>{{ data[date][name]?.inTime ?? "---" }}</span>
            </td>
            <td>
              <span>{{ data[date][name]?.inTimeNote ?? "" }}</span>
            </td>
          </template>
        </tr>
        <tr>
          <template v-for="name in names" :key="name">
            <td>
              <span>{{ data[date][name]?.outTime ?? "---" }}</span>
            </td>
            <td>
              <span>{{ data[date][name]?.outTimeNote ?? "" }}</span>
            </td>
          </template>
        </tr>
      </template>
      <!-- <tr>
        <th rowspan="2">2025年1月28日</th>
        <td style="border-bottom: 1px dotted;"><span>出勤時間</span></td>
        <td style="border-bottom: 1px dotted;"><span>出勤理由</span></td>
      </tr>
      <tr>
        <td style="border-top: 1px dotted;"><span>退勤時間</span></td>
        <td style="border-top: 1px dotted;"><span>退勤理由</span></td>
      </tr> -->
    </tbody>
  </table>
</template>

<script setup lang="ts">
import { ref } from 'vue';

type Data = {
  [date: string]: {
    [name: string]: {
      name: string;
      inTime: string | undefined;
      outTime: string | undefined;
      inTimeNote: string | undefined;
      outTimeNote: string | undefined;
    };
  };
}

const names = ref<string[]>([]);
const data = ref<Data>({})

const handleChangeTextarea = (e: Event) => {
  const target = e.target as HTMLTextAreaElement;
  const value = target.value;

  data.value = {} as Data;
  names.value = [];

  const lines = value.split('\n');

  // sep by tab
  // head1 head2 head3 head4 head5 (head1=date, head2=name, head3=inTime, head4=outTime, head5=note)
  // 2025年1月14日	勝田麻美		18:06	
  // 2025年1月14日	萌々子清水萌々子		19:00	
  // 2025年1月14日	近藤駿一		19:06	
  // 2025年1月14日	三井健史		19:10	
  // 2025年1月14日	寺川晃司		19:19	
  // 2025年1月15日	勝田麻美	9:04		フォームのエラーで打刻遅れました
  // 2025年1月15日	三井健史	9:19		
  // 2025年1月15日	萌々子清水萌々子	9:51		
  lines.forEach((line) => {
    const items = line.split('\t');
    const date = items[0];
    const name = items[1];
    const inTime = toValue(items[2]);
    const outTime = toValue(items[3]);
    const note = toValue(items[4]);

    if (!names.value.includes(name)) {
      names.value.push(name);
    }

    if (!data.value[date]) {
      data.value[date] = {};
    }

    if (!data.value[date][name]) {
      data.value[date][name] = {} as Data[string][string];
      if (inTime) {
        data.value[date][name].inTime = inTime;
        data.value[date][name].inTimeNote = note;
      }
      if (outTime) {
        data.value[date][name].outTime = outTime;
        data.value[date][name].outTimeNote = note;
      }
    } else {
      if (inTime) {
        data.value[date][name].inTime = inTime;
        data.value[date][name].inTimeNote = note;
      }
      if (outTime) {
        data.value[date][name].outTime = outTime;
        data.value[date][name].outTimeNote = note;
      }
    }
  })

  // sort by date then sort by name
  const sortedData = Object.fromEntries(
    Object.entries(data.value).sort(([a], [b]) => new Date(a).getTime() - new Date(b).getTime())
  );
  for (const key in sortedData) {
    sortedData[key] = Object.fromEntries(
      Object.entries(sortedData[key]).sort(([a], [b]) => a.localeCompare(b))
    );
  }
  data.value = sortedData;

  // sort by name
  names.value = names.value.sort(([a], [b]) => a.localeCompare(b));
};

const toValue = (value: string | undefined) => {
  if(value == '' || value == undefined) {
    return undefined;
  }
  return value;
};

</script>

<style scoped>
textarea {
  width: 100%;
  height: 200px
}

table {
  border-collapse: collapse;
  border: 1px solid #000
}

th{
  border: 1px solid #000;
  padding: .5rem;
  min-width: 75px;
}
td{
  border-left: 1px solid #000;
  border-right: 1px solid #000;
  padding: .5rem;
  min-width: 75px;
  border-bottom: 1px dotted #000;
}
</style>