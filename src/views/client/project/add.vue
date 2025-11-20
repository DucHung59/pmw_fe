<template>
    <Sidebar :active="'add task'" :project_key="project.project_key" />
    <div style="margin-left: 60px;">
        <div class="project-header flex items-center justify-between">
            <div class="flex items-center gap-2" v-if="!project.project_name">
                <Skeleton class="w-10 h-10 rounded-full" />
            </div>
            <div class="project-name flex items-center gap-2" v-else>
                <p>{{ project.project_name }}</p>
                <p class="text-[14px]">({{ project.project_key }})</p>
            </div>
        </div>
        <div class="m-16">
            <div class="flex items-center justify-between">
                <div>
                    <p class="text-2xl font-semibold">Thêm công việc/nhiệm vụ</p>
                </div>
            </div>
            <div class="mt-4">
                <div class="flex flex-col gap-2">
                    <div>
                        <Select v-model="selectedIssue" :options="issues" optionLabel="category.category_type" optionValue="category.id" placeholder="Chọn danh mục" class="w-full md:w-56" />
                    </div>
                    <InputText v-model="subject" placeholder="Tên công việc (bắt buộc)" class="w-full" />
                </div>
                <div class="mt-4 border border-gray-300 rounded-md p-4">
                    <div class="flex flex-col gap-2 my-4">
                        <Editor v-model="description" editorStyle="height: 320px" />
                    </div>
                    <div class="grid grid-cols-2 gap-8">
                        <div class="grid grid-cols-[12rem_1fr] items-center justify-around gap-2 border-b border-gray-400 p-4">
                            <p>Trạng thái</p>
                            <Select v-model="selectedStatus" :options="statuses" optionLabel="status.status_type" optionValue="status.id" placeholder="Chọn trạng thái" class="w-full md:w-56" />
                        </div>
                        <div class="grid grid-cols-[12rem_1fr] items-center justify-around gap-2 border-b border-gray-400 p-4">
                            <p>Người được giao</p>
                            <Select v-model="assignee" :options="members" optionLabel="user.username" optionValue="user.id" placeholder="Chọn " class="w-full md:w-56" />
                        </div>
                        <div class="grid grid-cols-[12rem_1fr] items-center justify-around gap-2 border-b border-gray-400 p-4">
                            <p>Mức độ ưu tiên</p>
                            <Select v-model="priority" :options="priorities" optionLabel="label" optionValue="value" placeholder="Chọn " class="w-full md:w-56" />
                        </div>
                        <div class="grid grid-cols-[12rem_1fr] items-center justify-around gap-2 border-b border-gray-400 p-4">
                            <p>Hạn chót</p>
                            <DatePicker v-model="due_date" placeholder="Chọn hạn chót" showIcon  dateFormat="dd/mm/yy" class="w-full md:w-56" />
                        </div>
                    </div>
                </div>
                <div class="flex justify-end my-4">
                    <Button :loading="isCreating" loadingIcon="pi pi-spin pi-spinner" icon="pi pi-plus" label="Thêm công việc" @click="addTask" />
                </div>
            </div>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import { toastService } from '@/assets/js/toastHelper';
import Sidebar from '@/components/common/Sidebar.vue';
import { useUserStore } from '@/store/user';
import { Button, Select, InputText, DatePicker, useToast } from 'primevue';
import Editor from 'primevue/editor';
import { ref, computed, onMounted, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();
const project_key = computed(() => route.params.project_key);
const userStore = useUserStore();

const project = ref({});
const issues = ref([]);
const statuses = ref([]);
const members = ref([]);
const priorities = [
    {label: 'Thấp', value: 'low'},
    {label: 'Bình thường', value: 'normal'},
    {label: 'Cao', value: 'high'},
]

const subject = ref('')
const selectedIssue = ref(null);
const description = ref('');
const selectedStatus = ref(null);
const assignee = ref(null);
const due_date = ref(null);
const priority = ref('normal')

const isCreating = ref(false);

const toast = new toastService(useToast());

function validated() {
    const errors = [];

    if (!subject.value.trim()) {
        errors.push('Tiêu đề không được để trống');
    }

    if (!selectedIssue.value) {
        errors.push('Vui lòng chọn loại danh mục');
    }

    if (!selectedStatus.value) {
        errors.push('Vui lòng chọn trạng thái');
    }

    return errors;
}

async function getProject() {
    try {
        const response = await api.get('project/detail', {
            params: {
                project_key: project_key.value
            }
        })

        project.value = response.data.project;
        issues.value = response.data.issues;
        statuses.value = response.data.statuses;
        members.value = response.data.project.members;
        userStore.setProjectContext(project.value.id);
    } catch (error) {
        console.log(error.message);
    }
}

async function addTask() {
    const errors = validated();
    if( errors.length > 0 ) {
        errors.forEach(error => {
            toast.error(error, "Chọn thông tin");
        });
        return;
    }

    try {
        isCreating.value = true;
        const response = await api.post('task/create', {
            project_id: project.value.id,
            subject: subject.value,
            project_key: project_key.value,
            category: selectedIssue.value,
            priority: priority.value,
            status: selectedStatus.value,
            assignee: assignee.value,
            description: description.value,
            due_date: due_date.value,
        })

        const result = response.data;
        if(result.success == true) {
            router.push(`/workspace/task/${project_key.value}`)
            toast.success('Thêm mới thành công', 'Thông báo')
        } else {
            console.log(result.message);
        }
    } catch (error) {
        console.log(error.message); 
    } finally {
        isCreating.value = false
    }
}


watch(project_key, () => {
    getProject();
});
</script>