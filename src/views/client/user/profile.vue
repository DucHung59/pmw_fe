<template>
    <div class="header flex items-center justify-between p-4">
        <div class="flex items-center gap-4">
            <div v-if="!workspace">
                <RouterLink to="/">
                    <img :src="'/logo_rikai.png'" alt="logo" width="80px">
                </RouterLink>
            </div>
            <div v-else>
                <RouterLink to="/workspace/dashboard">
                    <img :src="'/logo_rikai.png'" alt="logo" width="80px">
                </RouterLink>
            </div>
        </div>
        <Button icon="pi pi-sign-out" variant="outlined" rounded @click="signout"/>
    </div>
    <div style="margin-top: 60px;">
        <Tabs value="0">
            <div class="flex flex-col items-center justify-center my-4">
                <p class="text-[24px] font-semibold text-primary">Cài đặt cá nhân</p>
                <TabList>
                    <Tab value="0">Cá nhân</Tab>
                    <Tab value="1">Bảo mật</Tab>
                    <Tab value="2">Workspace</Tab>
                </TabList>
            </div>
            <div class="container">
                <TabPanels>
                    <TabPanel value="0">
                        <div class="mt-4">
                            <div class="flex flex-col gap-2 mt-4">
                                <label for="username">Tên người dùng</label>
                                <InputText id="username" v-model="username" autocomplete="off" :disabled="userStore.isSystemAdmin"/>
                                <Message size="small" severity="secondary" variant="simple" class="px-4">
                                    <p>
                                        Tên hiển thị trong trong xuyên suốt quá trình triển khai dự án, đề xuất sử dụng 
                                        <span class="text-red-400 uppercase">Họ.Tên</span>
                                        của bạn
                                    </p>
                                </Message>
                            </div>
                            <div class="flex flex-col gap-2 mt-4">
                                <label for="email">Email</label>
                                <InputText id="email" v-model="email" autocomplete="off" disabled/>
                                <Message size="small" severity="secondary" variant="simple" class="px-4">
                                    <div class="flex items-center gap-2">
                                        <i class="pi" :class="verify ? 'pi-check' : 'pi-times'"></i>
                                        {{ verify ? 'Verified' : 'Unverified' }}
                                    </div>
                                </Message>
                            </div>
                            <template v-if="!userStore.isSystemAdmin">
                                <div class="button-feild flex justify-between mt-4">
                                    <Button icon="pi pi-trash" label="Xóa tài khoản" severity="danger" variant="text"/>
                                    <Button label="Chỉnh sửa" icon="pi pi-pencil" @click="changeAccountInfo"/>
                                </div>
                            </template>
                        </div>
                    </TabPanel>
                    <TabPanel value="1">
                        <template v-if="userStore.isSystemAdmin">
                            Bạn đang đăng nhập với tài khoản Admin, không thể chỉnh sửa thông tin này
                        </template>
                        <template v-else>
                            <div class="flex flex-col items-center justify-between">
                                <div class="w-lg">
                                    <Label class="text-lg font-medium">Độ phức tạp của mật khẩu</Label>
                                    <ProgressBar
                                        :value="user.password_complex*20" class="custom-progressbar w-full my-4">
                                        {{ user.password_complex }}
                                    </ProgressBar>
                                </div>
                                <div class="my-4">
                                    <Button label="Đổi mật khẩu" icon="pi pi-pencil" @click="openChangePasswordDialog = true" />
                                </div>
                            </div>
                            <Dialog v-model:visible="openChangePasswordDialog" :style="{width: '50vw'}" @hide="onDialogHide" header="Thay đổi mật khẩu" :draggable="false" :modal="true">
                                <div class="my-4">
                                    <label for="password">Mật khẩu hiện tại</label>
                                    <InputText id="password" v-model="currentPassword" class="w-full" type="password"/>
                                </div>
                                <div class="my-4">
                                    <label for="password">Mật khẩu mới</label>
                                    <InputText id="password" v-model="newPassword" class="w-full" type="password"/>
                                </div>
                                <div class="flex justify-around mt-4 pt-4">
                                    <Button label="Hủy" class="w-75" severity="secondary" size="small" variant="outlined" @click="openChangePasswordDialog = false"/>
                                    <Button label="Sửa" class="w-75" size="small" variant="outlined" @click="changePassword" :loading="isChangePassword" loadingIcon="pi pi-spin pi-spinner"/>
                                </div>
                            </Dialog>
                        </template>
                    </TabPanel>
                    <TabPanel value="2">
                        <template v-if="userStore.isSystemAdmin">
                            <span>Bạn đang đăng nhập với tài khoản admin, chọn workspace dưới đây để quản lý</span>
                            <div class="my-4 flex justify-between items-center">
                                <label for="" class="text-lg font-medium">Lựa chọn Workspace</label>
                                <RouterLink to="/create_workspace">
                                    <Button label="Tạo workspace" icon="pi pi-plus" rounded variant="outlined"/>
                                </RouterLink>
                            </div>
                            <div class="my-4 flex justify-between items-center">
                                <Select v-model="selectedWorkspace" :options="workspaces" optionLabel="workspace_name" optionValue="id" placeholder="Lựa chọn Workspace" class="w-lg" @change="onWorkspaceChange"/>
                                <RouterLink to="/workspace/dashboard">
                                    <Button label="Tới Dashboard" icon="pi pi-arrow-up-right" :loading="isChangeWorkspace" loadingIcon="pi pi-spin pi-spinner"/>
                                </RouterLink>
                            </div>
                        </template>
                        <template v-else>
                            <div v-if="workspace == null" class="m-4">
                                <div class="flex justify-around items-center gap-2">
                                    <p>
                                        Bạn chưa có Workspace, vui lòng liên hệ quản lý để thêm vào workspace
                                    </p>
                                </div>
                            </div>
                            <div v-else>
                                <p class="text-center text-2xl font-semibold">Thông tin workspace</p>
                                <div class="flex justify-around items-center gap-2 mt-4">
                                    <div class="text-lg">Workspace: <span class="font-semibold">{{ workspace.workspace_name }}</span></div>
                                    <RouterLink to="/workspace/dashboard">
                                        <Button label="Tới Dashboard" icon="pi pi-arrow-up-right"/>
                                    </RouterLink>
                                </div>
                            </div>
                        </template>
                    </TabPanel>
                </TabPanels>
            </div>
        </Tabs>
    </div>
