<template>
  <div>
    <h1>Lista de Usuarios</h1>
    <div class="mb-3 text-left">
      <b-button v-b-modal.modal-1 variant="outline-primary">Crear nuevo usuario</b-button>
      <b-modal id="modal-1" ref="modal-1" :title="title" hide-footer>
        <b-form @submit="onSubmit" v-if="show">
          <b-form-group
            id="input-group-1"
            label="Nombre:"
            label-for="input-name"
          >
            <b-form-input
              id="input-name"
              v-model="form.name"
              placeholder="Ingrese nombre"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group
            id="input-group-2"
            label="Correo electrónico:"
            label-for="input-mail"
          >
            <b-form-input
              id="input-mail"
              v-model="form.mail"
              type="email"
              placeholder="Ingrese correo electrónico"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group id="input-group-3" label="Edad:" label-for="input-age">
            <b-form-input
              id="input-age"
              v-model="form.age"
              type="number"
              placeholder="Ingrese edad"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group id="input-group-4" label="Password:" label-for="input-password">
            <b-form-input
              id="input-password"
              v-model="form.password"
              type="password"
              :required="!modoEdit"
            ></b-form-input>
          </b-form-group>  

          <b-form-group id="input-group-5" label="Username:" label-for="input-username">
            <b-form-input
              id="input-username"
              v-model="form.username"
              required
            ></b-form-input>
          </b-form-group>                 

          <div class="text-center">
            <b-button type="submit" variant="primary" class="mr-2">Aceptar</b-button>
            <b-button  @click="$bvModal.hide('modal-1'), limpiar()">Cancelar</b-button>
          </div>
        </b-form>
      </b-modal>
    </div>
    <div>
      <b-table striped hover :fields="fields" :items="items">
        <template #cell(actions) = 'row'>
          <b-button @click="editUser(row.item)" size="sm" variant="warning" class="mr-1">
            Editar
          </b-button>
          <b-button @click="DeleteUser(row.item)" size="sm" variant="danger">
            Eliminar
          </b-button>
        </template>
      </b-table>
    </div>    
  </div>
</template>
<script>
import axios from 'axios'
  export default {
    data() {
      return {
        title: 'Creando nuevo usuario',
        items: [],
        fields: [
          {
            key: 'name',
            label: 'Nombre'
          },
          {
            key: 'mail',
            label: 'E-mail',
          },
          {
            key: 'age',
            label: 'Edad',
          },
          {
            key: 'username',
            label: 'Nombre de usuario',
          },
          {
            key: 'actions',
            label: 'Acciones',
          }
        ],
        form: {
          id: '',
          mail: '',
          name: '',
          age: null,
          password: '',
          username: '',
        },
        show: true,
        modoEdit: false
      }
    },
    created() {
      axios.defaults.headers.common["Authorization"] = localStorage.getItem(
      "jwtToken"
      )
      axios.get('https://fast-dusk-52904.herokuapp.com/api/user')
      .then(res => {
        this.items = res.data.users
      })
      .catch(err => {
        console.error(err.response.status);
        if (err.response.status === 401) {
          console.log('prueba push')
          this.$router.push('/login')
        }
      })
    },
    methods: {
      limpiar() {
        this.form = {
          mail: '',
          name: '',
          age: null,
          password: '',
          username: ''
        }
        this.title = 'Creando nuevo usuario'
        this.modoEdit = false
      },
      onSubmit(event) {
        event.preventDefault()
        if (this.modoEdit) {
          axios.put(`https://fast-dusk-52904.herokuapp.com/api/user/${this.form.id}`, this.form)
          .then(res => {
            //console.log(res)
            this.$swal({
              icon: 'success',
              title: res.data.message,
              showConfirmButton: false,
              timer: 2000
            })
            this.$refs['modal-1'].hide()
            this.limpiar()
            axios.get('https://fast-dusk-52904.herokuapp.com/api/user')
            .then(res => {
              this.items = res.data.users
            })
            .catch(err => {
              console.error(err);
            })              
          })
          .catch(err => {
            console.error(err); 
          })      
        } else {
          axios.post('https://fast-dusk-52904.herokuapp.com/api/user', this.form)
          .then(res => {
            //console.log(res)
            this.items.push(res.data.user)
            this.$swal({
              icon: 'success',
              title: res.data.message,
              showConfirmButton: false,
              timer: 2000
            })
            this.$refs['modal-1'].hide()
            this.limpiar()
          })
          .catch(err => {
            console.error(err); 
          })      
        }  
      },      
      editUser(item) {
        this.modoEdit = true
        this.title = `Editando a ${item.name}`
        axios.get(`https://fast-dusk-52904.herokuapp.com/api/user/${item._id}`)
        .then(res => {
          this.form.id = res.data.user._id
          this.form.name = res.data.user.name
          this.form.mail = res.data.user.mail
          this.form.age = res.data.user.age
          this.form.username = res.data.user.username
          this.$refs['modal-1'].show()
        })
        .catch(err => {
          console.error(err); 
        })
      },
      DeleteUser(item) {
        this.$swal({
          title: '¿Esta seguro?',
          text: "No podra deshacer esto!",
          icon: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Si, Eliminar!'
        }).then((result) => {
          if (result.isConfirmed) {
            axios.delete(`https://fast-dusk-52904.herokuapp.com/api/user/${item._id}`)
            .then(res => {
              axios.get('https://fast-dusk-52904.herokuapp.com/api/user')
              .then(res => {
                this.items = res.data.users
              })
              .catch(err => {
                console.error(err);
              })   
              this.$swal({
                icon: 'success',
                title: res.data.message,
                showConfirmButton: false,
                timer: 2000
              })
            })
            .catch(err => {
              console.error(err); 
            })            
          }
        })
      }
    }
  }
</script>