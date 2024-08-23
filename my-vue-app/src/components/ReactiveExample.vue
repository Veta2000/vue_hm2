<template>
  <div>
    <h1>People List</h1>
    <input v-model="searchQuery" placeholder="Поиск по Имени, Фамилии" />
    
    <ul>
      <li v-for="person in filteredPeople" :key="person.id">
        {{ person.firstName }} {{ person.lastName }} - Возраст: {{ person.age }}
        <button @click="selectPerson(person)">Edit Age</button>
      </li>
    </ul>

    <button @click="addRandomPerson">Добавить случайного человека</button>

    <div v-if="selectedPerson">
      <h2> Добавлен возраст {{ selectedPerson.firstName }} {{ selectedPerson.lastName }}</h2>
      <input v-model="newAge" type="number" @input="validateAge" />
      <button @click="updateAge">Update Возраст</button>
      <p v-if="ageError" style="color:red;">{{ ageError }}</p>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, ref, watch } from 'vue';

// Изначальный список людей
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

// Обчислювальне свойство для фильтрации списка объектов
const filteredPeople = computed(() => {
  return people.filter(person => 
    person.firstName.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
    person.lastName.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// Метод для выбора человека и подготовки к изменению возраста
const selectPerson = (person) => {
  selectedPerson.value = person;
  newAge.value = person.age;
  ageError.value = '';
};

// Метод для изменения возраста с проверкой на соответствие правилам
const updateAge = () => {
  if (newAge.value >= 18) {
    selectedPerson.value.age = newAge.value;
    ageError.value = '';
  } else {
    alert('Возраст не может быть меньше 18 лет');
  }
};

// Метод для валидации возраста при вводе
const validateAge = (event) => {
  const age = event.target.value;
  if (age < 18) {
    alert('Возраст не может быть меньше 18 лет');
  } else {
    ageError.value = '';
  }
};

// Метод для добавления случайного человека из API
const addRandomPerson = async () => {
  try {
    const response = await fetch('https://randomuser.me/api/');
    const data = await response.json();
    const randomPerson = data.results[0];
    const newPerson = {
      id: Date.now(),
      firstName: randomPerson.name.first,
      lastName: randomPerson.name.last,
      age: Math.floor(Math.random() * 50) + 18  // Возраст от 18 до 67
    };
    people.push(newPerson);
  } catch (error) {
    console.error('Ошибка при получении случайного человека:', error);
  }
};



// Watcher для отслеживания изменений в selectedPerson
watch(selectedPerson, (newVal, oldVal) => {
  if (newVal) {
    console.log(`Выбран новый человек: ${newVal.firstName} ${newVal.lastName}`);
  }
});

// Watcher для отслеживания изменений возраста
watch(newAge, (newVal) => {
  if (newVal < 18) {
    ageError.value = 'Возраст не может быть меньше 18 лет';
  } else {
    ageError.value = '';
  }
});


// Watcher, который реагирует на изменения в computed property
watch(filteredPeople, (newVal) => {
  console.log('Отфильтрованный список обновлен', newVal);
});
</script>
