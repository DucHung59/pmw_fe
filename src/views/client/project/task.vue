<template>
    <Sidebar :active="'tasks'" :project_key="project.project_key" />
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
        <div class="mt-4">
            <div class="my-4 p-4">
                <label class="font-medium text-2xl">Tìm kiếm</label>
                <div class="flex items-center justify-between m-4">
                    <div class="flex items-center gap-2">
                        <label class="font-medium" for="selectStatus">Trạng thái:</label>
                        <SelectButton id="selectStatus" v-model="statusFilter" :options="statuses" optionLabel="status.status_type" optionValue="status.id" @change="onStatusChange" />
                    </div>
                    <div v-if="userStore.isSystemAdmin || userStore.projectRole == 'PManager'" class="flex items-center gap-2">
                        <label class="font-medium" for="isDelEnabel">Đã xóa:</label>
                        <ToggleButton id="isDelEnabel" v-model="isDelEnable" onLabel="On" offLabel="Off" @change="isDelChange"/>
                    </div>
                </div>
                <div class="m-4 flex items-center justify-between">
                    <div class="m-4 flex items-center gap-2">
                        <div class="flex flex-col gap-1 mx-4">
                            <label for="" class="font-medium">Danh mục</label>
                            <Select v-model="categoryFilter" :options="issues" optionLabel="category.category_type" optionValue="category.id" class="w-40" />
                        </div>
                        <div class="flex flex-col gap-1 mx-4">
                            <label for="" class="font-medium">Assignee</label>
                            <Select v-model="assigneeFilter" :options="project.members" optionLabel="user.username" optionValue="user.id" class="w-40" />
                        </div>
                        <div class="flex flex-col gap-1 mx-4">
                            <label for="" class="font-medium">Từ khóa</label>
                            <InputText type="text" v-model="keyword" class="w-60" />
                        </div>
                    </div>
                    <div class="flex flex-col gap-1">
                        <Button icon="pi pi-filter-slash" :loading="isTaskLoading" loadingIcon="pi pi-spin pi-spinner" label="Đặt lại" size="small" @click="clearFilter"/>
                        <Button icon="pi pi-search" :loading="isTaskLoading" loadingIcon="pi pi-spin pi-spinner" label="Tìm kiếm" size="small" @click="tasksFilter"/>
                    </div>
                </div>
            </div>
            <div class="my-4 p-4">
                <div>
                    <table class="w-full text-sm tr-border">
                        <thead>
                            <tr>
                                <th class="px-3 py-2 border">Danh mục</th>
                                <th class="px-3 py-2 border">Key</th>
                                <th class="px-3 py-2 border">Tiêu đề</th>
                                <th class="px-3 py-2 border">Trạng thái</th>
                                <th class="px-3 py-2 border">Người được giao</th>
                                <th class="px-3 py-2 border">Ngày hạn</th>
                                <th class="px-3 py-2 border">Ngày tạo</th>
                                <th class="px-3 py-2 border">Người tạo</th>
                            </tr>
                        </thead>
                        <tbody :class="isTaskLoading ? 'opacity-50 pointer-events-none select-none' : ''">
                            <template v-if="tasks.length == 0">
                                <tr class="text-center">
                                    <td colspan="8" class="text-center py-3 cursor-default font-medium text-gray-500">Không có bản ghi</td>
                                </tr>
                            </template>
                            <template v-else>
                                <template v-for="(task, index) in tasks">
                                    <tr class="cursor-default hover:bg-cyan-100">
                                            <td class="text-center py-3"> 
                                                <span class="px-4 py-1 text-white font-medium rounded-full" 
                                                    :style="{ backgroundColor: task.category_info.category_color }">
                                                    {{task.category_info.category_type}}
                                                </span>
                                            </td>
                                            <td class="text-center py-3">
                                                <RouterLink :to="'/workspace/view/' + task.task_key">
                                                    {{ task.task_key }}
                                                </RouterLink>
                                            </td>
                                            <td class="text-center py-3">
                                                <RouterLink :to="'/workspace/view/' + task.task_key">
                                                    {{ task.subject }}
                                                </RouterLink>
                                            </td>
                                            <td class="text-center py-3">
                                                <span class="px-4 py-1 text-white font-medium rounded-full" 
                                                    :style="{ backgroundColor: task.status_info.status_color }">
                                                    {{task.status_info.status_type}}
                                                </span>
                                            </td>
                                            <td class="text-center py-3">{{ task.assignee_user ? task.assignee_user.username : 'Không có' }}</td>
                                            <td class="text-center py-3">{{ task.due_date ? formatDate(task.due_date) : "Không có" }}
                                                <FlameIcon v-if="task.due_date && dayjs(task.due_date).isBefore(dayjs(), 'day') && task.status_info.status_type != 'Closed'"/>
                                            </td>
                                            <td class="text-center py-3">{{ formatDate(task.created_at) }}</td>
                                            <td class="text-center py-3">{{ task.creator.username }}</td>
                                        </tr>
                                </template>
                            </template>
                        </tbody>
                    </table>
                </div>
                <Paginator
                    :rows="perPage"
                    :totalRecords="total"
                    :first="(currentPage - 1) * perPage"
                    @page="(event) => onPageChange(event)"
                />
            </div>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import Sidebar from '@/components/common/Sidebar.vue';
