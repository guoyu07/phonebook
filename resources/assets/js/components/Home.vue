<template>
    <div>
        <nav class="panel column is-offset-2 is-8">
            <p class="panel-heading">
                VueJS Phone Book App
                <button class="button is-link is-outlined" @click="openAdd">
                    Add New
                </button>

                <span class="is-pulled-right" v-if="loading">
                    <i class="fa fa-refresh fa-spin fa-2x fa-fw"></i>
                </span>
            </p>
            <div class="panel-block">
                <p class="control has-icons-left">
                    <input class="input is-small" type="text" placeholder="search" v-model="searchQuery">
                    <span class="icon is-small is-left">
                        <i class="fas fa-search"></i>
                    </span>
                </p>
            </div>
            <a class="panel-block" v-for="(item, key) in temp">
                <span class="column is-9">
                    {{ key +1 }}. {{ item.name }}
                </span>

                <span class="panel-icon column is-1">
                  <i class="has-text-danger fa fa-trash" @click="deletePhone(key, item.id)"></i>
                </span>
                <span class="panel-icon column is-1">
                  <i class="has-text-info fa fa-edit" @click="editPhone(key)"></i>
                </span>
                <span class="panel-icon column is-1">
                  <i class="has-text-primary fa fa-eye" @click="showPhone(key)"></i>
                </span>
            </a>
        </nav>

        <add :openModal="addActive" @closeModal="closeModal"></add>
        <show :showModal="showActive" :phone="phone" @closeModal="closeModal"></show>
        <edit :editActive="editActive" :phone="phone" @closeModal="closeModal"></edit>
    </div>
</template>

<script>
    let Add = require('./Add');
    let Show = require('./Show');
    let Edit = require('./Edit');

    export default {
        data () {
            return {
                addActive: '',
                showActive: '',
                editActive: '',
                loading: false,
                searchQuery: '',
                phones: {},
                phone: {},
                temp: {},
                errors: {}
            }
        },
        components: {
            Add,
            Show,
            Edit
        },
        mounted () {
            axios.get('/phone')
                .then((response) => {
                    this.phones = this.temp = response.data
                })
                .catch((error) => this.errors = error.response.data.errors);
        },
        watch: {
            searchQuery () {
                if (!this.searchQuery.length > 0) this.temp = this.phones;

                this.temp = this.phones.filter((item) => {

                    return Object.keys(item).some((key) => {
                        let string = String(item[key]);

                        return string.toLowerCase().indexOf(this.searchQuery.toLowerCase()) > -1;
                    });

                });
            }
        },
        methods: {
            openAdd () {
                this.addActive = 'is-active';
            },
            closeModal () {
                this.addActive = this.showActive = this.editActive = '';
            },
            showPhone (key) {
                this.phone = this.temp[key];
                this.showActive = 'is-active';
            },
            editPhone (key) {
                this.phone = this.temp[key];
                this.editActive = 'is-active';
            },
            deletePhone (key, itemId) {
                if (!confirm('Are you sure ?')) return false;

                this.loading = !this.loading;

                axios.delete('/phone/' + itemId)
                    .then((response) => {
                        this.loading = !this.loading;
                        this.temp.splice(key, 1);
                    })
                    .catch((error) => this.errors = error.response.data.errors);
            }
        }
    }
</script>

<style></style>
