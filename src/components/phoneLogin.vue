<template>
  <div class="animated fadeIn centered" style="max-width:500px; margin:auto">
      <b-card no-body v-if="formConfig.visible" style="max-width:600px;">
        <div slot="header" style="text-align: center">
          <i class="icon-note"></i> {{ formConfig.header }}
          <div class="card-header-actions" v-if="formConfig.visibleHeaderAction">
            <b-link href="#" class="card-header-action btn-close" v-on:click="toggleVisible(formConfig)">
              <i class="icon-close"></i>
            </b-link>
          </div>
        </div>
        <b-card-body>
          <b-form>
            <b-form-group :label="object.label" v-if="object.type === 'phoneValid'">
              <b-form-input class="d-none" type="text" :placeholder="object.placeholder" v-model="object.value" :aria-describedby="object.label" :state="checkState(object)"></b-form-input>
               <b-dropdown class = "myDropdown" variant="outline-primary" style="position:absolute; padding-bottom:6px; ">
                <template slot="button-content" style="display: inline-block; width:150px; ">
                    <div style="display: inline-block; "><i :class="areaClass" ></i> {{areaName}}</div>
                </template>
                <template v-for="option in object.options">
                    <b-dropdown-item :key="option.country" @click="selectOption(option)" >
                    <i :class="option.flagClass"></i> {{option.country+' (+'+option.code+')'}}
                    </b-dropdown-item>
                </template>
                </b-dropdown>
              <input style="width:100%; display: inline; padding-left:150px" v-model="phone" type="text" :placeholder="object.placeholder" class="form-control my-input">
              <b-form-invalid-feedback :id="object.label">
              {{ object.feedback.replace('fixedLength', this.areaPhoneLength) }}
              </b-form-invalid-feedback>
  
            </b-form-group>
            <a  style="position:absolute; padding-bottom:6px; right:20px" href="" >忘记密码</a>
            <b-form-group :label="object2.label" v-if="object2.type === 'password'">
              <b-form-input class="d-none" type="password" :placeholder="object2.placeholder" v-model="object2.value"  :aria-describedby="object2.label" :state="checkState3()"></b-form-input>
              <input v-model="object2.value" type="password" :placeholder="object2.placeholder" class="form-control">
              <b-form-invalid-feedback :state="checkState3()">
                  {{ object2.feedback }}
              </b-form-invalid-feedback>
            </b-form-group>
          </b-form>
          <div style="text-align: center">
            <b-button  style="width:100px;" 
              @click="requsetLogin" :disabled="!checkState()||!checkState3()" 
              variant="primary">登陆</b-button>
              <a style="position:absolute; left: calc(50% + 60px); bottom:8%"  href="" >创建账号</a>
          </div>
        </b-card-body>
      </b-card>
    </div>
</template>

<script>
import { validationMixin } from "vuelidate"
import { required, minLength, maxLength } from "vuelidate/lib/validators"

const config = {
  visibleHeaderAction: false,
  visible: true,
  header: '手机登陆',
  groups: [
    {
      type: 'phoneValid',
      label: '电话',
      label2: '验证码',
      placeholder: '请输入电话',
      placeholder2: '请输入验证码',
      feedback: '请选择地区, 电话为fixedLength位数字',
      feedback2: '请输入正确的验证码',
      key: 'phone',
      value: null,
      value2: null,
      fixedLength: {'86': 11, '1': 10},
      holdOnSeconds:5,
      options:  [
        {  country: '中国', code:'86', flagClass: 'flag-icon flag-icon-cn', phoneLength:11 },
        {  country: '加拿大', code:'1', flagClass: 'flag-icon flag-icon-ca',  phoneLength:10 },
      ]
    },
    {
      type: 'password',
      label: '密码',
      label2: '密码确认',
      placeholder: '请输入密码',
      feedback: '密码长度必须是6-12位',
      feedback2: '两次密码必须一致',
      key: 'password',
      value: null,
      value2: null,
      minLength: 6,
      maxLength: 12
    },
  ]
}

export default {
  name: 'PhoneLogin',
  props: {
  },
  data() {
    return {
      formConfig: config,
      object: config.groups[0],
      object2: config.groups[1],
      areaClass:'',
      areaName:'请选择区码',
      areaCode:'',
      areaPhoneLength:'',
      phone: '',
      buttonName: '发送验证码',
      validCode: '',
      holdOnValidate: false,
    }
  },
  watch: {
    areaCode(val) {
      this.object.value = val + '-' + this.phone
    },
    phone(val) {
      this.object.value = this.areaCode + '-' + val
    },
  },
  computed: {
  },
  mounted: function() {
  },
  mixins: [validationMixin],
  validations() {
    const validPhone = (value) => {
      let areaCode = ''
      let phone = ''
      if(value){
        areaCode = value.split('-')[0]
        phone = value.split('-')[1]
      }
      return areaCode.length >0 && areaCode.length <=3 && !isNaN(areaCode) && phone.length == this.object.fixedLength[this.areaCode] && !isNaN(phone)
    }
    return {
      object: {
        value: {
          validPhone,
        },
      },
      object2: {
        value: {
          required,
          minLength: minLength(this.object2.minLength),
          maxLength: maxLength(this.object2.maxLength)
        },
      }
    }
  },
  methods: {
    checkState() {
      return !this.$v.object.value.$invalid
    },

    checkState3() {
      return !this.$v.object2.value.$invalid
    },

    async sendValidCode() {
      //this.validCode = sendValidCodeApi(this.object.value)
      this.validCode = Math.round(Math.random()*(9999-1000)+1000)
      console.log("phoneNumber is ", this.object.value)
      console.log("validCode is ", this.validCode)
      this.holdOnValidate = true
      for(var i=this.object.holdOnSeconds; i>0; i--){
        this.buttonName = "等待"+i+"秒"
        await this.sleep(1000);
      }
      this.holdOnValidate = false
      this.buttonName = "再次发送"
    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    selectOption(option) {
      this.areaCode = option.code
      this.areaName = option.country + ' (+'+option.code+')'
      this.areaClass = option.flagClass
      this.areaPhoneLength = option.phoneLength
    },
    requsetLogin() {
      //loginApi(object1.value, object2.value)
    }
  }
}
</script>

<style scoped>
.myDropdown {

}
.centered {

  margin-left: auto;
  margin-right: auto;
   position: relative;
     top: calc(50vh - 210px);
  /*position: fixed;*/
  /*top: 50%;
  -ms-transform: translateY(-50%);
  transform: translateY(-50%);*/
}
html, body {margin: 0; height: 100%; overflow: hidden}
/**/
</style>