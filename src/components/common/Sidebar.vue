<template>
    <div class="sidebar" style="margin-top: 60px;">
        <div class="nav-icon">
            <i class="pi pi-bars"></i>
        </div>
        <template v-if="project_key && items.length">
          <RouterLink
              v-for="item in items"
              :key="item.name"
              :to="item.to"
              class="nav-icon"
              :class="{ active: item.name.toLowerCase() === active }"
              v-tooltip.right="item.name"
              >
              <i :class="item.icon"></i>
          </RouterLink> 
        </template>
    </div>
</template>
<style src="../../assets/css/sidebar.css"></style>
<script setup>
import { computed } from 'vue';

const props = defineProps({
  active: String,
  project_key: String,
});

const items = computed(() => {
  if (!props.project_key) return [];

  return [
    { name: 'Home', icon: 'pi pi-home', to: '/workspace/project/' + props.project_key },
    { name: 'Add Task', icon: 'pi pi-plus', to: '/workspace/add/' + props.project_key },
    { name: 'Tasks', icon: 'pi pi-list', to: '/workspace/task/' + props.project_key },
    { name: 'Files', icon: 'pi pi-folder-open', to: '/workspace/file/' + props.project_key },
    { name: 'Reports', icon: 'pi pi-chart-bar', to: '/workspace/reports/' + props.project_key },
    { name: 'Settings', icon: 'pi pi-cog', to: '/workspace/settings/' + props.project_key },
  ];
});
</script>