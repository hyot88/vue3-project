<template>
    <nav aria-label="Page navigation example">
        <ul class="pagination">
            <li v-if="currentPage !== 1" class="page-item">
                <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage - 1)">
                    Previous
                </a>
            </li>
            <li
                v-for="page in numberOfPages"
                :key="page"
                class="page-item"
                :class="currentPage === page ? 'active' : ''"
            >
                <a style="cursor: pointer" class="page-link" @click="getTodos(page)">{{ page }}</a>
            </li>
            <li v-if="numberOfPages !== currentPage" class="page-item">
                <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage + 1)">
                    Next
                </a>
            </li>
        </ul>
    </nav>
</template>

<script>
import { ref, getCurrentInstance } from 'vue';

export default {
    props: {
        pageInfo: {
            type: Object,
            required: true
        }
    },
    emits: ['get-todo'],
    setup(props) {
        const { emit } = getCurrentInstance();
        const getTodos = (page) => {
            emit('get-todo', page);
        };

        const currentPage = ref(props.pageInfo.currentPage);
        const numberOfPages = ref(props.pageInfo.numberOfPages);

        return {
            getTodos,
            currentPage,
            numberOfPages,
        }
    }
}
</script>

<style>

</style>