<template>
  <div v-show="show" class="alert alert-success alert-dismissible" role="alert">
    <strong>Felicitaciones!</strong> Usuario guardado correctamente.
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
  </div>
  <div class="container py-5">
    <div class="card card-body">
    <div class="row">
      <div class="col-lg-4 mb-5">
        <h2>Nuevo Usuario</h2>
        <div class="card card-body" v-on:submit="addUser">
          <form>
            <div class="mb-3">
              <input type="email" ref="email" v-model="user.email" class="form-control" placeholder="Email" required>
            </div>
            <div class="mb-3">
              <input type="text" v-model="user.first_name" class="form-control" placeholder="Nombre(s)" required>
            </div>
            <div class="mb-3">
              <input type="text" v-model="user.last_name" class="form-control" placeholder="Apellido(s)" required>
            </div>
            <div class="mb-3">
              <input type="date" v-model="user.birthday" class="form-control" placeholder="Fecha de nacimiento" required>
            </div>
            <div class="d-grid mb-3">
              <input type="submit" class="btn btn-success btn-block" value="Agregar Usuario">
            </div>
          </form>
        </div>
      </div>
      <div class="col-lg-8">
        <h3 class="mb-3 fw-bold">
          Lista de Usuarios
          <span class="badge bg-primary rounded-pill ms-3 fs-6 fw-normal">
              {{ totalUsers }} Usuarios
            </span>
        </h3>
        <div class="table-responsive">
          <table class="table table-striped">
            <thead>
            <tr>
              <th>Id</th>
              <th>Nombre</th>
              <th>Edad</th>
              <th>Email</th>
              <th>Eliminar</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(user, index) in users">
              <td>{{ user.id }}</td>
              <td>{{ user.first_name }} {{ user.last_name }}</td>
              <td>{{ user.birthday !== '' ? this.userAge(user.birthday) : '-' }}</td>
              <td>{{ user.email }}</td>
              <td><button @click="deleteUser(user.id, $event)"
                          class="btn btn-danger btn-block">
                <font-awesome-icon icon="xmark" />
              </button></td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    </div>
  </div>
</template>

<style>
@import './assets/base.css';
</style>

<script>
export default {
  data() {
    return {
      user: {
        id: '',
        email: '',
        first_name: '',
        last_name: '',
        birthday: ''
      },
      users: [],
      show: false,
    }
  },
  created() {
    if (localStorage.getItem('dzc.users') != null) {
      this.users = JSON.parse(localStorage.getItem('dzc.users'));
    } else {
      this.listUsers();
    }
  },
  mounted() {
    this.$refs.email.focus();
  },
  methods: {
    addUser: function(event) {
      event.preventDefault();
      this.user.id = this.findMaxId() + 1;
      this.users.push({
        id: this.user.id,
        email: this.user.email,
        first_name: this.user.first_name,
        last_name: this.user.last_name,
        birthday: this.user.birthday
      });
      this.show = true;
      this.updateLocalStorage();
      this.clearInputs();
    },
    listUsers: async function () {
      const res = await fetch('https://reqres.in/api/users?page=1');
      const data = await res.json();

      data.data.map(function (item) {
        return item.birthday = '';
      });

      this.users = data.data;
      this.updateLocalStorage();
    },
    updateLocalStorage: function () {
      localStorage.setItem('dzc.users', JSON.stringify(this.users));
    },
    deleteUser: function (id, event) {
      const confirmation = confirm('¿Realmente desea eliminar este usuario?');
      if (confirmation) {
        this.users = this.users.filter(user => user.id != id);
        this.updateLocalStorage();
      } else {
        event.preventDefault();
      }
    },
    findMaxId: function () {
      const maxId = Math.max.apply(Math, this.users.map(function (user) {
        return user.id;
      }));

      return maxId;
    },
    clearInputs: function () {
      this.user.id = '';
      this.user.email = '';
      this.user.first_name = '';
      this.user.last_name = '';
      this.user.birthday = '';
      this.$refs.email.focus();
    },
    userAge: function(birthday) {
      const today = new Date();
      const currentYear   = parseInt(today.getFullYear());
      const currentMonth  = parseInt(today.getMonth()) + 1;
      const currentDay    = parseInt(today.getDate());

      const birthdayYear  = parseInt(String(birthday).substring(0, 4));
      const birthdayMonth = parseInt(String(birthday).substring(5, 7));
      const birthdayDay   = parseInt(String(birthday).substring(8, 10));

      let age = currentYear - birthdayYear;
      if (currentMonth < birthdayMonth) {
        age--;
      } else if (currentMonth === birthdayMonth) {
        if (currentDay < birthdayDay) {
          age--;
        }
      }

      return age;
    }
  },
  computed: {
    totalUsers() {
      return this.users.length;
    }
  }
}
</script>