import { onMounted, ref, computed, watch } from 'vue';
import { useRoute } from 'vue-router';
import { Paginator, SelectButton, Select, Button, InputText, ToggleButton  } from 'primevue';
import dayjs from 'dayjs';
import FlameIcon from '@/components/icons/FlameIcon.vue';
import { useUserStore } from '@/store/user';


const route = useRoute();
const project_key = computed(() => route.params.project_key);
const userStore = useUserStore();

const project = ref({});
const statuses = ref({});
const issues = ref({});


const isDelEnable = ref(false);
const statusFilter = ref();
const categoryFilter = ref();
const assigneeFilter = ref();
const keyword = ref();

const isTaskLoading = ref(false);
const perPage = 10;
const total = ref(0);
const currentPage = ref(1);

const tasks = ref({});

function onPageChange(event) {
    const page = event.page + 1;
    getAllTasks(page)
}

function formatDate(dateStr) {
    const date = new Date(dateStr);
    const day = String(date.getDate()).padStart(2, '0');
    const month = String(date.getMonth() + 1).padStart(2, '0'); // Tháng bắt đầu từ 0
    const year = date.getFullYear();
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    return `${day}/${month}/${year} ${hours}:${minutes}`;
}

function onStatusChange() {
    getAllTasks();
}

function clearFilter() {
    statusFilter.value = null;
    categoryFilter.value = null;
    assigneeFilter.value = null;
    keyword.value = null;
    isDelEnable.value = false;
    getAllTasks();
}

function tasksFilter() {
    getAllTasks();
}

function isDelChange() {
    getAllTasks();
}

async function getProject() {
    try {
        const response = await api.get('project/detail', {
            params: {
                project_key: project_key.value
            }
        })

        project.value = response.data.project;
        statuses.value = response.data.statuses;
        issues.value = response.data.issues;
        await userStore.setProjectContext(project.value.id);
    } catch (error) {
        console.log('Có lỗi xảy ra: ' + error.message);
    }
}

async function getAllTasks(page = 1) {
    try {
        isTaskLoading.value = true;
        const response = await api.get('task/get', {
            params: {
                project_id : project.value.id,
                status : statusFilter.value,
                category : categoryFilter.value,
                assignee : assigneeFilter.value,
                search : keyword.value,
                isDelEnable: isDelEnable.value,
                page: page,
            }
        })

        const result = response.data;
        currentPage.value = result.task.current_page;
        total.value = result.task.total
        tasks.value = result.task.data;
    } catch (error) {
        console.log(error.message);
    } finally {
        isTaskLoading.value = false;
    }
    
}

onMounted(async () => {
    await getProject();
    getAllTasks();
})

watch(project_key, () => {
    getProject();
});


</script>