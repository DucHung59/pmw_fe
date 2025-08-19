<template>
    <Sidebar :active="'settings'" :project_key="project.project_key" />
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
        <div class="mt-4 mx-6">
            <Tabs value="0">
                <TabList>
                    <Tab value="0">Cài đặt chung</Tab>
                    <Tab value="1">Thành viên dự án</Tab>
                    <Tab value="2">Danh mục</Tab>
                    <Tab value="3">Trạng thái</Tab>
                </TabList>
                <TabPanels>
                    <TabPanel value="0">
                        <div class="p-4">
                            <p class="font-medium text-lg">{{ project.project_name }} <span class="text-sm">({{ project.project_key }})</span></p>
                            <span v-if="project.start_date && project.end_date">Từ {{ dayjs(project.start_date).format('DD/MM/YYYY') }} tới {{ dayjs(project.end_date).format('DD/MM/YYYY') }}</span>
                        </div>
                        <div class="px-4">
                            <div class="flex justify-between">
                                <p class="font-medium">Mô tả</p>
                                <Button icon="pi pi-pen-to-square" v-tooltip:left="'Chỉnh sửa mô tả dự án'"/>
                            </div>
                            <div class="px-8">
                                <span v-html="project.description"></span>
                            </div>
                        </div>
                    </TabPanel>
                    <TabPanel value="1">
                        <div class="m-4">
                            <p class="text-lg font-medium">Thành viên dự án</p>
                        </div>
                        <Dialog v-model:visible="confirmDialog" header="Bạn có chắc chắn không?" :style="{ width: '40vw' }" :draggable="false" :modal="true">
                            <p class="text-center">Bạn có chắc chắn xóa {{ selectedMember.user.username }} khỏi dự án</p>
                            <div class="flex justify-center items-center my-8 gap-2">
                                <Button icon="pi pi-times" label="Không" @click="confirmDialog = false"/>
                                <Button icon="pi pi-trash" label="Chắc chắn" severity="danger" @click="delMember"/>
                            </div>
                        </Dialog>
                        <div class="m-4">
                            <table class="w-full text-sm tr-border mb-4">
                                <thead>
                                    <tr>
                                        <th class="px-3 py-2 border">#</th>
                                        <th class="px-3 py-2 border">Tên</th>
                                        <th class="px-3 py-2 border">Email</th>
                                        <th class="px-3 py-2 border">Vai trò</th>
                                        <th class="px-3 py-2 border">Ngày tham gia</th>
                                        <th class="px-3 py-2 border">Cài đặt</th>
                                    </tr>
                                </thead>
                                <tbody class="text-center">
                                    <template v-for="(member, index) in members" :key="member.id">
                                        <tr class="hover:bg-gray-100 cursor-default">
                                            <td class="px-3 py-2">{{ index + 1 }}</td>
                                            <td class="px-3 py-2">{{ member.user.username }}</td>
                                            <td class="px-3 py-2">{{ member.user.email }}</td>
                                            <td class="px-3 py-2">
                                                {{ member.project_role == 'PManager' ? 'Quản lý dự án' : 'Thành viên dự án'}}
                                                <Button
                                                    v-if="userStore.isSystemAdmin"
                                                    icon="pi pi-pencil"
                                                    rounded
                                                    text
                                                    size="small"
                                                    @click="editMemberRole(member)"
                                                    class="text-blue-500"
                                                />
                                            </td>
                                            <td class="px-3 py-2">{{ dayjs(member.created_at).format('DD/MM/YYYY HH:mm') }}</td>
                                            <template v-if="(userStore.isSystemAdmin || (!userStore.isSystemAdmin && userStore.isProjectManager && member.project_role !== 'PManager')) && userStore.user.id != member.user_id">
                                                <td class="px-3 py-2">
                                                    <Button icon="pi pi-trash" size="small" severity="danger" @click="openConfirmDialog(member)"/>
                                                </td>
                                            </template>
                                            <template v-else>
                                                <td class="px-3 py-2">
                                                    <Button icon="pi pi-ban" size="small" disabled severity="warn"/>
                                                </td>
                                            </template>
                                        </tr>
                                    </template>
                                </tbody>
                            </table>
                            <Paginator
                                :rows="perPage"
                                :totalRecords="totalMember"
                                :first="(currentPageMember - 1) * perPage"
                                @page="(event) => onMemberPageChange(event, 'member')"
                            />    
                        </div>
                    </TabPanel>
                    <TabPanel value="2">
                        <div class="m-4 flex items-center justify-between">
                            <p class="text-lg font-medium">Danh mục</p>
                            <template v-if="userStore.isSystemAdmin || userStore.projectRole == 'PManager'">
                                <Button label="Thêm" icon="pi pi-plus" rounded variant="outlined" @click="openCategoryDialog('add')"/>
                            </template>
                            <Dialog v-model:visible="openAddTaskCategory" :style="{width: '50vw'}" @hide="onDialogHide" header="Thêm danh mục mới" :draggable="false" :modal="true">
                                <div class="grid grid-cols-2 gap-2 m-4">
                                    <template v-for="(category, index) in allExistCategories">
                                        <div class="m-4">
                                            <Button icon="pi pi-plus" variant="outlined" rounded size="small" @click="addCategory(category.id)"/>
                                            <span class="px-4 py-1 mx-4 text-white font-medium rounded-full" 
                                                :style="{ backgroundColor: category.category_color }">
                                                {{ index + 1 }}: {{category.category_type}}
                                            </span>
                                        </div>
                                    </template>
                                </div>
                                <Button label="Đóng" icon="pi pi-times" @click="openAddTaskCategory = false"/>
                            </Dialog>
                        </div>
                        <div class="m-4">
                            <table class="w-full text-sm tr-border mb-4">
                                <thead>
                                    <tr>
                                        <th class="px-3 py-2 border">#</th>
                                        <th class="px-3 py-2 border">Tên</th>
                                        <th class="px-3 py-2 border">Màu hiển thị</th>
                                        <th class="px-3 py-2 border">Ngày tạo</th>
                                        <th class="px-3 py-2 border">Cài đặt</th>
                                    </tr>
                                </thead>
                                <tbody class="text-center">
                                    <template v-for="(issue, index) in issues" :key="issue.id">
                                        <tr class="hover:bg-gray-100 cursor-default">
                                            <td class="px-3 py-2">{{ index + 1 }}</td>
                                            <td class="px-3 py-2">{{ issue.category.category_type }}</td>
                                            <td class="px-3 py-2">
                                                <div class="flex items-center justify-center gap-2">
                                                    <div :style="{ backgroundColor: issue.category.category_color }" class="p-4 w-2 h-2 rounded-full"></div>
                                                    <span>
                                                        {{ issue.category.category_color }}
                                                    </span>
                                                </div>
                                            </td>
                                            <td class="px-3 py-2">{{ dayjs(issue.created_at).format('DD/MM/YYYY HH:mm') }}</td>
                                            <template v-if="userStore.isSystemAdmin || userStore.projectRole == 'PManager'">
                                                <td class="px-3 py-2 flex gap-2 items-center justify-center">
                                                    <Button icon="pi pi-trash" size="small" severity="danger" @click="deleteCategory(issue.id)"/>
                                                </td>
                                            </template>
                                            <template v-else>
                                                <td class="px-3 py-2">
                                                    <Button icon="pi pi-ban" size="small" disabled severity="warn"/>
                                                </td>
                                            </template>
                                        </tr>
                                    </template>
                                </tbody>
                            </table>
                            <Paginator
                                :rows="perPage"
                                :totalRecords="totalIssue"
                                :first="(currentPageIssue - 1) * perPage"
                                @page="(event) => onIssuePageChange(event, 'member')"
                            />    
                        </div>
                    </TabPanel>
                    <TabPanel value="3">
                        <div class="m-4 flex items-center justify-between">
                            <p class="text-lg font-medium">Trạng thái</p>
                            <template v-if="userStore.isSystemAdmin || userStore.projectRole == 'PManager'">
                                <Button label="Thêm" icon="pi pi-plus" rounded variant="outlined" @click="openStatusDialog('add')" />
                            </template>
                            <Dialog v-model:visible="openAddTaskStatus" :style="{width: '50vw'}" @hide="onDialogHide" :header="isEditStatus ? 'Chỉnh sửa trạng thái' : 'Tạo mới trạng thái'" :draggable="false" :modal="true">
                                <div class="grid grid-cols-2 gap-2 m-4">
                                    <template v-for="(status, index) in allExistStatuses">
                                        <div class="m-4">
                                            <Button icon="pi pi-plus" variant="outlined" rounded size="small" @click="addStatus(status.id)"/>
                                            <span class="px-4 py-1 mx-4 text-white font-medium rounded-full" 
                                                :style="{ backgroundColor: status.status_color }">
                                                {{ index + 1 }}: {{status.status_type}}
                                            </span>
                                        </div>
                                    </template>
                                </div>
                                <Button label="Đóng" icon="pi pi-times" @click="openAddTaskStatus = false"/>
                            </Dialog>
                        </div>
                        <div class="m-4">
                            <table class="w-full text-sm tr-border mb-4">
                                <thead>
                                    <tr>
                                        <th class="px-3 py-2 border">#</th>
                                        <th class="px-3 py-2 border">Tên</th>
                                        <th class="px-3 py-2 border">Màu hiển thị</th>
                                        <th class="px-3 py-2 border">Ngày tạo</th>
                                        <th class="px-3 py-2 border">Cài đặt</th>
                                    </tr>
                                </thead>
                                <tbody class="text-center">
                                    <template v-for="(status, index) in statuses" :key="status.id">
                                        <tr class="hover:bg-gray-100 cursor-default">
                                            <td class="px-3 py-2">{{ index + 1 }}</td>
                                            <td class="px-3 py-2">{{ status.status.status_type }}</td>
                                            <td class="px-3 py-2">
                                                <div class="flex items-center justify-center gap-2">
                                                    <div :style="{ backgroundColor: status.status.status_color }" class="p-4 w-2 h-2 rounded-full"></div>
                                                    <span>
                                                        {{ status.status.status_color }}
                                                    </span>
                                                </div>
                                            </td>
                                            <td class="px-3 py-2">{{ dayjs(status.created_at).format('DD/MM/YYYY HH:mm') }}</td>
                                            <template v-if="userStore.isSystemAdmin || userStore.projectRole == 'PManager'">
                                                <td class="px-3 py-2 flex gap-2 items-center justify-center">
                                                    <Button icon="pi pi-trash" size="small" severity="danger" @click="deleteStatus(status.id)"/>
                                                </td>
                                            </template>
                                            <template v-else>
                                                <td class="px-3 py-2">
                                                    <Button icon="pi pi-ban" size="small" disabled severity="warn"/>
                                                </td>
                                            </template>
                                        </tr>
                                    </template>
                                </tbody>
                            </table>
                            <Paginator
                                :rows="perPage"
                                :totalRecords="totalStatus"
                                :first="(currentPageStatus - 1) * perPage"
                                @page="(event) => onStatusPageChange(event, 'member')"
                            />    
                        </div>
                    </TabPanel>
                </TabPanels>
            </Tabs>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import Sidebar from '@/components/common/Sidebar.vue';
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { Tabs, TabList, Tab, TabPanels, TabPanel, Button, Paginator, Dialog, InputText, ColorPicker, useToast } from 'primevue';
import { useUserStore } from '@/store/user';
import dayjs from 'dayjs';
import { toastService } from '@/assets/js/toastHelper';

