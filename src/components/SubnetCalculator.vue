<template>
  <div class="subnet-calculator">
    <h2>Калькулятор подсетей</h2>

    <label>
      IP-адрес:
      <input
        v-model="ipInput"
        type="text"
        placeholder="192.168.1.150"
        :class="{ 'input-error': !isValid && ipInput !== '' }"
        @keyup.enter="calculate"
      />
    </label>

    <label>
      Маска подсети:
      <select v-model="selectedMask">
        <option v-for="opt in MASK_OPTIONS" :key="opt.value" :value="opt.value">
          {{ opt.label }}
        </option>
      </select>
    </label>

    <button
      @click="calculate"
      :disabled="!isValid"
      type="button"
    >
      Рассчитать
    </button>

    <div v-if="result" class="result">
      <p><strong>IP:</strong> {{ ipInput }}</p>
      <p><strong>Маска:</strong> {{ selectedMask }}</p>
      <p><strong>Адрес сети:</strong> {{ result.network }}</p>
      <p><strong>Количество хостов:</strong> {{ result.hosts }}</p>
    </div>

    <p v-if="!isValid && ipInput !== ''" class="error-text">
      Некорректный IP-адрес
    </p>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { isIpValid } from '../utils/validation';
import { getNetworkAddress, getAddressesCount } from '../utils/network';
import { MASK_OPTIONS } from '../utils/masks';

const ipInput = ref('');
const selectedMask = ref(MASK_OPTIONS[24].value); // по умолчанию /24
const result = ref<{ network: string; hosts: number } | null>(null);

const isValid = computed(() => isIpValid(ipInput.value));

function calculate() {
  if (!isValid.value) return;

  const network = getNetworkAddress(ipInput.value, selectedMask.value);
  const hosts = getAddressesCount(selectedMask.value);

  result.value = { network, hosts };
}
</script>

<style scoped lang="scss">
.subnet-calculator {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid var(--color-gray);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

input,
select,
button {
  padding: 8px;
  font-size: 16px;
  border: 1px solid var(--color-gray);
  border-radius: 4px;
}

.input-error {
  border-color: var(--color-error);
}

button:disabled {
  background-color: var(--color-gray);
  cursor: not-allowed;
}

.result {
  background-color: #f9f9f9;
  padding: 12px;
  border-radius: 6px;
}

.error-text {
  color: var(--color-error);
  font-size: 14px;
}
</style>