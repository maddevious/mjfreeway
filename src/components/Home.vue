<template>
  <div>
    <div class="container jumbotron">
      <p>Please select your favorite drink that you consumed today.</p>
    </div>
    <div class="container">
      <b-container fluid="md">
        <div class="alert" :class="updateMessageType" id="alert" v-if="message">{{ message }}</div>
        <b-form @submit="onSubmit">
          <b-form-group id="input-group-3" label="Drink:" label-for="input-3">
            <b-form-select
              id="input-3"
              v-model="form.drink"
            >
              <option value="">Select</option>
              <option v-for="drink in computedDrinks" :key="drink.uuid" :value="drink.uuid">{{ drink.name }}</option>
            </b-form-select>
          </b-form-group>

          <b-form-group id="input-group-2" label="Quantity" label-for="input-2">
            <b-form-input
              id="input-2"
              @keypress="onlyNumber"
              v-model="form.quantity"
              required
              placeholder="Enter quantity"
            ></b-form-input>

          </b-form-group>
          <b-button type="submit" variant="primary">Submit</b-button>
        </b-form>
        <b-card class="mt-3" header="Result" v-if="caffeine_available">
      <pre class="m-0">
        Available caffeine: {{ caffeine_available }}<br/>
        {{ caffeine_message }}
      </pre>
        </b-card>
      </b-container>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        quantity: '',
        drink: ''
      },
      drinks: [],
      caffeine_available: '',
      caffeine_message: '',
      message: null,
      messageType: 'error'
    }
  },
  methods: {
    onSubmit (evt) {
      evt.preventDefault()
      this.axios.post('http://localhost:8085/api/usage', {
        user: '429922675d595568740b776521f07c30',
        drink: this.form.drink,
        quantity: this.form.quantity
      })
        .then((response) => {
          this.message = null
          if (response.data.message.code !== 100) {
            if (Array.isArray(response.data.message.messages)) {
              this.message = response.data.message.messages.join('<br />')
            } else {
              this.message = response.data.message.messages
            }
            this.messageType = 'warning'
          }
          this.caffeine_available = response.data.data.caffeine_available
          this.caffeine_message = response.data.data.caffeine_message
        })
        .catch(function (error) {
          alert(error)
        })
    },
    onlyNumber ($event) {
      let keyCode = ($event.keyCode ? $event.keyCode : $event.which)
      if ((keyCode < 48 || keyCode > 57) && keyCode !== 46) {
        $event.preventDefault()
      }
    }
  },
  mounted () {
    this.axios.get('http://localhost:8085/api/drink')
      .then((response) => {
        this.drinks = response.data.data.drink
      })
      .catch(function (error) {
        alert(error)
      })
  },
  computed: {
    updateMessageType () {
      if (this.messageType === 'warning') {
        return 'alert-warning'
      } else if (this.messageType === 'success') {
        return 'alert-success'
      } else {
        return 'alert-danger'
      }
    },
    computedDrinks () {
      return this.drinks
    }
  }
}
</script>
