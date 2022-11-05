<template>

    <div class="container">
        <div class="row mt-5">
            <div class="col-md-6">
                <div class="form-group">
                    <label for="">Nome: </label>
                    <input type="text" class="form-control" v-model="employee.name">
                </div>
            </div>
            <div class="col-md-2">
                <div class="form-group">
                    <label for="">Idade: </label>
                    <input type="text" class="form-control" v-model="employee.age">
                </div>
            </div>
            <div class="col-md-3">
                <div class="form-group">
                    <label for="">Profissão: </label>
                    <input type="text" class="form-control" v-model="employee.role">
                </div>
            </div>
            <div class="col-md-1 d-flex align-items-end justify-content-end">
                <button @click="save" class="btn btn-outline-dark"><img src="./assets/images/save.png" alt="salvar"></button>
            </div>
        </div>

        <table class="table table-hover mt-4">
            <thead class="thead-dark">
                <tr>
                    <th scope="col">Nome</th>
                    <th scope="col">Idade</th>
                    <th scope="col">Profissão</th>
                    <th scope="col" style="width:100px;"></th>
                    <th scope="col" style="width:100px;"></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="employee in employees" :key="employee.id">
                    <td>{{ employee.name }}</td>
                    <td>{{ employee.age }}</td>
                    <td>{{ employee.role }}</td>
                    <td class="text-end">
                        <button @click="startUpdate(employee)" class="btn btn-outline-dark">
                            <img src="./assets/images/edit.png" alt="editar">
                        </button>
                    </td>
                    <td class="text-end">
                        <button @click="del(employee.id)" class="btn btn-outline-dark">
                            <img src="./assets/images/delete.png" alt="deletar">
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

</template>

<script>
    import Vue from 'vue';
    import axios from "axios";

    const url = "http://127.0.0.1:3001/api/employees";

    export default {
        data: function () {
            return {
                employeeEdit: null,
                employees: [],
                employee: {}
            }
        },
        methods: {
            save() {
                if(this.hasValidInput()){
                    if (this.employeeEdit) {
                        this.update(this.employeeEdit);
                    } else {
                        this.create();
                    }
                } else {
                    Vue.$toast.warning('Preencha todos os campos');
                }
            },
            create() {
                let vm = this;

                axios.post(url,
                    {
                        age: this.employee.age,
                        name: this.employee.name,
                        role: this.employee.role
                    }
                )
                .then(function (response) {
                    vm.employees.splice(0,0,response.data.data);
                    vm.employee = {};
                    Vue.$toast.success('Criado com sucesso.');
                })
                .catch (function () {
                    Vue.$toast.error('Algo aconteceu.');
                });
            },
            startUpdate(employee) {
                this.employeeEdit = employee.id;

                Vue.set(this.employee, "age", employee.age);
                Vue.set(this.employee, "name", employee.name);
                Vue.set(this.employee, "role", employee.role);
            },
            update(id) {
                let vm = this;
                this.employeeEdit = null;

                axios.put(`${url}/${id}`,
                    {
                        age: this.employee.age,
                        name: this.employee.name,
                        role: this.employee.role
                    }
                )
                .then(function (response) {
                    vm.removeFromGrid(id, vm.employees);
                    vm.employees.splice(0,0,response.data.data);
                    vm.employee = {};
                    Vue.$toast.success('Atualizado com sucesso.');
                })
                .catch (function () {
                    Vue.$toast.error('Algo aconteceu.');
                });
            },
            del(id) {
                let vm = this;
                this.employeeEdit = null;
                this.employee = {};

                axios.delete(`${url}/${id}`)
                    .then(function (response) {
                        vm.removeFromGrid(id, vm.employees);
                        Vue.$toast.success('Removido com sucesso.');
                    })
                    .catch (function () {
                        Vue.$toast.error('Algo aconteceu.');
                    });
            },
            removeFromGrid(id, employees) {
                let selectedEmployee = employees.find(p => p.id == id);
                const i = employees.indexOf(selectedEmployee);
                employees.splice(i,1);
            },
            hasValidInput() {
                return this.employee.age && this.employee.name && this.employee.role;
            },
            loadEmployees() {
                axios.get(url)
                    .then(res => {
                        const result = res.data;
                        this.employees = result.data;
                    });
            }
        },
        mounted() {
            this.loadEmployees();
        }
    }
</script>

<style>
    html, body {
        height: 100%;
    }

    body {
        margin: 0;
        background-color: #fafafa;
        font-family: Roboto, "Helvetica Neue", sans-serif;
    }

    .btn {
        min-width: 100px;
    }
</style>