const route = useRoute();
const project_key = route.params.project_key;

const userStore = useUserStore();
const toast = new toastService(useToast());

const project = ref({});
const members = ref ({});
const issues = ref({});
const statuses = ref({});

const allExistCategories = ref({});
const allExistStatuses = ref({})

const totalMember = ref(0);
const currentPageMember = ref(1);
const totalIssue = ref(0);
const currentPageIssue = ref(1);
const totalStatus = ref(0);
const currentPageStatus = ref(1);
const perPage = 10;

const openAddTaskCategory = ref(false);
const openAddTaskStatus = ref(false);
const isEditCategory = ref(false);
const isEditStatus = ref(false)
const confirmDialog = ref(false);

const isCategoryLoading = ref(false);
const isStatusLoading = ref(false);

function openCategoryDialog(mode, data = null) {
    if(mode == 'add') {
        isEditCategory.value = false;
        category_id.value = null;
        category_type.value = '';
        category_color.value = '';
        openAddTaskCategory.value = true;
    } else {
        isEditCategory.value = true;
        category_id.value = data.id;
        category_type.value = data.category_type;
        category_color.value = data.category_color;
        openAddTaskCategory.value = true;
    }
}

function openStatusDialog(mode, data = null) {
    if(mode == 'add') {
        isEditStatus.value = false;
        status_id.value = null;
        status_type.value = '';
        status_color.value = '';
        openAddTaskStatus.value = true;
    } else {
        isEditStatus.value = true;
        status_id.value = data.id;
        status_type.value = data.status_type;
        status_color.value = data.status_color;
        openAddTaskStatus.value = true;
    }
}

