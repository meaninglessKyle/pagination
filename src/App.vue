<template>
    <Pagination
        :total="100"
        :pageSize="4"
        :maxPageIndicatorNum="4"
        v-model:nowPage="nowPage"
    />
</template>

<script lang="ts">
import {defineComponent, ref} from "vue";
import Pagination from "./Pagination/Pagination.vue";

export default defineComponent({
    name: 'App',
    components: {Pagination},
    setup() {
        const nowPage = ref(1);
        return {
            nowPage,
        };
    },
});
</script>

<style scoped>
</style>
