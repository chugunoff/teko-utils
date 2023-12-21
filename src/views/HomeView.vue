<template>
  <main>
    <div class="form">
      <div class="flex flex-row flex-wrap gap-9">
        <div>
          <Badge class="mb-2">Обозначение</Badge>
          <div class="flex flex-row gap-2">
            <div class="w-20">
              <Input v-model="groupDescription" type="text" placeholder="Группа" />
            </div>
            <div class="w-28">
              <Input v-model="numberDescription" type="text" placeholder="Номер" />
            </div>
          </div>
        </div>

        <div>
          <Badge class="mb-2">Генерация</Badge>
          <div class="flex flex-row gap-2">
            <Button variant="outline" @click="copyText">copy</Button>
            <Button variant="outline" @click="() => generateTxt(true)">new tab</Button>
            <Button variant="outline" @click="() => generateTxt(false)">.txt</Button>
          </div>
        </div>
      </div>

      <div class="overflow-auto">
        <div class="flex flex-row items-center gap-3 mb-2">
          <Badge>Переменные</Badge>

          <Button size="icon" class="w-6 h-6 rounded-full" @click="addTV">
            <Plus class="w-4 h-4" />
          </Button>
        </div>

        <div class="flex flex-row flex-nowrap gap-3">
          <div
            v-for="(tableVariable, tvIndex) in tableVariables"
            :key="tvIndex"
            class="flex flex-col gap-2 min-w-56 w-56"
          >
            <div class="mb-2">
              <Label for="email">Название переменной</Label>
              <div class="flex flex-row gap-1">
                <Input v-model="tableVariable.name" placeholder="Название переменной" />

                <Button variant="ghost" size="icon" @click="() => deleteTV(tvIndex)">
                  <X class="w-3 h-3" />
                </Button>
              </div>
            </div>

            <div>
              <Label for="email">Значения переменной</Label>

              <div class="flex flex-col gap-2">
                <div
                  v-for="(tvValue, tvvIndex) in tableVariable.values"
                  :key="tvvIndex"
                  class="flex flex-row gap-1"
                >
                  <Input
                    :id="tableVariable.uid + '-tvValue' + tvvIndex"
                    :modelValue="tvValue"
                    @update:model-value="
                      (value) => updateTVValue(tvIndex, tvvIndex, value as string)
                    "
                    @keyup.enter="() => addTVValue(tvIndex, tvvIndex, tableVariable.uid)"
                  />

                  <Button
                    variant="ghost"
                    size="icon"
                    @click="() => deleteTVValue(tvIndex, tvvIndex)"
                  >
                    <X class="w-3 h-3" />
                  </Button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, nextTick, ref } from 'vue';
import { nanoid } from 'nanoid';
import type { TableVariable } from '@/types/app';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { X, Plus } from 'lucide-vue-next';

const DIVIDER = ';;;';
const ORGANIZATION = 'ТЭФ';

const groupDescription = ref<string>('');
const numberDescription = ref<string>('');
const tableVariables = ref<TableVariable[]>([
  // {
  //   uid: '1',
  //   name: 'd',
  //   values: ['15', '20', '25', '3', '4', '5', '6', '7'],
  // },
  // {
  //   uid: '2',
  //   name: 'h',
  //   values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  // },
]);
const resultTxt = ref<string>('');

const baseDescription = computed<string>(() => {
  return ORGANIZATION + '.' + groupDescription.value + '.' + numberDescription.value + '.';
});

function addTV(): void {
  tableVariables.value.push({
    uid: nanoid(),
    name: '',
    values: [''],
  });
}

function deleteTV(tvIndex: number): void {
  tableVariables.value = tableVariables.value.filter((_, index) => index !== tvIndex);
}

function addTVValue(tvIndex: number, tvvIndex: number, tvUid: string): void {
  if (tvvIndex !== tableVariables.value[tvIndex].values.length - 1) {
    return;
  }

  const newLength = tableVariables.value[tvIndex].values.push('');

  nextTick(() => {
    document.getElementById(tvUid + '-tvValue' + (newLength - 1))?.focus();
  });
}

function updateTVValue(tvIndex: number, tvvIndex: number, value: string): void {
  tableVariables.value[tvIndex].values[tvvIndex] = value;
}

function deleteTVValue(tvIndex: number, tvvIndex: number): void {
  if (tableVariables.value[tvIndex].values.length === 1) {
    tableVariables.value[tvIndex].values[tvvIndex] = '';
  } else {
    tableVariables.value[tvIndex].values = tableVariables.value[tvIndex].values.filter(
      (_, index) => index !== tvvIndex,
    );
  }
}

function unionAll(): void {
  const result: string[] = [];

  // Обработать переменные
  const firstColumnValues = tableVariables.value[0].values;

  for (let i = 0; i < firstColumnValues.length; i++) {
    next(1, firstColumnValues[i]);
  }

  function next(columnIndex: number, upValue: string) {
    for (let i = 0; i < tableVariables.value[columnIndex].values.length; i++) {
      const value = tableVariables.value[columnIndex].values[i].trim();

      if (tableVariables.value[columnIndex + 1]) {
        next(columnIndex + 1, upValue + DIVIDER + value);
      } else {
        result.push(upValue + DIVIDER + value);
      }
    }
  }

  // Добавить базовое обозначение
  // ТЭФ.08.999.[00.000]

  if (result.length > 99999) {
    alert('АЛЯРМ БОЛЬШЕ 100 000');

    return;
  }

  for (let i = 0; i < result.length; i++) {
    result[i] = baseDescription.value + formatIndex(i) + DIVIDER + result[i];
  }

  // Добавить заголовки столбцов
  const headers = tableVariables.value.map((item) => item.name).join(DIVIDER);
  result.unshift('Обозначение' + DIVIDER + headers);

  resultTxt.value = result.join('\n');
}

function formatIndex(index: number): string {
  const number = index + 1;
  let result = '';

  if (number < 10) {
    result = '00.00' + number;
  } else if (number < 100) {
    result = '00.0' + number;
  } else if (number < 1000) {
    result = '00.' + number;
  } else if (number < 10000) {
    const ost = number % 1000;
    const cel = (number - ost).toString().substring(0, 1);

    result = '0' + cel + '.' + addBeforeZero(ost, 3);
  } else {
    const ost = number % 1000;
    const cel = (number - ost).toString().substring(0, 2);

    result = cel + '.' + addBeforeZero(ost, 3);
  }

  return result;
}

function addBeforeZero(value: number, resultSymbolCount: number): string {
  let result = value.toString();

  while (result.length < resultSymbolCount) {
    result = '0' + result;
  }

  return result;
}

function copyText(): void {
  unionAll();

  navigator.clipboard.writeText(resultTxt.value);
}

function generateTxt(newTab: boolean): void {
  unionAll();

  const blob = new Blob([resultTxt.value], { type: 'text/plain;charset=utf8' });

  const url = URL.createObjectURL(blob);

  if (newTab) {
    window.open(url);
  } else {
    const aEl = document.createElement('a');
    document.body.appendChild(aEl);
    aEl.style.display = 'none';
    aEl.download = `${
      ORGANIZATION + '.' + groupDescription.value + '.' + numberDescription.value
    }.txt`;
    aEl.href = url;
    aEl.click();
    aEl.remove();
  }
}
</script>

<style scoped>
.form {
  display: flex;
  flex-direction: column;
  gap: 16px;

  padding: 12px;
  margin: auto;
}
</style>
