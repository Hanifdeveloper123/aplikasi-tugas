<script>
export default {
    layout: AppLayout
}
</script>

<script setup>
import axios from "axios"
import { notify } from "notiwind"
import { ref, onMounted } from "vue"
import { object, string } from "vue-types"
import { Inertia } from '@inertiajs/inertia'
import { Head, Link } from "@inertiajs/inertia-vue3"
import AppLayout from '@/layouts/apps.vue'
import VInput from "@/components/VInput/index.vue"
import VButton from "@/components/VButton/index.vue"
import VBreadcrumb from '@/components/VBreadcrumb/index.vue'
import VBack from '@/components/src/icons/VBack.vue'
import VModalForm from './CreateModalForm.vue'

const breadcrumb = [
    {
        name: "Dashboard",
        active: false,
        to: route('dashboard.index')
    },
    {
        name: "Tugas Management",
        active: true,
        to: route('tugas.tugas.createpage')
    },
]
const isLoading = ref(false)
const backActive = ref(false)
const form = ref({
    judul: '',
    deskripsi: '',
    user_id: '',
    status: '',
    deadline: '',
    permissions: []
})
const formError = ref({})
const itemSelected = ref({})
const openModalForm = ref(false)

const props = defineProps({
    title: string(),
    additional: object()
})

const handleActiveAll = (sub_group, index) => {
    const selectId = sub_group[0]['group'] + '_' + index
    if (document.getElementById(selectId).checked) {
        Object.values(sub_group).forEach(val => {
            let permission = form.value.permissions.find(e => e.id === val.id)
            permission.status = true
        })
    } else {
        Object.values(sub_group).forEach(val => {
            let permission = form.value.permissions.find(e => e.id === val.id)
            permission.status = false
        })
    }
}

const initData = () => {
    Object.values(props.additional.permission_list).forEach(val => {
        Object.values(val).forEach(value => {
            Object.values(value).forEach(res => {
                const obj = {
                    id: res.id,
                    status: false
                }
                form.value.permissions.push(obj)
            })
        })
    })
}

const handleManageModal = (data) => {
    itemSelected.value = data
    openModalForm.value = true
}

const closeModalForm = () => {
    openModalForm.value = false
}

const successSubmit = (data) => {
    form.value = data
    openModalForm.value = false
}

const activeChecker = (subGroup) => {
    return form.value.permissions.filter(function (item) {
        return subGroup.map(function (item) { return item['id'] }).includes(item.id);
    }).map(function (item) { return item['status'] }).includes(false) ? false : true
}

const discard = () => {
    form.value.permissions = []
    initData()
}
const user_id = ref(null);
    fetch('http://127.0.0.1:8000/getUser')
    .then(response => response.json())
    .then(data=> user_id.value = data);

const submit = async () => {
    isLoading.value = true
    axios.post(route('tugas.tugas.storetugas'), form.value)
        .then((res) => {
            discard()
            notify({
                type: "success",
                group: "top",
                text: res.data.meta.message
            }, 2500)
            Inertia.visit(route('tugas.tugas.index'))
        }).catch((res) => {
            // Handle validation errors
            const result = res.response.data
            const metaError = res.response.data.meta?.error
            if (result.hasOwnProperty('errors')) {
                formError.value = ref({});
                Object.keys(result.errors).map((key) => {
                    formError.value[key] = result.errors[key].toString();
                });
            }

            if (metaError) {
                notify({
                    type: "error",
                    group: "top",
                    text: metaError
                }, 2500)
            } else {
                notify({
                    type: "error",
                    group: "top",
                    text: result.message
                }, 2500)
            }
        }).finally(() => isLoading.value = false)
}

onMounted(() => {
    initData()
});

;
</script>


