<template>
  <main>
    <div class="form">
      <div>
        <Badge class="badge">Базовое значение</Badge>
        <Input v-model="baseDescription" type="text" placeholder="Базовое значение" />
      </div>

      <div class="variables">
        <Badge class="badge">Переменные</Badge>

        <div class="flex flex-col gap-3">
          <div
            v-for="(tableVariable, tvIndex) in tableVariables"
            :key="tvIndex"
            class="flex flex-row gap-2"
          >
            <div class="mb-2">
              <div class="flex flex-row gap-1">
                <Input v-model="tableVariable.name" placeholder="Название переменной" />

                <Button
                  v-if="tableVariable.values.length > 1"
                  variant="outline"
                  size="icon"
                  @click="() => deleteTV(tvIndex)"
                >
                  <Trash2 class="w-4 h-4" />
                </Button>
              </div>
            </div>

            <div class="flex flex-col gap-2">
              <div
                v-for="(tvValue, tvvIndex) in tableVariable.values"
                :key="tvvIndex"
                class="flex flex-row gap-1"
              >
                <Input
                  :id="tableVariable.name + '-tvValue' + tvvIndex"
                  :modelValue="tvValue"
                  @update:model-value="(value) => updateTVValue(tvIndex, tvvIndex, value as string)"
                  @keyup.enter="() => addTVValue(tvIndex, tvvIndex, tableVariable.name)"
                />

                <Button
                  v-if="tableVariable.values.length > 1"
                  variant="outline"
                  size="icon"
                  @click="() => deleteTVValue(tvIndex, tvvIndex)"
                >
                  <Trash2 class="w-4 h-4" />
                </Button>
              </div>
            </div>
          </div>

          <Button @click="addTV">Добавить переменную</Button>
        </div>
      </div>

      <Button @click="unionAll">Объеденить</Button>
    </div>

    <Textarea v-model="resultTxt" placeholder="Type your message here." />
  </main>
</template>

<script setup lang="ts">
import { nextTick, ref } from 'vue';
import { nanoid } from 'nanoid';
import type { TableVariable } from '@/types/app';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Trash2 } from 'lucide-vue-next';

const baseDescription = ref<string>('ТЭФ.08.999.');
const tableVariables = ref<TableVariable[]>([
  {
    uid: '1',
    name: 'd',
    values: ['15', '20', '25', '3', '4', '5', '6', '7'],
  },
  {
    uid: '2',
    name: 'h',
    values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  },
  {
    uid: '3',
    name: 'w',
    values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  },
  {
    uid: '4',
    name: 'g',
    values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  },
  {
    uid: '5',
    name: 'x',
    values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  },
  {
    uid: '5',
    name: 'x',
    values: ['3', '4', '5', '6', '7', '3', '4', '5', '6', '7'],
  },
]);
const resultTxt = ref<string>('');

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

function addTVValue(tvIndex: number, tvvIndex: number, tvName: string): void {
  if (tvvIndex !== tableVariables.value[tvIndex].values.length - 1) {
    return;
  }

  const newLength = tableVariables.value[tvIndex].values.push('');

  nextTick(() => {
    document.getElementById(tvName + '-tvValue' + (newLength - 1))?.focus();
  });
}

function updateTVValue(tvIndex: number, tvvIndex: number, value: string): void {
  tableVariables.value[tvIndex].values[tvvIndex] = value;
}

function deleteTVValue(tvIndex: number, tvvIndex: number): void {
  tableVariables.value[tvIndex].values = tableVariables.value[tvIndex].values.filter(
    (_, index) => index !== tvvIndex,
  );
}

function unionAll(): void {
  const result: string[] = [];

  const firstColumnValues = tableVariables.value[0].values;

  for (let i = 0; i < firstColumnValues.length; i++) {
    next(1, firstColumnValues[i]);
  }

  function next(columnIndex: number, upValue: string) {
    for (let i = 0; i < tableVariables.value[columnIndex].values.length; i++) {
      const value = tableVariables.value[columnIndex].values[i].trim();

      if (tableVariables.value[columnIndex + 1]) {
        next(columnIndex + 1, upValue + ';;;' + value);
      } else {
        result.push(upValue + ';;;' + value);
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
    result[i] = baseDescription.value + formatIndex(i) + ';;;' + result[i];
  }

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
</script>

<style scoped>
.form {
  display: flex;
  flex-direction: column;
  gap: 16px;

  width: 100%;
  max-width: 800px;

  padding: 12px;
  margin: auto;
}

.badge {
  margin-bottom: 8px;
}
</style>
