<template>
  <main>
    <template v-if="!isAuth">
      <v-form-run
        v-if="!stepAuth"
        @error="showErrorNotification"
        @success="formRunSuccessHandler"
        :settings-form="{ buttonText: 'Отправить заявку' }"
        ref="VFormRun"
      />
      <v-form-auth
        v-else
        :form-object="formAuth"
        :settings-form="{
          readOnlyFields: { email: true, password: false },
          buttonText: 'Зарегистрироваться'
        }"
        @success="formAuthSuccessHandler"
        @error="showErrorNotification"
        ref="VFormAuth"
      />
    </template>
    <template v-else>
      <p>
        Заявку можно отредактировать перейдя по
        <router-link to="/profile">ссылке</router-link>
      </p>
    </template>
  </main>
</template>

<script>
import { useToast } from 'vue-toastification'
import { useVuelidate } from '@vuelidate/core'
import {
  required,
  email,
  helpers,
  minLength,
  maxLength,
  maxValue,
  minValue,
  numeric
} from '@vuelidate/validators'
import { mapActions, mapGetters, mapMutations } from 'vuex'
import VFormAuth from '@/components/VFormAuth.vue'
import VFormRun from '@/components/VFormRun.vue'

export default {
  name: 'HomeView',
  components: { VFormRun, VFormAuth },
  data() {
    return {
      v$: useVuelidate(),
      optionsDistance: [
        { value: 3, text: '3 км' },
        { value: 5, text: '5 км' },
        { value: 10, text: '10 км' }
      ],
      formAuth: {
        email: '',
        password: ''
      },
      formRun: {
        name: '',
        date: '',
        email: '',
        phone: '',
        distance: '',
        payment: '',
        isCreateProfile: false
      },
      toast: useToast(),
      stepAuth: false
    }
  },
  validations() {
    return {
      form: {
        name: {
          required: helpers.withMessage('Поле обязательно для заполнения', required),
          minLength: helpers.withMessage(
            'Отображаемое имя должно быть больше 3 символов',
            minLength(4)
          ),
          maxLength: helpers.withMessage(
            'Отображаемое имя должно быть меньше 34 символов',
            maxLength(34)
          )
        },
        date: {
          required: helpers.withMessage('Поле обязательно для заполнения', required)
        },
        email: {
          required: helpers.withMessage('Поле обязательно для заполнения', required),
          email: helpers.withMessage('Email введён некорректно', email)
        },
        phone: {
          required: helpers.withMessage('Поле обязательно для заполнения', required)
        },
        distance: {
          required: helpers.withMessage('Поле обязательно для заполнения', required)
        },
        payment: {
          required: helpers.withMessage('Поле обязательно для заполнения', required),
          numeric: helpers.withMessage('Поле должно быть числовым', numeric),
          minLength: helpers.withMessage('Поле должено быть больше 100руб.', minValue(100)),
          maxLength: helpers.withMessage(
            'Поле должено быть меньше или равно 10 000руб.',
            maxValue(10000)
          )
        }
      }
    }
  },
  computed: {
    ...mapGetters({
      isAuth: 'auth/getAuth',
      userGuid: 'auth/getUserGuid'
    })
  },
  methods: {
    ...mapMutations({
      addParticipants: 'participants/addParticipants'
    }),
    ...mapActions({
      registerUser: 'auth/registerUser'
    }),
    formRunSuccessHandler(formData) {
      this.formAuth.email = formData.email

      this.formRun = formData
      formData.guid = this.userGuid

      this.addParticipants(formData)

      this.showSuccessNotification('Вы успешно зарегистрировались в забеге!')

      if (formData.isCreateProfile) {
        this.stepAuth = true
      }

      this.$refs.VFormRun.resetForm()
    },
    async formAuthSuccessHandler(formData) {
      const res = await this.registerUser(formData)

      if (res.status === 'Error') {
        const errors = res.data?.errors
        errors?.forEach((error) => {
          this.$refs.VFormAuth.vuelidateExternalResults.form[error.param] = error.msg
        })
        this.showErrorNotification('Не все поля заполнены верно!')
      } else {
        this.showSuccessNotification('Вы успешно зарегистрировались!')
      }
    },
    showErrorNotification(message) {
      this.toast.error(message)
    },
    showSuccessNotification(message) {
      this.toast.success(message)
    }
  }
}
</script>