<template>
    <Head :title="title"></Head>
    <VBreadcrumb :routes="breadcrumb" />
    <div class="mb-4 sm:mb-6 flex justify-start space-x-2 items-center">
        <Link :href="route('tugas.tugas.index')" class="cursor-pointer">
            <VBack width="32" height="32" :is-active="backActive" @mouseover="backActive = true" @mouseout="backActive = false"/>
        </Link>
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">Tambah Tugas</h1>
    </div>
    <div class="bg-white shadow-lg rounded-sm border border-slate-200">
        <!-- Panel Content -->
        <div class="p-6 space-y-6">
            <!-- Basic Information -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">Form Tambah Tugas</h3>
                    <div class="text-sm text-slate-500 mb-4">Silahkan Isi Form Berikut</div>
                    <VInput placeholder="Masukkan Judul Tugas" label="Judul Tugas" :required="true" v-model="form.judul"
                        :errorMessage="formError.judul" @update:modelValue="formError.judul = ''" class="lg:w-1/2 md:w-auto"/>

                    <VInput placeholder="Masukkan Deskripsi Tugas" label="Deskripsi Tugas" :required="true" v-model="form.deskripsi"
                        :errorMessage="formError.deskripsi" @update:modelValue="formError.deskripsi = ''" class="lg:w-1/2 sm:w-auto"/>


                        <label for="formBookImage"
                        class="block text-gray-700 text-sm  mb-2">User ID</label>
                         <select
                         v-model="form.user_id"
                        class="shadow appearance-none border rounded py-2 lg:w-1/2 sm:w-auto px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                        id="formBookImage" placeholder="Masukkan User">
                        <option v-for="user in user_id" :value="user.id">{{user.name}}</option>
                     </select>

                        <label class="flex">Status</label>
                        <select  
                        label="Form Status" :required="true"
                         v-model="form.status"
                         placeholder="Insert Tugas"
                        class="lg:w-1/2 sm:w-auto"
                        :errorMessage="formError.status" @update:modelValue="formError.status = ''" 
                        id="formBookImage">
                            <option value="draft" >Draft</option>
                            <option value="on_progress">On Progress</option>
                            <option value="done">Done</option>
                   </select>


                    <VInput placeholder="Insert Deadline" label="Deadline Tugas" :required="true" v-model="form.deadline" type="date"
                        :errorMessage="formError.deadline" @update:modelValue="formError.deadline = ''" class="lg:w-1/2 sm:w-auto"/>
                    
                </div>
            </section>

           

            <!-- List Permission -->
            <section>
                <div class="w-full">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">List Permission</h3>
                    <div class="text-sm text-slate-500">List to set the permissions used for the role</div>
                </div>
                <div class="w-full my-6" v-for="(data, index) in additional.permission_list" :key="index">
                    <p class="uppercase font-bold text-slate-500 text-xl">{{ index.replace(/_/g, " ") }}</p>
                    <div class="w-full flex justify-between my-4 border-b pb-4" v-for="(subGroup, index) in data" :key="index" 
                        :class="{
                            'border-b-0 pb-0' :  Object.keys(data).pop() === index
                        }">
                        <div class="max-w-[60%]">
                            <p class="capitalize text-base text-slate-800 font-semibold">{{ index.replace(/_/g, " ") }}</p>
                            <div class="font-normal text-slate-400 text-sm">
                                (<span v-for="(permission, index) in subGroup" :key="index">
                                    {{ permission.label }}{{index + 1 === subGroup.length ? '' : ', '}}
                                </span>)
                            </div>
                        </div>
                        <div class="flex">
                            <div class="flex  my-auto">
                                <div class="text-sm font-normal text-slate-400 italic mr-2">{{ subGroup.length === 1 ? 'Active' : 'Active All'}}</div>
                                <div class="form-switch">
                                    <input type="checkbox" :id="subGroup[0]['group'] + '_' + index" class="sr-only" @change="handleActiveAll(subGroup, index)"
                                        :checked="activeChecker(subGroup)" />
                                    <label class="bg-slate-400" :for="subGroup[0]['group'] + '_' + index">
                                        <span class="bg-white shadow-sm" aria-hidden="true"></span>
                                        <span class="sr-only">Enable smart sync</span>
                                    </label>
                                </div>
                            </div>
                            <div class="my-auto ml-2" v-if="subGroup.length > 1">
                                <VButton label="Manage" type="outline-primary" @click="handleManageModal(subGroup)" />
                            </div>
                        </div>
                    </div>
                </div>
            </section> 

            <!-- Panel footer -->
            <footer>
                <div class="flex flex-col px-6 py-3 border-t border-slate-200">
                    <div class="flex self-end space-x-3">
                        <VButton :is-loading="isLoading" label="Discard" type="default" @click="discard" />
                        <VButton :is-loading="isLoading" label="Kirim" type="primary" @click="submit" />
                    </div>
                </div>
            </footer>
        </div>
    </div>
    <VModalForm :data="itemSelected" :open-dialog="openModalForm" @close="closeModalForm" @successSubmit="successSubmit" :additional="form"/>
</template>

<!-- <script>
    import { ref } from 'vue';
    
</script> -->