</template>
<style>
.custom-progressbar .p-progressbar-value {
  transition: background-color 0.3s ease;
}
</style>
<script setup>
import api from '@/api/axios';
import { toastService } from '@/assets/js/toastHelper';
import { useUserStore } from '@/store/user';
import { Tabs, Tab, TabList, TabPanels, TabPanel, Button, InputText, Message, Select, ProgressBar, Dialog, FloatLabel, useToast } from 'primevue';
import { onMounted, ref, watch } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const userStore = useUserStore();

const toast = new toastService(useToast());
const workspace = userStore.workspace;
const workspaces = userStore.workspaces;


const user = userStore.user;
const username = user.username;
const email = user.email;
const verify = user.verify;

const selectedWorkspace = ref();
const openChangePasswordDialog = ref(false);
const isChangeWorkspace = ref(false);

const currentPassword = ref();
const newPassword = ref();
const isChangePassword = ref(false);

//UI function
const getColor = (level) => {
  switch (level) {
    case 1: return '#f87171'
    case 2: return '#fb7185'
    case 3: return '#fbbf24'
    case 4: return '#facc15'
    case 5: return '#4ade80'
    default: return '#d1d5db' // màu xám nhạt mặc định
  }
}


//Data function

async function onWorkspaceChange() {
    isChangeWorkspace.value = true;
    await userStore.getWorkspaceById(selectedWorkspace.value);
    isChangeWorkspace.value = false;
}

async function signout() {
    try {
        userStore.logout();
        router.push('/'); // Chuyển hướng về trang đăng nhập
    } catch (error) {
        console.error('Logout failed:', error);
    }
}

async function changeAccountInfo() {
    console.log(username.value);
}

async function changePassword() {
    try {
        isChangePassword.value = true;
        const response = await api.post('/auth/change-password', {
            current_password: currentPassword.value,
            new_password: newPassword.value,
        })

        const result = response.data;
        if (result.success) {
            toast.success(result.message, 'Thông báo');
            openChangePasswordDialog.value = false;
            userStore.fetchUser();
        } else {
            toast.warn(result.success, 'Thông báo');
        }
    } catch (error) {
        console.log(error.message);
    } finally {
        isChangePassword.value = false;
    }
}

onMounted(() =>{ 
    const el = document.querySelector('.custom-progressbar .p-progressbar-value')
    if (el) {
        el.style.backgroundColor = getColor(user.password_complex)
    }
})

watch(() => user.password_complex, (newVal) => {
  const el = document.querySelector('.custom-progressbar .p-progressbar-value')
  if (el) {
    el.style.backgroundColor = getColor(newVal)
  }
})

</script>