<template>
    <div class="container">
        <div class="form-group">
            <input type="text" @blur="saveName" v-model="name" class="form-control" :class="{ 'is-invalid': $v.name.$error }">
            <div class="invalid-feedback" v-if="!$v.name.required">
               Это поле обязательное.
            </div>
            <div class="invalid-feedback" v-if="!$v.name.maxLength">
               Максимальная длина {{$v.name.$params.maxLength.max}} символов
            </div>
        </div>
        <div class="alert alert-danger" role="alert" v-if="errored">
            Ошибка загрузки данных!
        </div>
        <div class="spinner-border" style="width: 3rem; height: 3rem;" role="status" v-if="loading">
            <span class="visually-hidden">Loading...</span>
        </div>
    </div>
</template>

<script>
    import { required, maxLength } from 'vuelidate/lib/validators'
    export default {
        props: [
            'deskId'
        ],
        data(){
            return {
                name: null,
                errored: false,
                loading: true
            }
        },
        methods:{
            saveName(){
                this.$v.$touch()
                if(this.$v.$anyError){
                    return;
                }
                axios.post( '/api/V1/desks/'+this.deskId,{
                        _method: 'PUT',
                        name: this.name
                    })
                    .then( response => {
                        // this.name = response.data.data.name
                        // console.log(response.data.data)
                    })
                    .catch( error => {
                        console.log(error)
                        this.errored = true
                    })
                    .finally( () => {
                        this.loading = false
                    })
            }
        },
        mounted() {
            axios.get( '/api/V1/desks/'+this.deskId)
                .then( response => {
                    this.name = response.data.data.name
                    // console.log(response.data.data)
                })
                .catch( error => {
                    console.log(error)
                    this.errored = true
                })
                .finally( () => {
                    this.loading = false
                })
        },
        validations: {
            name:{
                required,
                maxLength: maxLength(10)
            }
        }
    }
</script>

