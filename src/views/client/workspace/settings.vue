<template>
    <div>
        <div class="settings-sidebar">
            <ul>
                <li class="sidebar-items">
                    <p class="text-2xl font-medium">{{ workspace.workspace_name }}</p>
                </li>
                <li class="sidebar-items">
                    <div class="flex items-center gap-2">
                        <i class="pi pi-cog"></i>
                        <p>Cài đặt Space</p>
                    </div>
                    <div class="detail-settings">
                        <ul>
                            <li :class="{ active: activeSetting === 'info' }"
                                @click="setActive('info')"
                            >Thông tin</li>
                        </ul>
                    </div>
                </li>
                <li class="sidebar-items">
                    <div class="flex items-center gap-2">
                        <i class="pi pi-users"></i>
                        <p>Cài đặt người dùng</p>
                    </div>
                    <div class="detail-settings">
                        <ul>
                            <li :class="{ active: activeSetting === 'member' }"
                                @click="setActive('member')"
                            >Thành viên</li>
                        </ul>
                        <ul>
                            <li :class="{ active: activeSetting === 'invite-member' }"
                                @click="setActive('invite-member')"
                            >Thành viên được mời</li>
                        </ul>
                    </div>
                </li>
                <li class="sidebar-items">
                    <div class="flex items-center gap-2">
                        <i class="pi pi-users"></i>
                        <p>Cài đặt dự án</p>
                    </div>
                    <div class="detail-settings">
                        <ul>
                            <li :class="{ active: activeSetting === 'project' }"
                                @click="setActive('project')"
                            >Danh sách</li>
                        </ul>
                    </div>
                </li>
            </ul>
        </div>
        <div class="content">
            <div v-if="!isLoading">
                <div class="info " v-if="activeSetting == 'info'">
                    <label class="font-medium">Tên workspace hiện tại: {{ workspace.workspace_name }}</label>
                    <div class="flex gap-2 items-center my-4">
                        <label for="">Chỉnh sửa tên workspace</label>
                        <InputText v-model="workspace_name"/>
                    </div>
                </div>
                <div class="member" v-if="activeSetting == 'member'">
                    <p class="text-2xl font-semibold">Danh sách thành viên <span class="text-sm text-gray-500">({{ members.length }} Người)</span></p>
                    <div class="flex items-center gap-4 my-4">
                        <Button label="Thêm thành viên" icon="pi pi-plus" variant="outlined" rounded size="small" @click="openAddMemberDialog = true"/>
                    </div>
                    <Dialog v-model:visible="openAddMemberDialog" header="Thêm thành viên" maximizable modal :draggable="false" resizable="false" :style="{ width: '50vw' }">
                        <div>
                            <p class="text-1xl font-semibold">Thêm thành viên</p>
                            <div class="flex flex-col items-center gap-2">
                                <div class="flex flex-wrap gap-4">
                                    <label class="font-semibold">Vai trò:</label>
                                    <div class="flex items-center gap-2">
                                        <RadioButton v-model="role" inputId="manager" name="role" value="manager" />
                                        <label for="manager">Quản lý</label>
                                    </div>
                                    <div class="flex items-center gap-2">
                                        <RadioButton v-model="role" inputId="member" name="role" value="member" />
                                        <label for="member">Thành viên</label>
                                    </div>
                                </div>
                                <div class="flex flex-col gap-2">
                                    <p v-if="role == 'manager'" class="text-sm text-gray-600">Quản lý cài đặt Workspace: người dùng, thành viên, dự án</p>
                                    <p v-if="role == 'member'" class="text-sm text-gray-600">Tham gia vào Workspace chỉnh sửa, triển khai tiến độ dự án</p>
                                </div>
                                <div class="my-4">
                                    <InputText v-model="newMemberEmail" placeholder="Nhập email" class="w-2xl" :invalid="emailValid"/>
                                    <Message v-if="emailValid" severity="error" variant="simple" size="small" class="mt-2 ml-2">Email is not valid</Message>
                                </div>
                            </div>
                                <Button label="Thêm" icon="pi pi-plus" :loading="isInviteMemberLoading" loadingIcon="pi pi-spin pi-spinner" variant="outlined" rounded size="small" @click="inviteMember"/>
                        </div>
                    </Dialog>
                    <Dialog v-model:visible="openEditRoleDialog" header="Sửa quyền thành viên" maximizable modal :draggable="false" resizable="false" :style="{ width: '50vw' }">
                        <div>
                            <template v-if="selectedMember?.user?.username == 'admin'">
                                Admin không thể chỉnh sửa quyền bản thân!
                            </template>
                            <template v-else>
                                <p class="mb-2"><strong>{{ selectedMember?.user?.username }}</strong> ({{ selectedMember?.user?.email }})</p>
                                <div class="flex flex-col items-center gap-2">
                                    <div class="flex flex-wrap gap-4">
                                        <label class="font-semibold">Vai trò:</label>
                                        <div class="flex items-center gap-2">
                                            <RadioButton v-model="editedRole" inputId="manager" name="role" value="manager" />
                                            <label for="manager">Quản lý</label>
                                        </div>
                                        <div class="flex items-center gap-2">
                                            <RadioButton v-model="editedRole" inputId="member" name="role" value="member" />
                                            <label for="member">Thành viên</label>
                                        </div>
                                    </div>
                                    <div class="flex flex-col gap-2">
                                        <p v-if="editedRole == 'manager'" class="text-sm text-gray-600">Quản lý cài đặt Workspace: người dùng, thành viên, dự án</p>
                                        <p v-if="editedRole == 'member'" class="text-sm text-gray-600">Tham gia vào Workspace chỉnh sửa, triển khai tiến độ dự án</p>
                                    </div>
                                </div>
                                <Button
                                    label="Lưu"
                                    icon="pi pi-check"
                                    :loading="isUpdateRole"
                                    loadingIcon="pi pi-spin pi-spinner"
                                    @click="changeMemberRole"
                                    rounded
                                    size="small"
                                    class="mt-2"
                                />
                            </template>
                        </div>
                    </Dialog>
                    <div>
                        <table class="w-full text-sm tr-border">
                            <thead>
                                <tr>
                                    <th class="px-3 py-2 border">#</th>
                                    <th class="px-3 py-2 border">Tên</th>
                                    <th class="px-3 py-2 border">Email</th>
                                    <th class="px-3 py-2 border">Vai trò</th>
                                    <th class="px-3 py-2 border">Ngày tham gia</th>
                                </tr>
                            </thead>
                            <tbody class="text-center">
                                <template v-for="(member, index) in members" :key="member.id">
                                    <tr class="hover:bg-gray-100 cursor-default">
                                        <td class="px-3 py-2">{{ index + 1 }}</td>
                                        <td class="px-3 py-2">{{ member.user.username }}</td>
                                        <td class="px-3 py-2">{{ member.user.email }}</td>
                                        <td class="px-3 py-2">
                                            {{ member.role }}
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
                                    </tr>
                                </template>
                            </tbody>
                        </table>
                        <Paginator
                            :rows="perPage"
                            :totalRecords="total"
                            :first="(currentPage - 1) * perPage"
                            @page="(event) => onPageChange(event, 'member')"
                        />
                    </div>
                </div>
                <div class="invite-member" v-if="activeSetting == 'invite-member'">
                    <div class="flex items-center gap-4 my-4">
                        <p class="text-2xl font-semibold">Danh sách thành viên được mời <span class="text-sm text-gray-500">({{ inviteMembers.length }} Người)</span></p>
                    </div>
                    <div>
                        <table class="w-full text-sm tr-border">
                            <thead>
                                <tr>
                                    <th class="px-3 py-2 border">#</th>
                                    <th class="px-3 py-2 border">Email</th>
                                    <th class="px-3 py-2 border">Người mời</th>
                                    <th class="px-3 py-2 border">Vai trò (Role)</th>
                                    <th class="px-3 py-2 border">Trạng thái</th>
                                    <th class="px-3 py-2 border">Ngày mời</th>
                                </tr>
                            </thead>
                            <tbody class="text-center">
                                <template v-for="(invite, index) in inviteMembers" :key="invite.id">
                                    <tr class="hover:bg-gray-100 cursor-default">
                                        <td class="px-3 py-2">{{ index + 1 }}</td>
                                        <td class="px-3 py-2">{{ invite.email }}</td>
                                        <td class="px-3 py-2">{{ invite.inviter.username }}</td>
                                        <td class="px-3 py-2">{{ invite.role }}</td>
                                        <td class="px-3 py-2">
                                            <span v-if="invite.status == 'pending'" class="block px-2 py-1 rounded-md bg-yellow-500 text-white">
                                                Chờ xác nhận
                                            </span>
                                            <span v-if="invite.status == 'accepted'" class="block px-2 py-1 rounded-md bg-green-500 text-white">
                                                Đã chấp nhận
                                            </span>
                                            <span v-if="invite.status == 'rejected'" class="block px-2 py-1 rounded-md bg-red-500 text-white">
                                                Đã từ chối
                                            </span>
                                        </td>
                                        <td class="px-3 py-2">{{ dayjs(invite.created_at).format('DD/MM/YYYY HH:mm') }}</td>
                                    </tr>
                                </template>
                            </tbody>
                        </table>
                        <Paginator
                            :rows="perPage"
                            :totalRecords="total"
                            :first="(currentPage - 1) * perPage"
                            @page="(event) => onPageChange(event, 'invite-member')"
                        />
                    </div>
                </div>
                <div class="project" v-if="activeSetting == 'project'">
                    <div>
                        <p class="text-2xl font-semibold">Danh sách dự án <span class="text-sm text-gray-500">( {{ projects.length }} )</span></p>
                    </div>
                    <div>
                        <table class="w-full text-sm tr-border">
                            <thead>
                                <tr>
                                    <th class="px-3 py-2 border">#</th>
                                    <th class="px-3 py-2 border">Tên dự án</th>
                                    <th class="px-3 py-2 border">Cài đặt</th>
                                </tr>
                            </thead>
                            <tbody class="text-center">
                                <template v-for="(project, index) in projects" :key="project.id">
                                    <tr class="hover:bg-gray-100 cursor-default">
                                        <td class="px-3 py-2">{{ index + 1 }}</td>
                                        <td class="px-3 py-2">
                                            {{ project.project_name }} 
                                            <span class="text-sm">({{ project.project_key }})</span>
                                        </td>
                                        <td class="px-3 py-2 flex gap-2 justify-center">
                                            <template v-if="userStore.isSystemAdmin">
                                                <RouterLink :to="'project/' + project.project_key">
                                                    <Button label="Tới" icon="pi pi-arrow-up-right" iconPos="right" size="small"/>
                                                </RouterLink>
                                            </template>
                                            <Button label="Xóa" icon="pi pi-trash" size="small" iconPos="right" severity="danger" v-if="userStore.user.id = project.created_by"/>
                                        </td>
                                    </tr>
                                </template>
                            </tbody>
                        </table>
                        <Paginator
                            :rows="perPage"
                            :totalRecords="total"
                            :first="(currentPage - 1) * perPage"
                            @page="(event) => onPageChange(event, 'invite-member')"
                        />
                    </div>
                </div>
            </div>
            <div v-else>
                <div class="loading">
                    <i class="pi pi-spin pi-spinner" style="font-size: 2rem;"></i>
                </div>
            </div>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import { ref, watch } from 'vue';