function openConfirmDialog(member) {
    selectedMember.value = member;
    confirmDialog.value = true;
}

function onDialogHide() {
    category_id.value = '';

}

//Catergory data
const category_id = ref('');
const category_type = ref('');
const category_color = ref('');

//Status data
const status_id = ref('');
const status_type = ref('');
const status_color = ref('');

//member data
const selectedMember = ref({});

async function getProject() {
    try {
        const response = await api.get('project/detail', {
            params: {
                project_key: project_key
            }
        })

        project.value = response.data.project;
        userStore.setProjectContext(project.value.id);
    } catch (error) {
        console.log(error.message);
    }
}

async function getProjectMembers() {
    try {
        const response = await api.get('project/getProjectMembers', {
            params: {
                project_id: project.value.id,
            }
        })

        const result = response.data;
        if (result.success) {
            members.value = result.members.data;
            totalMember.value = result.members.total;
            currentPageMember.value = result.members.current_page;
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function getProjectIssues() {
    try {
        const response = await api.get('project/getProjectIssues', {
            params: {
                project_id: project.value.id,
            }
        })

        const result = response.data;
        if (result.success) {
            issues.value = result.issues.data;
            totalIssue.value = result.issues.total;
            currentPageIssue.value = result.issues.current_page;
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function getProjectStatuses() {
    try {
        const response = await api.get('project/getProjectStatuses', {
            params: {
                project_id: project.value.id,
            }
        })

        const result = response.data;
        if (result.success) {
            statuses.value = result.statuses.data;
            totalStatus.value = result.statuses.total;
            currentPageStatus.value = result.statuses.current_page;
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function addCategory(category_id) {
    try {
        isCategoryLoading.value = true;
        const response = await api.post('project/createProjectIssue', {
            project_id: project.value.id,
            category_id: category_id,
        })
        
        const result = response.data;
        if (result.success) {
            toast.success('Thêm mới thành công', 'Thông báo');
            getProjectIssues();
            getTaskCategory();
            openAddTaskCategory.value = false;
        } else {
            toast.error(result.message ,'Có lỗi xảy ra')
        }
    } catch (error) {
        console.log(error.message);
        isCategoryLoading.value = false;
    } finally {
        isCategoryLoading.value = false;
    }
}

async function getTaskCategory() {
    try {
        const response = await api.get('project/get/task-category', {
            params: {
                project_id: project.value.id
            }
        })
        
        const result = response.data;
        if (result.success) {
            allExistCategories.value = result.categories.data;
        } else {
            toast.error(result.message ,'Có lỗi xảy ra')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function addStatus(status_id) {
    try {
        isStatusLoading.value = true;
        const response = await api.post('project/createProjectStatus', {
            project_id: project.value.id,
            status_id: status_id,
        })
        
        const result = response.data;
        if (result.success) {
            toast.success('Thêm mới thành công', 'Thông báo');
            getProjectStatuses();
            getTaskStatus();
            openAddTaskStatus.value = false;
        } else {
            toast.error(result.message ,'Có lỗi xảy ra')
        }
    } catch (error) {
        console.log(error.message);
        isCategoryLoading.value = false;
    } finally {
        isCategoryLoading.value = false;
    }
}


async function getTaskStatus() {
    try {
        const response = await api.get('project/get/task-status', {
            params: {
                project_id: project.value.id
            }
        })
        
        const result = response.data;
        if (result.success) {
            allExistStatuses.value = result.statuses.data;
        } else {
            toast.error(result.message ,'Có lỗi xảy ra')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function delMember() {
    try {
        const response = await api.post('/project/member/delete', {
            project_id: project.value.id,
            user_id: selectedMember.value.user_id,
        })

        const result = response.data;
        if (result.success) {
            toast.success(result.message, 'Thông báo');
            confirmDialog.value = false;
            getProjectMembers();
        } else {
            toast.error(result.message, 'Lỗi')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function deleteCategory(category_id) {
    try {
        const response = await api.post('/project/delete/project-category', {
            category_id: category_id,
            project_id: project.value.id,
        })

        const result = response.data;
        
        if (result.success) {
            toast.success(result.message, 'Thông báo')
            getProjectIssues();
            getTaskCategory();
        } else {
            toast.warn(result.message, 'Lỗi')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function deleteStatus(status_id) {
    try {
        const response = await api.post('/project/delete/project-status', {
            status_id: status_id,
            project_id: project.value.id,
        })

        const result = response.data;
        
        if (result.success) {
            toast.success(result.message, 'Thông báo')
            getProjectStatuses();
            getTaskStatus();
        } else {
            toast.warn(result.message, 'Lỗi')
        }
    } catch (error) {
        console.log(error.message);
    }
}

onMounted(async () => {
    await getProject();
    getProjectMembers();
    getProjectIssues();
    getProjectStatuses();
    getTaskCategory();
    getTaskStatus();
})

</script>
