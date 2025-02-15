<script>
export default {
    layout: AppLayout
}
</script>
<script setup>
import axios from "axios";
import { notify } from "notiwind";
import { ref, onMounted, reactive } from "vue";
import debounce from "@/composables/debounce"
import { object, string } from "vue-types";
import { Head } from "@inertiajs/inertia-vue3";
import AppLayout from '@/layouts/apps.vue';
import VBreadcrumb from '@/components/VBreadcrumb/index.vue';
import VSidebarSetting from '@/components/VSidebarSetting/index.vue';
import VFilter from './Filter.vue';
import VButton from '@/components/VButton/index.vue';
import VAlert from '@/components/VAlert/index.vue';
import VDataTable from '@/components/VDataTable/index.vue';
import VPagination from '@/components/VPagination/index.vue'
import VBadge from '@/components/VBadge/index.vue'
import VDropdownEditMenu from '@/components/VDropdownEditMenu/index.vue';
import VEmpty from '@/components/src/icons/VEmpty.vue';
import VTrash from '@/components/src/icons/VTrash.vue';
import VEdit from '@/components/src/icons/VEdit.vue';
import VLoading from '@/components/VLoading/index.vue';
import VModalForm from './ModalForm.vue';

const query = ref([])
const searchFilter = ref("");
const breadcrumb = [
    {
        name: "Dashboard",
        active: false,
        to: route('dashboard.index')
    },
    {
        name: "Company Settings",
        active: false,
        to: route('settings.company.npwp.index')
    },
    {
        name: "NPWP",
        active: true,
        to: route('settings.company.npwp.index')
    },
]
const pagination = ref({
    count: '',
    current_page: 1,
    per_page: '',
    total: 0,
    total_pages: 1
})
const alertData = reactive({
    headerLabel: '',
    contentLabel: '',
    closeLabel: '',
    submitLabel: '',
})
const updateAction = ref(false)
const itemSelected = ref({})
const openAlert = ref(false)
const openModalForm = ref(false)
const heads = ["NPWP Name", "NPWP Number", "City", "KPP", "Active Month", "Status", ""]
const isLoading = ref(true)

const props = defineProps({
    additional: object(),
    title: string()
})

const getData = debounce(async (page) => {
    axios.get(route('settings.company.npwp.getdata'), {
        params: {
            page: page,
            search: searchFilter.value
        }
    }).then((res) => {
        query.value = res.data.data
        pagination.value = res.data.meta.pagination
    }).catch((res) => {
        notify({
            type: "error",
            group: "top",
            text: res.response.data.message
        }, 2500)
    }).finally(() => isLoading.value = false)
}, 500);

const nextPaginate = () => {
    pagination.value.current_page += 1
    isLoading.value = true
    getData(pagination.value.current_page)
}

const previousPaginate = () => {
    pagination.value.current_page -= 1
    isLoading.value = true
    getData(pagination.value.current_page)
}

const searchNpwp = (search) => {
    searchFilter.value = search
    isLoading.value = true
    getData(1)
};

const handleAddModalForm = () => {
    updateAction.value = false
    openModalForm.value = true
}

const handleEditModalForm = (data) => {
    updateAction.value = true
    itemSelected.value = data
    openModalForm.value = true
}

const successSubmit = () => {
    isLoading.value = true
    getData(pagination.value.current_page)
}

const closeModalForm = () => {
    itemSelected.value = ref({})
    openModalForm.value = false
}

const alertDelete = (data) => {
    itemSelected.value = data
    openAlert.value = true
    alertData.headerLabel = 'Are you sure to delete this?'
    alertData.contentLabel = "You won't be able to revert this!"
    alertData.closeLabel = 'Cancel'
    alertData.submitLabel = 'Delete'
}

const closeAlert = () => {
    itemSelected.value = ref({})
    openAlert.value = false
}

const deleteNpwp = async () => {
    axios.delete(route('settings.company.npwp.delete', { 'id': itemSelected.value.id })
    ).then((res) => {
        notify({
            type: "success",
            group: "top",
            text: res.data.meta.message
        }, 2500)
        openAlert.value = false
        isLoading.value = true
        getData(pagination.value.current_page)
    }).catch((res) => {
        notify({
            type: "error",
            group: "top",
            text: res.response.data.message
        }, 2500)
    })
};

const getTypeStatus = (status) => {
    if (status === 'active') {
        return 'success';
    } else {
        return 'danger';
    }
}

