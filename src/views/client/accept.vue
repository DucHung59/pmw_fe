<template>
    <div class="header flex items-center justify-between px-4">
        <img src="https://placehold.co/600x400" alt="logo" width="80px">
        <div class="" v-if="!userStore.isLoggedIn">
            <Button class="mx-2 button">
                <RouterLink :to="{ path: '/login', query: { inviteToken: inviteToken } }">
                    Đăng nhập
                </RouterLink>
            </Button>
            <Button class="mx-2 button" variant="outlined">
                <RouterLink :to="{ path: '/signup', query: { inviteToken: inviteToken } }">
                    Đăng ký
                </RouterLink>
            </Button>
        </div>
        <div v-else-if="!userStore.isLoggedIn && inviteData.success == 'false'">
            <Button icon="pi pi-sign-out" variant="outlined" rounded @click="signout"/>
        </div>
        <div v-else>
            <Button class="mx-2 button" variant="outlined">
                <RouterLink :to="{ path: '/workspace/dashboard' }">
                    tới Dashboard
                </RouterLink>
            </Button>
        </div>
    </div>
    <div class="container mx-auto" v-if="!isAccepting">
        <div v-if="!userStore.isLoggedIn">
            <h4>Bạn chưa đăng nhập, vui lòng đăng nhập hoặc đăng ký để chấp nhận lời mời</h4>
            <Button class="mx-2 button">
                <RouterLink :to="{ path: '/login', query: { inviteToken: inviteToken } }">
                    Đăng nhập
                </RouterLink>
            </Button>
            <Button class="mx-2 button" variant="outlined">
                <RouterLink :to="{ path: '/signup', query: { inviteToken: inviteToken } }">
                    Đăng ký
                </RouterLink>
            </Button>
        </div>
        <div v-else-if="!userStore.isLoggedIn && inviteData.success == 'false'">
            <h4>Bạn đăng nhập không đúng thông tin được mời, vui lòng đăng xuất và đăng nhập lại</h4>
        </div>
        <div v-else>
            <h4>Chào mừng bạn tới workspace: {{ inviteData?.workspace.workspace_name }}</h4>
        </div>
    </div>
</template>
<script setup>
import api from '@/api/axios';
import { useUserStore } from '@/store/user';
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { Button } from 'primevue';


const route = useRoute();
const inviteToken = route.params.inviteToken;

const userStore = useUserStore();
const isAccepting = ref(false)

async function acceptInvite() {
    try {
        isAccepting.value = true;
        const res = await api.post(`workspace/invite/accept`, {
            token: inviteToken
        });
        const inviteData = res.data;
        userStore.workspace = inviteData.workspace;
        userStore.role = inviteData.role;
    } catch (error) {
        console.error('Không thể accept lời mời:', error);
    } finally {
        isAccepting.value = false;
    }
}

async function signout() {
    try {
        userStore.logout();
    } catch (error) {
        console.error('Logout failed:', error);
    }
}

onMounted(() => {
    acceptInvite();
})

</script>