<template>
    <div class="px-10 pt-8">
        <div class="flex justify-center items-center gap-4 pt-4 pb-10" v-if="userStore.role === 'manager' || userStore.isSystemAdmin">
            <img :src="'/app_icon.png'" alt="hehee" width="60px"/>
            <p class="text-[24px] font-semibold">{{ workspace.workspace_name }}</p>
            <RouterLink to="settings">
                <Button icon="pi pi-cog" rounded variant="outlined" severity="contrast"/>
            </RouterLink>
        </div>
        <div class="flex gap-8">
            <div>
                <div class="accordion mb-10">
                    <div class="accordion-title flex justify-between">
                        <div class="toggle flex items-center gap-1.5" @click="toggleOpen('prj')">
                            <i class="pi " :class="isPrjOpen ? 'pi-chevron-up' : 'pi-chevron-down'"></i>
                            <p>Dự án</p>
                        </div>
                        <div>
                            <Button class="btn-add" size="small" icon="pi pi-plus" rounded variant="outlined" v-tooltip.top="'Thêm Project'" v-if="userStore.role === 'manager' || userStore.isSystemAdmin" v-show="isPrjOpen" @click="addProjectDialog = true"/>
                            <Dialog v-model:visible="addProjectDialog" :style="{width: '50vw'}" @hide="onDialogHide" header="Thêm Dự án mới" :draggable="false" maximizable="true" :modal="true">
                                <div class="flex flex-col gap-2 pt-4">
                                    <label for="project-name" class="pl-4">Tên dự án</label>
                                    <InputText id="project-name" v-model="project_name" aria-describedby="project-name-help" />
                                    <Message severity="error" class="pl-4 italic" size="small" variant="simple" v-if="project_name_error">Tên dự án không được để trống</Message>
                                </div>
                                <div class="flex flex-col gap-2 pt-4">  
                                    <label for="project-key" class="pl-4">Khóa dự án</label>
                                    <InputText id="project-key" v-model="project_key" aria-describedby="project-key-help" @input="toUppercase"/>
                                    <Message :severity="project_key_error ? 'error' : 'secondary'" class="pl-4 italic" size="small" variant="simple">Khóa dự án không được để trống, chỉ bao gồm các chữ cái in hoa, số và gạch dưới</Message>
                                </div>
                                <div class="flex flex-col gap-2 pt-4">
                                    <label for="description">Mô tả dự án</label>
                                    <Editor v-model="description" editorStyle="height: 120px"/>                                
                                </div>
                                <div class="flex flex-col gap-2 pt-4">  
                                    <label for="project-key" class="pl-4">Ngày bắt đầu</label>
                                    <DatePicker v-model="start_date" placeholder="Chọn ngày bắt đầu" showIcon  dateFormat="dd/mm/yy" class="w-full md:w-56" />
                                    <Message severity="error" class="pl-4 italic" size="small" variant="simple" v-if="start_date_error">Không được chọn ngày trong quá khứ</Message>
                                </div>
                                <div class="flex flex-col gap-2 pt-4">  
                                    <label for="project-key" class="pl-4">Ngày kết thúc</label>
                                    <DatePicker v-model="end_date" placeholder="Chọn ngày kết thúc" showIcon  dateFormat="dd/mm/yy" class="w-full md:w-56" />
                                    <Message severity="error" class="pl-4 italic" size="small" variant="simple" v-if="end_date_error">Ngày kết thúc phải sau ngày bắt đầu</Message>
                                </div>
                                <div class="flex justify-around mt-4 pt-4">
                                    <Button label="Hủy" class="w-75" severity="secondary" size="small" variant="outlined" @click="addProjectDialog = false"/>
                                    <Button label="Thêm" class="w-75" size="small" variant="outlined" @click="addProject" :loading="isAddProjectLoading" loadingIcon="pi pi-spin pi-spinner"/>
                                </div>
                            </Dialog>
                        </div>
                    </div>
                    <div class="accordion-content" v-show="isPrjOpen">
                        <template v-if="projects.length > 0">
                            <template v-for="project in projects" :key="project.id">
                                <div class="accordion-item">
                                    <RouterLink class="relative group w-max" :to="`project/${project.project_key}`">
                                        <div class="project-item gap-2 flex">
                                            <img :src="'/project.png'" width="40px"/>
                                            <div>
                                                <p>{{ project.project_name }}</p>
                                                <p class="text-[12px] project-key absolute group-hover:hidden" style="color: var(--color-gray-500);">
                                                    {{ project.project_key }}
                                                </p>
                                                <div class="text-[12px] list-action hidden group-hover:block">
                                                    <ul class="flex gap-2">
                                                        <li>
                                                            <RouterLink :to="'/workspace/add/' + project.project_key">
                                                                Thêm Task
                                                            </RouterLink>
                                                        </li>
                                                        <li>
                                                            <RouterLink :to="'/workspace/task/' + project.project_key">
                                                                Tasks
                                                            </RouterLink>
                                                        </li>
                                                        <li>
                                                            <RouterLink :to="'/workspace/settings/' + project.project_key">
                                                                Cài đặt
                                                            </RouterLink>
                                                        </li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                    </RouterLink>
                                </div>
                            </template>
                        </template>
                        <template v-else>
                            <div class="text-center border-t border-gray-400 p-4 cursor-default">
                                <span class="font-medium text-gray-500">Không có dự án đã tham gia</span>
                            </div>
                        </template>
                    </div>
                </div>
                <div class="accordion mb-10">
                    <div class="accordion-title" >
                        <div class="toggle flex items-center gap-1.5" @click="toggleOpen('issue')">
                            <i class="pi " :class="isIssueOpen ? 'pi-chevron-up' : 'pi-chevron-down'"></i>
                            <p>Công việc</p>
                        </div>
                    </div>
                    <div class="accordion-content" v-show="isIssueOpen">
                        <div class="filter-issue p-4">
                            <div class="flex items-center gap-2">
                                <label for="">Filter:</label>
                                <SelectButton v-model="meFilter" :options="me" optionLabel="label" optionValue="value" @change="onTaskFilterChange"/>
                            </div>
                        </div>
                        <div class="issue-item">
                            <table class="w-full text-sm tr-border">
                                <thead>
                                    <tr>
                                        <th class="px-3 py-2 border">Key</th>
                                        <th class="px-3 py-2 border w-48">Tiêu đề</th>
                                        <th class="px-3 py-2 border">Trạng thái</th>
                                        <th class="px-3 py-2 border">Ngày hạn</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <template v-if="isTasksLoading">
                                        <tr class="">
                                            <td class="py-3 px-2" colspan="4">
                                                <Skeleton class="mb-2"></Skeleton>
                                            </td>
                                        </tr>
                                    </template>
                                    <template v-else>
                                        <template v-if="tasks.length <= 0">
                                            <tr class="cursor-default hover:bg-cyan-100">
                                                <td colspan="4" class="text-center py-3 text-gray-400 font-medium">Không có task được hiển thị</td>
                                            </tr>
                                        </template>
                                        <template v-else>
                                            <template v-for="task in tasks">
                                                <tr class="cursor-default hover:bg-cyan-100">
                                                    <td class="text-center py-3">
                                                        <RouterLink :to="'/workspace/view/' + task.task_key">
                                                            {{ task.task_key }}
                                                        </RouterLink>
                                                    </td>
                                                    <td class="text-center py-3 w-48">
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
                                                    <td class="text-center py-3">{{ task.due_date ? task.due_date : 'Không có' }}</td>
                                                </tr>
                                            </template>
                                        </template>
                                    </template>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
            <div class="w-full mb-16">
                <div class="accordion">
                    <div class="accordion-title flex justify-between" >
                        <div class="toggle flex items-center gap-1.5" @click="toggleOpen('recent')">
                            <i class="pi " :class="isRecentOpen ? 'pi-chevron-up' : 'pi-chevron-down'"></i>
                            <p>Hoạt động gần đây</p>
                        </div>
                    </div>
                    <div class="w-full" v-show="isRecentOpen">
                        <template v-for="log in logs">
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
            </div>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import { toastService } from '@/assets/js/toastHelper';