import { useUserStore } from '@/store/user';
import { Button, Dialog, InputText, Message, RadioButton, useToast, Paginator } from 'primevue';
import dayjs from 'dayjs';
import { toastService } from '@/assets/js/toastHelper';

//Store
const userStore = useUserStore();
const workspace = ref(userStore.workspace);
const workspace_name = ref();

//Handle UI
const activeSetting = ref('info');
const isLoading = ref(false);
const toast = new toastService(useToast());
const isInviteMemberLoading = ref(false);
const openAddMemberDialog = ref(false);
const emailValid = ref(false);

const currentPage = ref(1);
const total = ref(0);
const perPage = 10;

//Data
const members = ref();
const newMemberEmail = ref('');
const role = ref('');
const projects = ref([]);

const inviteMembers = ref([]);

//Function Handle UI
const selectedMember = ref(null);
const openEditRoleDialog = ref(false);
const editedRole = ref('');
const isUpdateRole = ref(false)

function editMemberRole(member) {
  selectedMember.value = member;
  editedRole.value = member.role;
  openEditRoleDialog.value = true;
}


function setActive(setting) {
    if (activeSetting.value === setting) return
    activeSetting.value = setting
    switch (setting) {
        case 'info':
            break;
        case 'email':
            break;
        case 'member':
            getMembers();
            break;
        case 'invite-member':
            getInviteMembers();
            break;
        case 'project':
            getProjects();
            break;
        default:
            break;
    }
}

