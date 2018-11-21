
<template>
<!--
  Необходимо выполнить:
    1. добавление нового пользователя через форму над таблицей
    2. При клике на th делать сортировку списка по выбраному полю.
    Т.е. если по "Имя", то сортировать по имени, если по "Фамилия", то сортировка по фамилии.
    При клике по одному и тому же полю несколько раз подряд переключать сортировку по этому полю в порядке ASC => DESC => Выкл.

  Дополнительно, по желанию:
    3. <select> с выбором отображаемого количества элемента - 5, 10, 50.
    3. Пагинация, если часть элементов оказалась скрытой

  Для выполнения необходимо использовать computed-свойства
  -->

<div class="UsersControl">
  <form class="UsersControl__add-form" @submit="e => onFormSubmit(e)">
    <input v-model="form.name" type="text" placeholder="Имя">
    <input v-model="form.surname" type="text" placeholder="Фамилия">
    <button>Добавить</button>
  </form>

  <table class="UsersControl__table">
    <tr>
      <th @click="e => toggleSort('index')">№
        <span v-if="sort.prop === 'index'">
          {{sortDirectionLabel}}
        </span></th>
      <th @click="e => toggleSort('name')">
        Имя
        <span v-if="sort.prop === 'name'">
          {{sortDirectionLabel}}
        </span>
      </th>
      <th @click="e => toggleSort('surname')">
        Фамилия
        <span v-if="sort.prop === 'surname'">
          {{sortDirectionLabel}}
        </span>
      </th>
    </tr>
    <tr v-for="(user, idx) in paginatedUsers">
      <td v-text="user.index + 1"></td>
      <td v-text="user.name"></td>
      <td v-text="user.surname"></td>
    </tr>
  </table>

  <div class="UsersControl__page-control">
    <select v-model="page.size">
      <option>5</option>
      <option>10</option>
      <option>50</option>
    </select>
    <span @click="e => prevPage()">◄</span>
    <span>{{page.index + 1}}</span>
    <span>/</span>
    <span>{{totalPages}}</span>
    <span @click="e => nextPage()">►</span>
</div>
</div>
</template>
<script>
export default {
  data: () => ({
    page: { size: 5, index: 0 },
    form: { name: "", surname: "" },
    sort: { prop: "", desc: false },
    users: [{ name: "Пётр", surname: "Васильев" }]
  }),

  computed: {
    newUser() {
      return { name: this.form.name, surname: this.form.surname };
    },
    paginatedUsers() {
      const idx = this.page.index;
      const size = this.page.size;
      const skip = idx * size;
      return this.sortedUsers.slice(skip, skip + size);
    },
    sortedUsers() {
      const users = this.indexedUsers;
      const compare = this.comparer;
      return !compare ? users : users.sort(compare);
    },
    indexedUsers() {
      return this.users.map((x, i) => {
        return { index: i, name: x.name, surname: x.surname };
      });
    },
    sortDirectionLabel() {
      return this.sort.desc ? "↑" : "↓";
    },
    comparer() {
      const compare = this.ascendingComparer;
      if (!compare) return null;

      return !this.sort.desc ? compare : (a, b) => compare(b, a);
    },
    ascendingComparer() {
      const prop = this.sort.prop;
      const users = this.users;
      switch (prop) {
        case "index":
          return (a, b) => a[prop] - b[prop];
        case "name":
        case "surname":
          return (a, b) => a[prop].localeCompare(b[prop]);
        default:
          return null;
      }
    },

    totalPages() {
      return Math.ceil(this.users.length / this.page.size);
    }
  },
  methods: {
    onFormSubmit(e) {
      e.preventDefault();
      this.addNewUser();
    },
    addNewUser() {
      const newUser = this.newUser;
      if (!newUser.name || !newUser.surname) {
        alert("Необходимо указать имя и фамилию нового пользователя");
        return;
      }
      this.users.push(newUser);
      this.form.name = this.form.surname = "";
    },

    toggleSort(prop) {
      const sort = this.sort;
      if (sort.prop !== prop) {
        // If not sorted by given prop then sort ascending
        this.sort.prop = prop;
        this.sort.desc = false;
      } else if (!sort.desc) {
        // If sorted ascending by given prop then sort descending
        this.sort.desc = true;
      } else {
        // If sorted descending by given prop then unsort
        this.sort.prop = "";
        this.sort.desc = false;
      }
      console.log({ prop: this.sort.prop, desc: this.sort.desc });
    },

    prevPage() {
      if (this.page.index > 0) this.page.index -= 1;
    },
    nextPage() {
      if (this.page.index < this.totalPages - 1) this.page.index += 1;
    }
  }
};
</script>
