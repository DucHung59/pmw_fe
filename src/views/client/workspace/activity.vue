<template>
    <Tabs value="0">
        <div class="flex flex-col items-center justify-center my-8 p-4">
            <template v-if="user">
                <div class="flex flex-col items-center" >
                    <button
                        class="w-16 h-16 rounded-full text-white flex items-center justify-center text-[24px] font-semibold uppercase border-2 border-black"
                        :style="{ backgroundColor: stringToColor(user.username) }"
                        v-tooltip.bottom="user.username"
                    >
                        {{ getInitial(user.username) }}
                    </button>
                    <p class="mt-4 font-medium text-[16px]">{{ user.username }}</p>
                    <span class="font-medium text-sm text-gray-500">({{ user.email }})</span>
                </div>
            </template>
            <TabList>
                <Tab value="0">Nhật ký</Tab>
                <Tab value="1">Công việc</Tab>
                <Tab value="2">Biểu đồ</Tab>
            </TabList>
        </div>
        <div class="mb-16">
            <TabPanels>
                <TabPanel value="0">
                    <div class="flex justify-center">
                        <div class="w-3xl border-2 border-gray-200 rounded-md p-4">
                            <p class="mb-4 text-lg font-medium">Nhật ký gần đây</p>
                            <template v-for="log in userLogs">
                                <div class="border-b border-gray-300 px-2 py-4 bg-cyan-50 hover:bg-cyan-100 cursor-default">
                                    <div class="flex gap-2 justify-between items-center">
                                        <div class="flex gap-2">
                                            <div>
                                                <span v-if="log.type == 'add' "class="px-4 py-1 text-white text-sm font-medium rounded-full bg-emerald-500">
                                                    Thêm mới
                                                </span>
                                                <span v-if="log.type == 'update' "class="px-4 py-1 text-white text-sm font-medium rounded-full bg-amber-500">
                                                    Cập nhật
                                                </span>
                                                <span v-if="log.type == 'delete' "class="px-4 py-1 text-white text-sm font-medium rounded-full bg-rose-500">
                                                    Xóa bỏ
                                                </span>
                                            </div>
                                            <div>
                                                <span class="text-lg">{{ log.user.username }}</span>
                                            </div>
                                        </div>
                                        <div>
                                            <span>{{ dayjs(log.created_at).format('DD/MM/YYYY') }}</span>
                                        </div>
                                    </div>
                                    <div class="px-4 py-2">
                                        <div class="overflow-hidden text-ellipsis whitespace-nowrap w-full max-w-2xl">
                                            <span v-html="log.description"></span>
                                        </div>
                                    </div>
                                </div>
                            </template>
                            <template v-if="!isLogLoading">
                                <div class="text-center mt-4" v-if="logsTotal >= logsPerPage">
                                    <Button icon="pi pi-angle-down" rounded variant="outlined" @click="viewMoreLogs"/>
                                </div>
                                <div v-else class="text-center p-4">
                                    <span class="text-gray-400 font-medium">Đã đến cuối trang</span>
                                </div>
                            </template>
                            <template v-else>
                                <div class="text-center mt-4">
                                    <i class="pi pi-spin pi-spinner" style="font-size: 1.4rem; color: var(--p-primary-400)"></i>
                                </div>
                            </template>
                        </div>
                    </div>
                </TabPanel>
                <TabPanel value="1">
                    <div class="border-2 border-gray-200 rounded-md p-4 mx-16">
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
                            <template v-if="userTask && Object.keys(userTask).length === 0">
                                <tr class="text-center">
                                    <td colspan="8" class="text-center py-3 cursor-default font-medium text-gray-500">Không có bản ghi</td>
                                </tr>
                            </template>
                            <template v-else>
                                <template v-for="(tasks, projectName, index) in userTask" :key="projectName">
                                    <tr class="bg-gray-50">
                                        <td colspan="8" class="text-left px-4 py-2 font-bold cursor-default " style="color: var(--p-primary-500);">
                                            {{ index + 1 }} - {{ projectName }}
                                        </td>
                                    </tr>
                                    <template v-for="task in tasks">
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
                            </template> 
                        </tbody>
                    </table>
                    <Paginator
                        :rows="tasksPerpage"
                        :totalRecords="taskTotal"
                        :first="(taskCurrentPage - 1) * tasksPerpage"
                        @page="(event) => onPageChange(event)"
                    />
                    </div>
                </TabPanel>
                <TabPanel value="2">
                    <template v-if="projectList.length > 0">
                        <template v-for="project in projectList" :key="project.project_id">
                            <div class="m-6">
                                <p class="text-lg font-semibold mb-2">{{ project.project_name }} <span class="text-sm">(Tổng công việc: {{ project.user_task_count }} / {{ project.total_tasks }} - {{ project.user_task_percentage }}%)</span></p>
                                <MeterGroup :value="project.statuses" labelPosition="end" labelOrientation="horizontal"/>
                            </div>
                        </template>
                    </template>
                    <template v-else>
                        <p>Không có công việc nào được hiển thị</p>
                    </template>
                </TabPanel>
            </TabPanels>
        </div>
    </Tabs>
