<template>
    <Sidebar :active="'reports'" :project_key="project.project_key" />
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
        <div class="my-8 mx-16">
            <div class="flex items-center justify-between gap-2">
                <div>
                    <p class="text-[24px] font-medium">Biểu đồ công việc</p>
                </div>
                <div>
                    <p>Tổng số task: {{ total }}</p>
                    <p>Tổng số task đã hoàn thành: {{ closedTask }} ({{ Math.round(closedTask / total * 100) }}%)</p>
                </div>
            </div>  
            <div class="m-4 p-4 border border-gray-300 rounded-md">
                <Chart type="bar" :data="chartData" :options="chartOptions" />
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import Sidebar from '@/components/common/Sidebar.vue';
import Chart from 'primevue/chart';
import { useRoute } from 'vue-router';
import { useUserStore } from '@/store/user';
import api from "@/api/axios";

const userStore = useUserStore();

const route = useRoute();
const project_key = computed(() => route.params.project_key);
const project = ref({});

const total = ref(0);
const closedTask = ref(0);
const chartData = ref({
    labels: [],
    datasets: [{
        label: 'Trạng thái',
        data: [],
        backgroundColor: []
    }]
});


onMounted(async () => {
    await getProject();
    getChartData();
    chartOptions.value = setChartOptions();
});

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

async function getChartData() {
    try {
        const response = await api.get('/project/get/reports', {
            params: {
                project_id: project.value.id
            }
        })

        const result = response.data;
        total.value = result.total;
        closedTask.value = result.data.find(item => item.label === 'Closed').count;
        chartData.value = {
            labels: ['Trạng thái'],
            datasets: result.data.map(item => ({
                label: item.label,
                data: [item.count],
                backgroundColor: item.color
            }))
        };
    } catch (error) {
        console.log('Có lỗi xảy ra: ' + error.message);
    }
}

const chartOptions = ref();

const setChartOptions = () => {
    const documentStyle = getComputedStyle(document.documentElement);
    const textColor = documentStyle.getPropertyValue('--p-text-color');
    const textColorSecondary = documentStyle.getPropertyValue('--p-text-muted-color');
    const surfaceBorder = documentStyle.getPropertyValue('--p-content-border-color');

    return {
        plugins: {
            legend: {
                labels: {
                    color: textColor
                }
            }
        },
        scales: {
            x: {
                ticks: {
                    color: textColorSecondary
                },
                grid: {
                    color: surfaceBorder
                }
            },
            y: {
                beginAtZero: true,
                ticks: {
                    color: textColorSecondary,
                    stepSize: 1,
                    precision: 0,
                },
                grid: {
                    color: surfaceBorder
                }
            }
        }
    };
}
</script>
