<template>
  <div>
    <h1>Список людей</h1>
    <input v-model="searchQuery" placeholder="Поиск по Имени, Фамилии" />

    <ul>
      <li v-for="person in sortedFilteredPeople" :key="person.id">
        {{ person.firstName }} {{ person.lastName }} - Возраст: {{ person.age }}
        <button @click="selectPerson(person)">Выбрать для редактирования</button>
      </li>
    </ul>

    <button @click="addRandomPerson">Добавить случайного человека</button>

    <div v-if="selectedPerson">
      <h2>Добавлен возраст {{ selectedPerson.firstName }} {{ selectedPerson.lastName }}</h2>
      <div v-if="!isAgeUpdated">
        <input v-model="newAge" type="number" @input="validateAge" />
        <button @click="updateAge">Обновить возраст</button>
      </div>
      <p v-if="ageError" :style="{ color: newAge.value < 18 ? 'red' : 'green' }">{{ ageError }}</p>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, ref, watch } from 'vue';

const people = reactive([
  { id: 1, firstName: 'Aiden', lastName: 'Miller', age: 24 },
  { id: 2, firstName: 'Emma', lastName: 'Johnson', age: 29 },
  { id: 3, firstName: 'Noah', lastName: 'Brown', age: 35 },
  { id: 4, firstName: 'Olivia', lastName: 'Williams', age: 22 },
  { id: 5, firstName: 'James', lastName: 'Taylor', age: 31 }
]);

const searchQuery = ref('');
const selectedPerson = ref(null);
const newAge = ref(null);
const ageError = ref('');
const isAgeUpdated = ref(false); // Флаг для отслеживания обновления возраста

const filteredPeople = computed(() => {
  return people.filter(person =>
    person.firstName.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    person.lastName.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

const sortedFilteredPeople = computed(() => {
  return filteredPeople.value.sort((a, b) => a.age - b.age);
});

watch(newAge, (newVal) => {
  if (newVal < 18) {
    ageError.value = 'Возраст не может быть меньше 18 лет';
  } else {
    ageError.value = '';
  }
});

const selectPerson = (person) => {
  selectedPerson.value = person;
  newAge.value = person.age;
  ageError.value = '';
  isAgeUpdated.value = false; // Сброс флага при выборе нового человека
};

const updateAge = () => {
  if (newAge.value >= 18) {
    selectedPerson.value.age = newAge.value;
    ageError.value = '';
    isAgeUpdated.value = true; // Устанавливаем флаг, чтобы скрыть поле ввода и кнопку
  } else {
    ageError.value = 'Возраст не может быть меньше 18 лет';
  }
};

const validateAge = (event) => {
  const age = event.target.value;
  if (age < 18) {
    ageError.value = 'Возраст не может быть меньше 18 лет';
  } else {
    ageError.value = '';
  }
};

const addRandomPerson = async () => {
  try {
    const response = await fetch('https://randomuser.me/api/');
    const data = await response.json();
    const randomPerson = data.results[0];
    const newPerson = {
      id: Date.now(),
      firstName: randomPerson.name.first,
      lastName: randomPerson.name.last,
      age: Math.floor(Math.random() * 50) + 18,
    };
    people.push(newPerson);
  } catch (error) {
    console.error('Ошибка при получении случайного человека:', error);
  }
};
</script>