</template>

<script setup>
import api from '@/api/axios';
import { toastService } from '@/assets/js/toastHelper';
import { useUserStore } from '@/store/user';
import dayjs from 'dayjs';
import { Tab, TabList, TabPanel, Tabs, useToast, Button, Paginator, MeterGroup } from 'primevue';
import { computed, onMounted, ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const toast = new toastService(useToast());

const route = useRoute();
const router = useRouter();
const userStore = useUserStore();
const workspace = userStore.workspace;

const user_id = computed(() => route.params.user_id);

const user = ref();
const userLogs = ref({});
const logsTotal = ref(0);
const logsPerPage = ref(15);
const isLogLoading = ref(false);

const userTask = ref({});
const taskTotal = ref(0);
const taskCurrentPage = ref(1);
const tasksPerpage = 10;

const projectList = ref([]);

//UI Function
function getInitial(name) {
  return name
    ?.trim()
    .split(' ')
    .filter(n => n)
    .map(w => w.charAt(0).toUpperCase())
    .slice(0, 1)
    .join('') || '?'
}

function stringToColor(str) {
    let hash = 0
    for (let i = 0; i < str.length; i++) {
        hash = str.charCodeAt(i) + ((hash << 5) - hash)
    }
    let color = '#'
    for (let i = 0; i < 3; i++) {
        const value = (hash >> (i * 8)) & 0xff
        color += ('00' + value.toString(16)).slice(-2)
    }
    return color
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

function viewMoreLogs() {
    const perPage = logsPerPage.value + 10;
    getLogs(perPage);
}

function onPageChange(event) {
    const page = event.page + 1;
    getAllTasks(page)
}

async function getUser() {
    try {
        const response = await api.get('/workspace/user-info', {
            params: {
                user_id: user_id.value
            }
        })

        const result = response.data;
        if (result.success) {
            user.value = result.user;
        } else {
            toast.error(result.message, 'Lỗi')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function getLogs(perPage = 15) {
    try {
        isLogLoading.value = true;
        const response = await api.get('log/getByUser', {
                params: {
                    workspace_id: workspace.id,
                    user_id: user.value.id,
                    perPage: perPage,
                }
            }
        )

        const result = response.data;
        userLogs.value = result.logs.data;
        logsTotal.value = result.logs.total;
        logsPerPage.value = result.logs.per_page;
    } catch (error) {
        console.log(error.message);
    } finally {
        isLogLoading.value = false;
    }
}

async function getTasksByUser(page = 1) {
    try {
        const response = await api.get('/task/getByUser', {
            params: {
                workspace_id: workspace.id,
                viewer_id: userStore.user.id,
                user_id: user.value.id,
                page: page,
            }
        })

        const result = response.data;
        if(result.success) {
            userTask.value = result.tasks.data;
            taskCurrentPage.value = result.tasks.current_page;
            taskTotal.value = result.tasks.total;
        }
        console.log('userTask:', userTask.value);

    } catch (error) {
        console.log(error.message);
        
    }
}

async function getReport(){
    try {
        const response = await api.get('/workspace/get/chart-user', {
            params: {
                user_id: user.value.id,
            }
        })
        const result = response.data;
        console.log(result);
        projectList.value = result.data;

        
    } catch (error) {
        
    }
}

onMounted(async () => {
    await getUser();
    getLogs();
    getTasksByUser();
    getReport();
})

watch(user_id, async () => {
    await getUser();
    getLogs();
    getTasksByUser();
    getReport();
})
</script>
    