onMounted(() => {
    getData(1);
});
</script>

<template>
    <Head :title="title"></Head>
    <VBreadcrumb :routes="breadcrumb" />
    <div class="mb-4 sm:mb-6 flex justify-between items-center">
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">Company Settings</h1>
    </div>
    <!-- Content -->
    <div class="bg-white shadow-lg rounded-sm mb-8">
        <div class="flex flex-col md:flex-row md:-mr-px">
            <VSidebarSetting :module="additional.menu" />
            <div class="grow overflow-scroll">
                <!-- Panel Header -->
                <div class="border-b">
                    <h2 class="text-2xl text-slate-800 font-bold p-6">NPWP</h2>
                </div>
                
                <div class="px-6 space-y-6">
                    <!-- Table  -->
                    <section>
                        <header class="block justify-between items-center sm:flex py-6">
                            <h2 class="font-semibold text-slate-800">
                                All NPWP <span class="text-slate-400 !font-medium ml">{{ pagination.total }}</span>
                            </h2>
                            <div class="mt-3 sm:mt-0 flex space-x-2 sm:justify-between justify-end">
                                <!-- Filter -->
                                <VFilter @search="searchNpwp" />
                                <VButton label="Add NPWP" type="primary" @click="handleAddModalForm" class="mt-auto"/>
                            </div>
                        </header>

                        <VDataTable :heads="heads" :isLoading="isLoading" wrapperClass="!px-0">
                            <tr v-if="isLoading">
                                <td class="h-[100%] overflow-hidden my-2" :colspan="heads.length">
                                    <VLoading />
                                </td>
                            </tr>
                            <tr v-else-if="query.length === 0 && !isLoading">
                                <td class="overflow-hidden my-2" :colspan="heads.length">
                                    <div class="flex items-center flex-col w-full my-32">
                                        <VEmpty />
                                        <div class="mt-9 text-slate-500 text-xl md:text-xl font-medium">Result not found.</div>
                                    </div>
                                </td>
                            </tr>
                            <tr v-for="(data, index) in query" :key="index" v-else>
                                <td class="px-4 whitespace-nowrap h-16"> {{ data.npwp_name }} </td>
                                <td class="px-4 whitespace-nowrap h-16"> {{ data.number_npwp }} </td>
                                <td class="px-4 whitespace-nowrap h-16"> {{ data.city ?? '-' }} </td>
                                <td class="px-4 whitespace-nowrap h-16"> {{ data.kpp ?? '-' }} </td>
                                <td class="px-4 whitespace-nowrap h-16"> {{ data.active_month_formatted }} </td>
                                <td class="px-4 whitespace-nowrap h-16 capitalize">
                                    <VBadge :text="data.status_formatted" :color="getTypeStatus(data.status_formatted)" size="sm" />
                                </td>
                                <td class="px-4 whitespace-nowrap h-16 text-right">
                                    <VDropdownEditMenu class="relative inline-flex r-0" :align="'right'"
                                        :last="index === query.length - 1 ? true : false">
                                        <li class="cursor-pointer hover:bg-slate-100" @click="handleEditModalForm(data)">
                                            <div class="flex items-center space-x-2 p-3">
                                                <span>
                                                    <VEdit />
                                                </span>
                                                <span>Edit</span>
                                            </div>
                                        </li>
                                        <li class="cursor-pointer hover:bg-slate-100" @click="alertDelete(data)">
                                            <div class="flex items-center space-x-2 p-3">
                                                <span>
                                                    <VTrash color="danger"/>
                                                </span>
                                                <span>Delete</span>
                                            </div>
                                        </li>
                                    </VDropdownEditMenu>
                                </td>
                            </tr>
                        </VDataTable>
                        <div class="py-6">
                            <VPagination :pagination="pagination" @next="nextPaginate" @previous="previousPaginate" />
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </div>
    <VModalForm :data="itemSelected" :update-action="updateAction" :open-dialog="openModalForm" @close="closeModalForm"
        @successSubmit="successSubmit" ref="modalForm" />
    <VAlert :open-dialog="openAlert" @closeAlert="closeAlert" @submitAlert="deleteNpwp" type="danger" :headerLabel="alertData.headerLabel" :content-label="alertData.contentLabel" :close-label="alertData.closeLabel" :submit-label="alertData.submitLabel"/>
</template>