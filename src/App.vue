<template>
  <table>
    <thead>
    <tr>
      <th
          v-for="key in keys"
          :key="key"
          @click="Display(key)"
          style="cursor: pointer"
      >
        {{ key }}
      </th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="route in Routref" :key="route.uuid">
      <td v-for="key in keys" :key="key">
        {{ route[key] }}
      </td>
    </tr>
    </tbody>
  </table>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import type { Route } from './route'
import {BehaviorSubject} from "rxjs";
const keys: (keyof Route)[] = ['address', 'gateway', 'interface']

const routes: Route[] = [
  { uuid: '1', address: '10.1.30.0/24', mask: '0.255.255.0', gateway: '60.1.30.1', interface: 'eth0' },
  { uuid: '2', address: '192.168.0.0/16', mask: '1.255.0.0', gateway: '192.168.0.1', interface: 'eth1' },
  { uuid: '3', address: '172.16.0.0/12', mask: '4.240.0.0', gateway: '23.16.0.1', interface: 'euh2' },
  { uuid: '4', address: '10.0.0.0/8', mask: '10.0.0.0', gateway: '6.0.0.1', interface: 'eth0' },
  { uuid: '5', address: '192.168.10.0/24', mask: '0.255.255.0', gateway: '192.168.10.1', interface: 'eth1' },
  { uuid: '6', address: '172.20.10.0/24', mask: '3.255.255.0', gateway: '45.20.10.1', interface: 'wlan0' },
  { uuid: '7', address: '100.64.0.0/10', mask: '2.192.0.0', gateway: '100.64.0.1', interface: 'eth3' },
  { uuid: '8', address: '192.0.2.0/24', mask: '7.255.255.0', gateway: '198.0.2.1', interface: 'etk460' },
  { uuid: '9', address: '203.0.113.0/24', mask: '89.255.255.0', gateway: '203.0.113.1', interface: 'eth2' },
]

const routes$ = new BehaviorSubject<Route[]>(routes);
const original$ =new BehaviorSubject<Route[]>(routes);
const clicked$ = new BehaviorSubject<boolean>(false);

const Routref = ref<Route[]>(routes);
routes$.subscribe(routes => {
  Routref.value = routes;
})

//преобразование строки в число(инт)
function strToInt(str: string): number[] {
  return str.split('/')[0].split('.').map(Number);
}
//есть другая реализация с объединением в единую строку и сравнением их(если нужно реализуется очень легко)

// отображение содержимого
function Display(key: keyof Route) {
  const clicked = !clicked$.value;
  clicked$.next(clicked)
  const sort = [...routes$.value]
  // Одинаковая логика сортировки(оставляем работать вместе)
  if(key === 'address' || key === 'gateway') {
    if (clicked) {
      quickSort(sort, 0, sort.length - 1, key);
      routes$.next(sort);
      return;
    } else {
      routes$.next(original$.value);
      return;
    }
  }
  //сортировка по строкам(в алфавитном порядке)
  if (key === 'interface') {
      if (clicked) {
        sort.sort((a, b) => String(a[key]).localeCompare(String(b[key])));
        routes$.next(sort);
        return;
      } else {
        routes$.next(original$.value);
        return;
      }
    }
  //реализация того же самого кода но на JS(намного трудоемней и сложнее)
  // if (key === 'gateway') {
  //   if (clicked.value) {
  //     const temp = routes.value.map(route => ({
  //       ...route,
  //       gatewayValue: strToInt(String(route[key]))
  //     }));
  //
  //     quickSort(temp, 0, temp.length - 1, 'gatewayValue');
  //     routes.value = temp.map(({ gatewayValue, ...rest }) => rest);
  //   } else {
  //     routes.value = JSON.parse(JSON.stringify(original.value));
  //   }
  // }
  // if (key === 'interface') {
  //   if (clicked.value) {
  //     const temp = [...routes.value];
  //     temp.sort((a, b) => String(a[key]).localeCompare(String(b[key])));
  //     routes.value = temp;
  //   } else {
  //     routes.value = JSON.parse(JSON.stringify(original.value));
  //   }
  // }
}
//компонент для квиксорта
function partition(arr: any[], low: number, high: number, key: string) {
  const pivot = strToInt(String(arr[high][key]));
  let i = low - 1;

  for (let j = low; j <= high - 1; j++) {
    const current = strToInt(String(arr[j][key]));
    if (compareIpArrays(current, pivot) < 0) {
      i++;
      swap(arr, i, j);
    }
  }

  swap(arr, i + 1, high);
  return i + 1;
}
//сравнение массива опять же для квиксорта
function compareIpArrays(a: number[], b: number[]): number {
  for (let i = 0; i < 4; i++) {
    if (a[i] !== b[i]) return a[i] - b[i];
  }
  return 0;
}

function swap(arr: any[], i: number, j: number) {
  const temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}
//сам квиксорт
function quickSort(arr: any[], low: number, high: number, key: string) {
  if (low < high) {
    let pi = partition(arr, low, high, key);
    quickSort(arr, low, pi - 1, key);
    quickSort(arr, pi + 1, high, key);
  }
}

</script>

<style>
table{
  border-collapse: collapse;
}
thead{
  font-size: 15px;
  background-color: gray;
}
tbody{
  background-color: #fff;
}
table, th, td {
  border: 1px solid black;
}
td,tr{
  width: 800px;
  height: 50px;
  text-align: center;
  color: black;
}

</style>