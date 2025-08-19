<template>
    <Sidebar :active="'home'" :project_key="project.project_key" />
    <div style="margin-left: 60px;">
        <div class="project-header flex items-center justify-between">
            <div class="flex items-center gap-2" v-if="!project.project_name">
                <Skeleton class="mb-2"></Skeleton>
            </div>
            <div class="project-name flex items-center gap-2" v-else>
                <p>{{ project.project_name }}</p>
                <p class="text-[14px]">({{ project.project_key }})</p>
            </div>
            <div class="project-member flex items-center gap-2">
                <div class="member-list flex gap-0.5">
                    <template v-for="(member, index) in project.members" :key="member.id">
                        <template v-if="index < 3">
                            <div class="member-item">
                                <button
                                    class="w-8 h-8 rounded-full text-white flex items-center justify-center text-sm font-semibold uppercase"
                                    :style="{ backgroundColor: stringToColor(member.user.username) }"
                                    v-tooltip.bottom="member.user.username"
                                >
                                    {{ getInitial(member.user.username) }}
                                </button>
                            </div>
                        </template>
                    </template>
                    <div>
                        <Button
                            class="w-8 h-8 rounded-full text-white flex items-center justify-center text-sm font-semibold uppercase"
                            rounded
                            size="small"
                            icon="pi pi-ellipsis-h"
                            v-tooltip.bottom="'Tất cả'"
                        />
                    </div>
                </div>
                <template v-if="userStore.projectRole == 'PManager' || userStore.isSystemAdmin">
                    <Button label="Thêm thành viên" variant="outlined" size="small" class="button" @click="openAddMemberDialog"/>
                    <Dialog v-model:visible="addMemberDialog" modal header="Thêm thành viên" :draggable="false" :style="{ width: '25rem' }">
                        <div v-if="isLoadingWorkspaceMember">
                            <Skeleton width="22rem" class="mb-2"></Skeleton>
                        </div>
                        <div v-else>
                            <div class="flex gap-2">
                                <IconField>
                                    <InputIcon class="pi pi-search"/>
                                    <InputText v-model="searchMember" placeholder="Search" :style="{ width: '19rem' }"/>
                                </IconField>
                                <Button icon="pi pi-eraser" @click="searchMember = ''"/>
                            </div>
                            <template v-for="(member, index) in workspaceMember">
                                <div class="flex items-center m-4 justify-between">
                                    <button
                                        class="w-10 h-10 rounded-full text-white flex items-center justify-center text-sm font-semibold uppercase"
                                        :style="{ backgroundColor: stringToColor(member.user.username) }"
                                        v-tooltip.bottom="member.user.username"
                                    >
                                        {{ getInitial(member.user.username) }}
                                    </button>
                                    <div class="flex items-center gap-2">
                                        {{ member.user.username }}
                                        <span class="text-sm">({{ member.role }})</span>
                                    </div>
                                    <template v-if="member.user.username !== 'admin'">
                                        <Button icon="pi pi-plus" rounded size="small" v-if="!isMemberInProject(member.user.id)" @click="openAddMemberRoleDialog(member)"/>
                                        <Button v-else icon="pi pi-check" rounded size="small" disabled/>
                                    </template>
                                    <template v-else>
                                        <Button icon="pi pi-check" rounded size="small" disabled/>
                                    </template>
                                </div>
                            </template>
                            <Dialog v-model:visible="addMemberRoleDialog" :style="{width: '40vw'}" @hide="onDialogHide" header="Thêm Dự án mới" :draggable="false" maximizable="true" :modal="true">
                                <p class="mb-2"><strong>{{ selectedMember?.user?.username }}</strong> ({{ selectedMember?.user?.email }})</p>
                                <div class="flex flex-col items-center gap-2">
                                    <div class="flex flex-wrap gap-4">
                                        <label class="font-semibold">Vai trò:</label>
                                        <div class="flex items-center gap-2">
                                            <RadioButton v-model="role" inputId="manager" name="role" value="PManager" />
                                            <label for="manager">Quản lý</label>
                                        </div>
                                        <div class="flex items-center gap-2">
                                            <RadioButton v-model="role" inputId="member" name="role" value="PMember" />
                                            <label for="member">Thành viên</label>
                                        </div>
                                    </div>
                                    <div class="flex flex-col gap-2">
                                        <p v-if="role == 'PManager'" class="text-sm text-gray-600">Quản lý dự án</p>
                                        <p v-if="role == 'PMember'" class="text-sm text-gray-600">Triển khai dự án</p>
                                    </div>
                                </div>
                                <Button
                                    label="Thêm"
                                    icon="pi pi-check"
                                    @click="addProjectMember"
                                    rounded
                                    size="small"
                                    class="mt-2"
                                />
                            </Dialog>
                            <Paginator
                                :rows="perPage"
                                :totalRecords="total"
                                :first="(currentPage - 1) * perPage"
                                @page="(event) => onPageChange(event)"
                            />
                        </div>
                    </Dialog>
                </template>
            </div>
        </div>
        <div class="project-content m-8 flex gap-8">
            <div class="project-main w-full border-2 border-gray-200 rounded-md p-4">
                <p class="mb-4 text-lg font-medium">Nhật ký gần đây</p>
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
            <div class="project-sidebar w-1/2 border-2 border-gray-200 rounded-md p-4 h-fit">
                <div class="project-sidebar-item">
                    <div class="flex justify-between">
                        <p>Bảng trạng thái</p>
                        <p>Tổng số công việc: {{ totalTask }}</p>
                    </div>
                    <template v-if="!isLoadingReports">
                        <div class="my-4" >
                            <MeterGroup :value="tasks" labelPosition="end" labelOrientation="vertical"/>
                        </div>
                    </template>
                    <div class="my-4" v-else>
                        <Skeleton class="mb-2"></Skeleton>
                    </div>
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
import dayjs from 'dayjs';
import { Button, Tooltip, Skeleton, Dialog, InputText, IconField, InputIcon, useToast, Paginator, RadioButton, MeterGroup } from 'primevue';
import { onMounted, ref, computed, watch } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const project_key = computed(() => route.params.project_key);

