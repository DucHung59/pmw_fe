<template>
    <Sidebar :active="'files'" :project_key="project.project_key" />
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
        <div class="m-8">
            <div class="flex justify-between items-center">
                <p class="font-medium text-lg">Tài liệu</p>
                <Button icon="pi pi-plus" rounded v-tooltip.left="'Thêm tài liệu'" @click="addDocumentDialog = true" v-if="userStore.isSystemAdmin || userStore.isProjectManager"/>
                <Dialog v-model:visible="addDocumentDialog" header="Thêm tài liệu mới" :style="{width: '50vw'}" :draggable="false" maximizable="true" :modal="true">
                    <div class="p-4 flex gap-2 items-center">
                        <label>Chỉ quản lý: </label>
                        <ToggleButton v-model="isManagerView" class="w-24" onLabel="Mở" offLabel="Đóng" size="small"/>
                    </div>
                    <FileUpload name="document[]" :customUpload="true" @uploader="onCustomUpload" :multiple="false" accept=".pdf,.docx,.xlsx,.png,.jpg" :maxFileSize="104857600" @select="onSelectedFiles">
                        <template #header="{ chooseCallback, clearCallback, files }">
                            <div class="flex flex-wrap justify-between items-center flex-1 gap-4">
                                <div class="flex gap-2">
                                    <Button @click="handleChooseFile(chooseCallback, files)" icon="pi pi-file" rounded outlined severity="secondary"></Button>
                                    <Button @click="onCustomUpload()" icon="pi pi-cloud-upload" rounded outlined severity="success" :disabled="!files || files.length === 0"></Button>
                                    <Button @click="clearCallback()" icon="pi pi-times" rounded outlined severity="danger" :disabled="!files || files.length === 0"></Button>
                                </div>
                            </div>
                        </template>
                        <template #content="{ files, removeFileCallback }">
                            <div v-if="files.length > 0" class="flex flex-wrap justify-center gap-4 p-4">
                                <div v-for="(file, index) in files" :key="file.name + file.size" class="w-40 flex flex-col items-center border rounded p-2">
                                    <div class="mb-2">
                                        <img
                                            v-if="isImage(file)"
                                            :src="file.objectURL"
                                            :alt="file.name"
                                            class="w-16 h-16 object-contain"
                                        />
                                        <img
                                            v-else-if="isPdf(file)"
                                            src="https://img.icons8.com/color/48/000000/pdf.png"
                                            alt="pdf"
                                            class="w-16 h-16"
                                        />
                                        <img
                                            v-else-if="isDocx(file)"
                                            src="https://img.icons8.com/color/48/000000/microsoft-word-2019.png"
                                            alt="word"
                                            class="w-16 h-16"
                                        />
                                        
                                        <img
                                            v-else-if="isExcel(file)"
                                            src="https://img.icons8.com/color/48/000000/microsoft-excel-2019.png"
                                            alt="word"
                                            class="w-16 h-16"
                                        />
                                    </div>
                                    <div class="text-xs text-center truncate max-w-[100px]">{{ file.name }}</div>
                                    <Button icon="pi pi-times" size="small" rounded outlined severity="danger" @click="removeFileCallback(index)" class="mt-1" />
                                </div>
                            </div>
                        </template>
                        <template #empty>
                            <div class="flex items-center justify-center flex-col">
                                <i class="pi pi-cloud-upload !border-2 !rounded-full !p-8 !text-4xl !text-muted-color" />
                                <p class="mt-6 mb-0">Kéo và thả tệp vào đây để upload.</p>
                            </div>
                        </template>
                    </FileUpload>
                </Dialog>
            </div>
            <div class="my-4">
                <p class="font-medium">Chi tiết các tài liệu của dự án</p>
            </div>
            <div>
                <Dialog v-model:visible="confirmDialog" header="Bạn có chắc chắn không?" :style="{ width: '40vw' }" :draggable="false" :modal="true">
                    <p class="text-center">Bạn có chắc chắn xóa {{ delFile.title }}</p>
                    <div class="flex justify-center items-center my-8 gap-2">
                        <Button icon="pi pi-times" label="Không" @click="confirmDialog = false"/>
                        <Button icon="pi pi-trash" label="Chắc chắn" severity="danger" @click="delDocument"/>
                    </div>
                </Dialog>

                <table class="w-full text-sm tr-border mb-4">
                    <thead>
                        <tr>
                            <th class="px-3 py-2 border">#</th>
                            <th class="px-3 py-2 border">Tên file</th>
                            <th class="px-3 py-2 border">Ngày tạo</th>
                            <th class="px-3 py-2 border">Chi tiết</th>
                            <th class="px-3 py-2 border">Cài đặt</th>
                        </tr>
                    </thead>
                    <tbody class="text-center">
                        <template v-for="(file, index) in filteredFiles">
                            <tr class="hover:bg-gray-100 cursor-default">
                                <td class="px-3 py-2">{{ index + 1 }}</td>
                                <td class="px-3 py-2">{{ file.title }}</td>
                                <td class="px-3 py-2">{{ dayjs(file.created_at).format('DD/MM/YYYY HH:mm') }}</td>
                                <td class="px-3 py-2"><a :href="`http://localhost:8000${file.full_url}`" target="_blank">Xem tài liệu</a></td>
                                <template v-if="userStore.isSystemAdmin || userStore.isProjectManager">
                                    <td class="px-3 py-2">
                                        <Button icon="pi pi-trash" variant="text" severity="danger" rounded @click="openConfirmDialog(file)"/>
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
                    :totalRecords="total"
                    :first="(currentPage - 1) * perPage"
                    @page="(event) => onMemberPageChange(event, 'member')"
                /> 
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
import { Button, Dialog, FileUpload, usePrimeVue, useToast, ToggleButton, Paginator } from 'primevue';
import { computed, onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';


const $primevue = usePrimeVue();
const route = useRoute();
const router = useRouter();
const userStore = useUserStore();

const project_key = computed(() => route.params.project_key);
const project = ref({});

const toast = new toastService(useToast());
const isManagerView = ref(false);

const confirmDialog = ref(false);
const addDocumentDialog = ref(false);
const totalSize = ref(0);
const files = ref([]);
const listFiles = ref([]);
const delFile = ref();
const total = ref(0);
const currentPage = ref(1);
const perPage = 15;

const isImage = (file) => file.type.startsWith('image/');
const isPdf = (file) => file.type === 'application/pdf';
const isDocx = (file) => file.name.endsWith('.doc') || file.name.endsWith('.docx');
const isExcel = (file) => file.name.endsWith('.xls') || file.name.endsWith('.xlsx');

const filteredFiles = computed(() => {
  return Array.isArray(listFiles.value)
    ? listFiles.value.filter(file => file.manager_view != 1 || userStore.isProjectManager || userStore.isSystemAdmin)
    : [];
});


const openConfirmDialog = (file) => {
    delFile.value = file;
    console.log(delFile.value);
    confirmDialog.value = true;
}
const handleChooseFile = (chooseCallback, files) => {
    if (files && files.length >= 1) {
        toast.warn('Bạn chỉ được chọn duy nhất 1 file', 'Thông báo');
        return;
    }
    chooseCallback();
};

const onSelectedFiles = (event) => {
    files.value = event.files;
    files.value.forEach((file) => {
        totalSize.value += parseInt(formatSize(file.size));
    });
};

const formatSize = (bytes) => {
    const k = 1024;
    const dm = 3;
    const sizes = $primevue.config.locale.fileSizeTypes;

    if (bytes === 0) {
        return `0 ${sizes[0]}`;
    }

    const i = Math.floor(Math.log(bytes) / Math.log(k));
    const formattedSize = parseFloat((bytes / Math.pow(k, i)).toFixed(dm));

    return `${formattedSize} ${sizes[i]}`;
};


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


async function onCustomUpload() {
    if (!files.value || files.value.length === 0) {
        toast.error('Vui lòng chọn một tệp để tải lên.', 'Lỗi');
        return;
    }
    
    const manager_view = isManagerView.value ? 1 : 0

    const file = files.value[0];
    const formData = new FormData();
    formData.append('document', file);
    formData.append('project_id', project.value.id);
    formData.append('manager_view', manager_view);

    try {
        const response = await api.post('project/document/upload', formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        })
        
        const result = response.data;
        if (result.success) {
            toast.success(result.message, 'Thành công')
            files.value = [];
            addDocumentDialog.value = false;
            getDocument();
        } else {
            toast.warn(result.message, 'Thất bại')
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function getDocument() {
    try {
        const response = await api.get('/project/documents/get', {
            params: {
                project_id: project.value.id,
            }
        })

        const result = response.data;
        if (result.success) {
            listFiles.value = result.documents.data;
            total.value = result.documents.total;
            currentPage.value = result.documents.current_page;
        }
    } catch (error) {
        console.log(error.message);
    }
}

async function delDocument() {
    try {
        const response = await api.post('/project/document/delete', {
            file_id: delFile.value.id,
        })
        
        const result = response.data;
        if (result.success) {
            toast.success(result.message, 'Thông báo');
            confirmDialog.value = false;
            getDocument();
        } else {
            toast.error(result.message, 'Lỗi');
        }
    } catch (error) {
        console.log(error.message);
    }
}

onMounted(async () => {
    await getProject();
    getDocument();
})
</script>