<template>
    <div class="container">
        <h1>Доски</h1>
        <form @submit.prevent="addNewDesk">
            <div class="form-group">
                <input type="text" v-model="name"
                       class="form-control"
                       :class="{ 'is-invalid': $v.name.$error }"
                       placeholder="Введите название">
                <div class="invalid-feedback" v-if="!$v.name.required">
                    Это поле обязательное.
                </div>
                <div class="invalid-feedback" v-if="!$v.name.maxLength">
                    Максимальная длина {{$v.name.$params.maxLength.max}} символов
                </div>
            </div>
            <br>
            <button type="submit" class="btn btn-primary p-2">Добавить</button>
            <div class="alert alert-danger mt-3" role="alert" v-if="errored">
                Ошибка загрузки данных!<br>
                {{errors[0]}}
            </div>
            <div class="spinner-border" style="width: 3rem; height: 3rem;" role="status" v-if="loading">
                <span class="visually-hidden">Loading...</span>
            </div>
        </form>
        <div class="row">
            <div class="col-lg-4" v-for="desk in desks">
                <div class="card mt-3" style="width: 18rem;">
                    <router-link class="card-body" :to="{name: 'showDesk', params: {deskId: desk.id}}">
                        <h4 class="card-title">{{ desk.name }}</h4>
                    </router-link>
                </div>
                <button type="button" class="btn btn-danger mt-3" @click="deleteDesk(desk.id)">Удалить</button>
            </div>
        </div>
    </div>
</template>

<script>
    import { required, maxLength } from 'vuelidate/lib/validators'
    export default {
        data(){
            return {
                desks: [],
                errored: false,
                loading: true,
                errors: [],
                name: null,
            }
        },
        mounted() {
           this.getAllDesks()
        },
        methods: {
            getAllDesks(){
                axios.get( '/api/V1/desks')
                    .then( response => {
                        this.name = ''
                        this.desks = response.data.data
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
            deleteDesk(id){
                if(confirm('Удалить?'))
                    axios.post( '/api/V1/desks/'+id, {
                        _method: 'DELETE'
                    })
                        .then( response => {
                            this.desks = []
                            this.getAllDesks()
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
            addNewDesk(){
                this.$v.$touch()
                if(this.$v.$anyError){
                    return;
                }
                axios.post( '/api/V1/desks/',{
                    name: this.name
                })
                    .then( response => {
                        this.desks = []
                        this.getAllDesks()
                    })
                    .catch( error => {
                        console.log(error)
                        if(error.response.data.errors.name){
                            this.errors = []
                            this.errors.push(error.response.data.errors.name[0])
                        }
                        this.errored = true
                    })
                    .finally( () => {
                        this.loading = false
                    })
            }
        },
        validations: {
            name:{
                required,
                maxLength: maxLength(10)
            }
        }
    }
</script>