function onPageChange(event, setting){
    const page = event.page + 1;
    switch (setting) {
        case 'space':
            //call api
            break;
        case 'info':
            break;
        case 'email':
            break;
        case 'member':
            getMembers(page);
            break;
        case 'invite-member':
            getInviteMembers(page);
            break;
        case 'project':
            getProjects(page);
            break;
        default:
            break;
    }
}

function validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}

//Function handle api
async function getMembers(page = 1) {
    try {
        isLoading.value = true;
        const response = await api.get('/workspace/members', {
            params: {
                workspaceId: workspace.value.id,
                page: page
            }
        })

        const data = response.data;
        members.value = data.members.data;
        total.value = data.members.total;
        currentPage.value = data.members.current_page;
    } catch (error) {
        console.log(error.message);
                
    } finally {
        isLoading.value = false;
    }
}

async function getInviteMembers(page = 1) {
    try {
        isLoading.value = true;
        const response = await api.get('/workspace/invites/get', {
            params: {
                workspace_id: workspace.value.id,
                page: page
            }
        })

        const data = response.data;
        inviteMembers.value = data.invites.data;
        total.value = data.invites.total;
        currentPage.value = data.invites.current_page;
    } catch (error) {
        console.log(error.message);
    } finally {
        isLoading.value = false;
    }
}

