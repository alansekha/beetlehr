<script>
export default {
    layout: AppLayout
}
</script>
<script setup>
import axios from "axios";
import { notify } from "notiwind";
import { reactive, ref } from "vue";
import { object, string } from "vue-types";
import { Head } from "@inertiajs/inertia-vue3";
import AppLayout from '@/layouts/apps.vue';
import { Inertia } from '@inertiajs/inertia'
import VBreadcrumb from '@/components/VBreadcrumb/index.vue';
import VSidebarSetting from '@/components/VSidebarSetting/index.vue';
import VSwitch from '@/components/VSwitch/index.vue';
import VButton from '@/components/VButton/index.vue';
import dayjs from "dayjs";

const breadcrumb = [
    {
        name: "Dashboard",
        active: false,
        to: route('dashboard.index')
    },
    {
        name: "Systems Settings",
        active: false,
        to: route('settings.leave.general.index')
    },
    {
        name: "General",
        active: true,
        to: route('settings.leave.general.index')
    },
]


const submitLoading = ref(false)
const formError = ref({})
const form = reactive({
    date_reset_leave_quota: props.additional.data.date_reset_leave_quota,
})

const handleDateReset = () => {
    if (form.date_reset_leave_quota) {
        const date = new Date(form.date_reset_leave_quota.getFullYear(), form.date_reset_leave_quota.getMonth(), form.date_reset_leave_quota.getDate());
        const newDate = dayjs(date).format('YYYY-MM-DD');
        form.date_reset_leave_quota = newDate
    }
}



const submit = () => {
    submitLoading.value = true
    axios.post(route('settings.leave.general.update'), {
        date_reset_leave_quota: form.date_reset_leave_quota,
    }).then((res) => {
        notify({
            type: "success",
            group: "top",
            text: res.data.meta.message
        }, 2500)
        Inertia.reload()
    }).catch((res) => {
        // Handle validation errors
        const result = res.response.data
        if (result.hasOwnProperty('errors')) {
            formError.value = ref({});
            Object.keys(result.errors).map((key) => {
                formError.value[key] = result.errors[key].toString();
            });
        }

        notify({
            type: "error",
            group: "top",
            text: result.message
        }, 2500)
    }).finally(() => submitLoading.value = false)
}

const discard = () => {
    form.date_reset_leave_quota = props.additional.data.close_break_page
}

const props = defineProps({
    additional: object(),
    title: string()
})

</script>

<template>

    <Head :title="title"></Head>
    <VBreadcrumb :routes="breadcrumb" />
    <div class="mb-4 sm:mb-6 flex justify-between items-center">
        <h1 class="text-2xl md:text-3xl text-slate-800 font-bold">Leave Management Settings</h1>
    </div>
    <!-- Content -->
    <div class="bg-white shadow-lg rounded-sm mb-8">
        <div class="flex flex-col md:flex-row md:-mr-px">
            <VSidebarSetting :module="additional.menu" />
            <div class="grow overflow-scroll">
                <!-- Panel Header -->
                <div class="border-b">
                    <h2 class="text-2xl text-slate-800 font-bold p-6">General</h2>
                </div>

                <!-- Panel Content -->
                <div class="p-6 space-y-6">
                    <h3 class="text-xl leading-snug text-slate-800 font-bold mb-1">Reset Leave Qouta</h3>
                    <div class="text-sm text-slate-500">Set the date to reset Leave Quota in 1 year, fill it in to
                        reset the leave quota for each employee according to the initial quota that has been set on
                        the Leave Type menu.</div>
                    <div class="sm:flex sm:items-center space-y-4 sm:space-y-0 sm:space-x-4 mt-5">
                        <div class="sm:w-1/3">
                            <label class="block text-sm font-medium text-slate-600 mb-1">
                                Date for Reset Leave Quota <span class="text-rose-500">*</span>
                            </label>
                            <Datepicker v-model="form.date_reset_leave_quota" @update:modelValue="handleDateReset"
                                single-picker :enableTimePicker="false" position="left" :clearable="false"
                                format="dd MMMM yyyy" previewFormat="dd MMMM yyyy"
                                placeholder="Insert Date Reset Leave Qouta"
                                :class="{ 'date_error': formError.date_reset_leave_quota }" />
                            <div class="text-xs" :class="[{
                                'text-rose-500': formError.date_reset_leave_quota,
                            }]" v-if="formError.date_reset_leave_quota">
                                {{ formError.date_reset_leave_quota }}
                            </div>
                        </div>
                    </div>

                    <!-- Panel footer -->
                    <footer>
                        <div class="flex flex-col px-6 py-3 border-t border-slate-200">
                            <div class="flex self-end space-x-3">
                                <VButton :is-loading="submitLoading" label="Discard" type="default" @click="discard" />
                                <VButton :is-loading="submitLoading" label="Save" type="primary" @click="submit" />
                            </div>
                        </div>
                    </footer>
                </div>
            </div>
        </div>
    </div>
</template>