const toast = new toastService(useToast());
const userStore = useUserStore()

const workspace = userStore.workspace;

const project = ref({});
const workspaceMember = ref([]);
const selectedMember = ref(null);

const isLoadingWorkspaceMember = ref(false);
const isLogLoading = ref(false);
const addMemberRoleDialog = ref(false);
const role = ref('');
const userId = ref();
const totalTask = ref(0);
const closedTask = ref(0);

const searchMember = ref('');
const addMemberDialog = ref(false);
const total = ref();
const currentPage = ref(1);
const perPage = 10;

const logs = ref({});
const logsTotal = ref();
const logsPerPage = ref(15);

const isLoadingReports = ref(false);
const tasks = ref([{}])
//UI
function onPageChange(event) {
    const page = event.page + 1;
    getWorkspaceMembers(page);
}

function viewMoreLogs() {
    const perPage = logsPerPage.value + 10;
    getProjectLogs(perPage);
}

function openAddMemberRoleDialog(member) {
    addMemberRoleDialog.value = true;
    selectedMember.value = member;
    role.value = member.role == 'manager' ? 'PManager' : 'PMember';
    userId.value = member.user_id;
}

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

function isMemberInProject(userId) {
    return project.value.members.some(pm => pm.user_id === userId);
}

function openAddMemberDialog() {
    getWorkspaceMembers();
    addMemberDialog.value = true;
}
 
//api
async function getProject() {
    try {
        const response = await api.get('project/detail', {
            params: {
                project_key: project_key.value
            }
        })

        project.value = response.data.project;
        await userStore.setProjectContext(project.value.id);
    } catch (error) {
        console.log('Có lỗi xảy ra: ' + error.message);
    }
}

async function getReportsData() {
    try {
        isLoadingReports.value = true;
        const response = await api.get('/project/get/reports', {
            params: {
                project_id: project.value.id
            }
        })

        const result = response.data;
        totalTask.value = result.total;
        closedTask.value = result.data.find(item => item.label === 'Closed').count;
        tasks.value = result.data.map(item => ({
            label: item.label,
            color: item.color,
            value: item.count ? (item.count/totalTask.value)*100 : 0,
        }));
        console.log(tasks.value);
    } catch (error) {
        console.log('Có lỗi xảy ra: ' + error.message);
    } finally {
        isLoadingReports.value = false;
    }
}

async function getWorkspaceMembers(page = 1) {
    try {
        isLoadingWorkspaceMember.value = true;
        const response = await api.get('/workspace/members', {
            params: {
                workspaceId: workspace.id,
                page: page
            }
        })

        const data = response.data;
        workspaceMember.value = data.members.data;
        total.value = data.members.total;
        currentPage.value = data.members.current_page;
    } catch (error) {
        console.log(error.message);
    } finally {
        isLoadingWorkspaceMember.value = false;
    }
}

async function getProjectLogs(perPage = 15) {
    try {
        isLogLoading.value = true;
        const response = await api.get('log/getByProject', {
                params: {
                    project_id: project.value.id,
                    perPage: perPage,
                }
            }
        )

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

async function addProjectMember() {
    try {
        console.log(userId.value);
        
        const response = await api.post('/project/addMember', {
            project_key: project_key.value,
            user_id : userId.value,
            role : role.value,
        })

        const data = response.data;
        if (data.success) {
            toast.success('Thêm thành viên thành công', 'Thành công');
            addMemberRoleDialog.value = false;
            getWorkspaceMembers();
            getProject();
            getProjectLogs();
        } else {
            toast.warn('Thêm thành viên không thành công', 'Lỗi');
        }
    } catch (error) {
        console.log(error.message);
    }
}

onMounted(async () => {
    await getProject();
    getProjectLogs();
    getReportsData();
})

watch(project_key, async () => {
    await getProject();
    getProjectLogs();
    getReportsData();
});

</script>