async function inviteMember() {
    if (!validateEmail(newMemberEmail.value)) {
        emailValid.value = true;
        return;
    }
    try {
        isInviteMemberLoading.value = true;
        const response = await api.post('/workspace/invite', {
            workspaceId: workspace.value.id,
            email: newMemberEmail.value,
            role: role.value
        });

        const data = response.data;
        toast.success('Thành viên đã được mời', 'Thành công');
        openAddMemberDialog.value = false;
        newMemberEmail.value = '';
        role.value = '';
    } catch (error) {
        console.log(error.message);
    } finally {
        isInviteMemberLoading.value = false;
    }
}

async function getProjects() {
    try {
        isLoading.value = true;
        const response = await api.get('/project/getAllProjects')

        const result = response.data;
        projects.value = result.projects.data;
        total.value = result.projects.total;
        currentPage.value = result.projects.current_page;
    } catch (error) {
        console.log(error.message);
    } finally {
        isLoading.value = false;
        openEditRoleDialog.value = false;
    }
}

async function changeMemberRole(){
    try {
        isUpdateRole.value = true
        const response = await api.post('/workspace/change-member-role', {
            user_id: selectedMember.value.user_id,
            workspace_id: workspace.value.id,
            new_role: editedRole.value,
        })
        const result = response.data;
        if(result.success) {
            toast.success('Chỉnh sửa thành công', 'Thông báo');
            getMembers();
            openEditRoleDialog.value = false;
        } else {
            toast.error(result.message, 'Lỗi')
        }
    } catch (error) {
        console.log(error.message);
    } finally {
        isUpdateRole.value = false
    }
}

watch(newMemberEmail, (newVal) => {
    emailValid.value = false;
})

</script>