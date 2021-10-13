<template>
  <div class="login">
    
    <div class="mt-5 login-content">
      <b-form @submit="onSubmit" @reset="onReset" v-if="show">
        <b-form-group
          id="input-group-1"
          label="Nombre de usuario:"
          label-for="input-1"
          description="We'll never share your email with anyone else."
        >
          <b-form-input
            id="input-1"
            v-model="form.username"
            placeholder="Ingrese usuario"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-2" label="Password:" label-for="input-2">
          <b-form-input
            id="input-2"
            v-model="form.password"
            type="password"
            required
          ></b-form-input>
        </b-form-group>  

        <b-button type="submit" variant="primary" class="mr-2">Submit</b-button>
        <b-button type="reset" variant="danger">Regresar</b-button>
      </b-form>      
    </div>
  </div>
</template>

<script>
import axios from 'axios'
  export default {
    data() {
      return {
        form: {
          username: '',
          password: '',
        },
        show: true,
        errors: []
      }
    },
    methods: {
      onSubmit(event) {
        event.preventDefault()
        console.log('hola')
        axios
          .post(`http://localhost:3000/api/auth/login/`, this.form)
          .then(response => {
            localStorage.setItem("jwtToken", response.data.token)
            this.$router.push('/admin')
          })
          .catch(err => {
            this.$swal({
              icon: 'error',
              title: 'Oops...',
              text: err.response.data.msg,
            })
            console.log("error", err.response.data.msg);
            this.errors = err;
          });        
      },
      onReset(event) {
        event.preventDefault()
        this.$router.push('/')
      }
    }
  }
</script>
<style>
  .login {
    display: flex;
    justify-content: center;
    align-content: center;
  }
  .login .login-content{
    background-color: aqua;
    padding: 15px;
    max-width: 400px;
    border-radius: 10px;
  }
</style>