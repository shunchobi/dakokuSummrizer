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

// サンプルデータ
// 2024年11月29日	近藤駿一		19:43	"recog
// "
// 2024年12月02日	近藤駿一	9:49		
// 2024年12月20日	萌々子清水萌々子	10:04		打刻遅くなりました！10:00です
// 2024年12月20日	萌々子清水萌々子		19:01	
// 2024年12月25日	萌々子清水萌々子	10:30		"JR東海の遅延のため
// ▼証明書
// https://traininfo.jr-central.co.jp/zairaisen/delay_certificate.html?date=2024-12-25&line=10001&time=2"
// 2024年12月25日	萌々子清水萌々子		19:01	
// 2024年12月27日	萌々子清水萌々子	10:37		体調不良で遅刻しました。
// 2025年1月10日	萌々子清水萌々子		19:00	

const handleChangeTextarea = (e: Event) => {
  const target = e.target as HTMLTextAreaElement;
  const value = target.value;

  data.value = {} as Data;

  const __lines = value.split('\n');

  // コメントが改行されている場合の対応
  // if value of each line is not starting date format(yyyy年m月dd日\t) string, append to previous line and remove the line
  for (let i = 0; i < __lines.length; i++) {
    if (!__lines[i].match(/^\d{4}年\d{1,2}月\d{1,2}日\t/)) {
      __lines[i - 1] += __lines[i];
      __lines.splice(i, 1);
      i--;
    }
  }
  
  // 日付ごとにデータを整形
  const _lines = __lines.map((line) => {
    const data = line.split('\t');
    const date = new Date(data[0].replace('年', '-').replace('月', '-').replace('日', ''));
    const name = data[1];
    const inTime = toValue(data[2]);
    const outTime = toValue(data[3]);
    const note = toValue(data[4]);
    return {
      date,
      name,
      inTime,
      outTime,
      note,
    };
  });

  names.value = Array.from(new Set(_lines.map((line) => line.name)));

  // 日付の範囲を取得
  // generate dates from min to max
  const minDate = new Date(Math.min(..._lines.map((line) => line.date.getTime())));
  const maxDate = new Date(Math.max(..._lines.map((line) => line.date.getTime())));
  const datesArray = generateDateStrRange(getFormattedDate(minDate), getFormattedDate(maxDate));

  // 抜け日のデータを埋める
  // arrenge lines for filling empty data
  const lines = datesArray.map((date) => {
    const sameDateLines = _lines.filter((line) => getFormattedDate(line.date) === date);
    if (sameDateLines.length === 0) {
      return names.value.map((name) => {
        return {
          date: date,
          name: name,
          inTime: undefined,
          outTime: undefined,
          note: undefined,
        }
      });
    }
    return sameDateLines.map((line) => {
      return {
        date: getFormattedDate(line.date),
        name: line.name,
        inTime: line.inTime,
        outTime: line.outTime,
        note: line.note,
      }
    });
  }).flat();

  // 日付ごとにデータを表示するために整形
  lines.forEach((line) => {
    const date = line.date;
    const name = line.name;
    const inTime = line.inTime;
    const outTime = line.outTime;
    const note = line.note;

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
  if (value == '' || value == undefined) {
    return undefined;
  }
  return value;
};

const getFormattedDate = (date: Date | undefined): string => {
  if (!date) return "";
  const YYYY = String(date.getFullYear());
  const MM = String(date.getMonth() + 1).padStart(2, "0");
  const dd = String(date.getDate()).padStart(2, "0");
  return `${YYYY}/${MM}/${dd}`; // 2022/08/25
};

const generateDateStrRange = (
  startDateStr: string | undefined,
  endDateStr: string | undefined,
) => {
  if (!startDateStr || !endDateStr) return [];
  const dateRange = [];
  const currentDate = new Date(startDateStr);
  const endDate = new Date(endDateStr);

  while (currentDate <= endDate) {
    const dateStr = getFormattedDate(currentDate);
    dateRange.push(dateStr);
    currentDate.setDate(currentDate.getDate() + 1);
  }

  return dateRange;
}
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

th {
  border: 1px solid #000;
  padding: .5rem;
  min-width: 75px;
}

td {
  border-left: 1px solid #000;
  border-right: 1px solid #000;
  padding: .5rem;
  min-width: 75px;
  border-bottom: 1px dotted #000;
}
</style>