import { useUserStore } from '@/store/user';
import dayjs from 'dayjs';
import { Button, Dialog, InputText, Message, SelectButton, useToast, Skeleton, DatePicker } from 'primevue';
import Editor from 'primevue/editor';
import { onMounted, ref, watch } from 'vue';


const userStore = useUserStore();
const workspace = ref(userStore.workspace);
const toast = new toastService(useToast());

const isPrjOpen = ref(true);
const isIssueOpen = ref(true);
const isRecentOpen = ref(true);
const addProjectDialog = ref(false);
const isLogLoading = ref(false);

//data project
const project_name = ref('');
const project_key = ref('');
const description = ref('');
const start_date = ref(null);
const end_date = ref(null);
const project_name_error = ref(false);
const project_key_error = ref(false);
const start_date_error = ref(false);
const end_date_error = ref(false);

const projects = ref([]);
const meFilter = ref("AssignTo");

//data task
const tasks = ref({});

//data log
const logs = ref({});

//Paginate
const perPage = 10;
const total = ref(0);
const currentPage = ref(1);

const logsPerPage = ref(15);
const logsTotal = ref(0);

//handle UI
const isAddProjectLoading = ref(false);
const isTasksLoading = ref(false);
const showFull = ref(false);

const me = [
    { label: "Dành cho tôi", value: "AssignTo"},
    { label: "Tạo bởi tôi", value: "CreatedBy" }
]


function viewMoreLogs() {
    const perPage = logsPerPage.value + 10;
    getLogsByWorkspace(perPage);
}

function toUppercase(e) {
  project_key.value = e.target.value.toUpperCase()
}

function checkValidated() {
    const project_key_regex = /^[A-Z0-9_]+$/;
    const today = dayjs().format('YYYY-MM-DD');

    let isValid = true;

    if(!project_key_regex.test(project_key.value)) {
        project_key_error.value = true;
        isValid = false;
    }
    if(project_name.value.length === 0) {
        project_name_error.value = true;
        isValid = false;
    }
    if (dayjs(start_date.value).isBefore(today, 'day')) {
        start_date_error.value = true;
        isValid = false;
    }

    if ((!start_date.value && end_date.value) || dayjs(end_date.value).isBefore(dayjs(start_date.value), 'day')) {
        end_date_error.value = true;
        isValid = false;
    }
    return isValid;
}

function onDialogHide() {
  project_key.value = '';
  project_name.value = '';
  description.value = '';
  start_date.value = null;
  end_date.value = null;
}

function toggleOpen(cate) {
    if(cate === 'prj') isPrjOpen.value = !isPrjOpen.value;
    else if(cate === 'issue') isIssueOpen.value = !isIssueOpen.value;
    else if (cate === 'recent') isRecentOpen.value = !isRecentOpen.value;
}

function onTaskFilterChange() {
    if(meFilter.value == 'AssignTo') {
        getTasksByAssginee();
    } else if (meFilter.value == 'CreatedBy') {
        getTasksByCreator();
    }
}

//add project
async function addProject() {
    if(!checkValidated()) return;
    try {
        isAddProjectLoading.value = true;
        const response = await api.post('/project/create', {
            workspace_id: workspace.value.id,
            project_name: project_name.value,
            project_key: project_key.value,
            description: description.value,
            start_date: start_date.value,
            end_date: end_date.value,
        })

        toast.success('Thêm dự án thành công');
        addProjectDialog.value = false;
        getProject();
    } catch (error) {
        toast.error('Thêm dự án thất bại');
    } finally {
        isAddProjectLoading.value = false;
    }
}

async function getProject() {
    try {
        const response = await api.get('/project/get');
    
        projects.value = response.data.projects;
    } catch (error) {
        console.log(error);
    }
}

async function getTasksByAssginee() {
    try {
        isTasksLoading.value = true;
        const response = await api.get('task/getByAssignee')

        const result = response.data;
        if(result.success) {
            tasks.value = result.task.data;
            total.value - result.total;
            currentPage.value = result.current_page;
        }
        
    } catch (error) {
        console.log(error.message);
    } finally {
        isTasksLoading.value = false;
    }
}


async function getTasksByCreator() {
    try {
        isTasksLoading.value = true;
        const response = await api.get('task/getByCreator')

        const result = response.data;
        if(result.success) {
            tasks.value = result.task.data;
            total.value - result.total;
            currentPage.value = result.current_page;
        }
    } catch (error) {
        console.log(error.message);
    } finally {
        isTasksLoading.value = false;
    }
}

async function getLogsByWorkspace(perPage = 15) {
    try {
        isLogLoading.value = true;
        const response = await api.get('log/getByWorkspace', {
            params: {
                workspace_id: workspace.value.id,
                perPage: perPage,
            }
        })

        const result = response.data;
        logs.value = result.logs.data;
        logsTotal.value = result.logs.total;
        logsPerPage.value = result.logs.per_page;
    } catch (error) {
        console.log(error.message);
    } finally {
        isLogLoading.value = false;
    }
}

onMounted(() => {
    getProject();
    getTasksByAssginee();
    getLogsByWorkspace();
})

watch(project_key, (newVal) => {
    project_key_error.value = false;
})

watch(project_name, (newVal) => {
    project_name_error.value = false;
})

watch(start_date, (newVal) => {
    start_date_error.value = false;
})

watch(end_date, (newVal) => {
    end_date_error.value = false;
})

</script>