<template>
    <ContentField>
        <table class="table table-striped table-hover" style="text-align:center;">
            <thead>
                <tr>
                    <th>玩家</th>
                    <th>天梯分</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="user in users" :key="user.id">
                    <td>
                        <img :src="user.photo" alt="" class="record-user-photo">
                        &nbsp;
                        <span class="record-user-username">{{ user.username }}</span>
                    </td>
                    <td>
                        {{ user.rating }}
                    </td>
                </tr>
            </tbody>
        </table>
        <nav aria-label="...">
            <ul class="pagination" style="float: right;">
                <li class="page-item">
                    <span class="page-link" @click="click_page(-2)">前一页</span>
                </li>
                <li :class="'page-item ' + page.is_active" v-for="page in pages" :key="page.number" @click="click_page(page.number)">
                    <a class="page-link"> {{ page.number }} </a>
                </li>
                <li class="page-item">
                    <a class="page-link" @click="click_page(-1)">后一页</a>
                </li>
            </ul>
        </nav>
    </ContentField>
</template>

<script setup>
import ContentField from '@/components/ContentField.vue';
import { getList } from '../../api/ranklist/getRankList';
import { useStore } from 'vuex';
import { onMounted, ref } from 'vue';

const store = useStore();
let current_page = 1;
let users = ref([])
let user_total = 0;
let pages = ref([]);

const click_page = page => {    
    if (page === -2) page = current_page - 1;
    else if (page === -1) page = current_page + 1;
    let max_pages = parseInt(Math.ceil(user_total / 3));

    if (page >= 1 && page <= max_pages) {
        pull_page(page);
    }
}

const update_pages = () => {
    let max_pages = parseInt(Math.ceil(user_total / 3));
    let new_pages = [];
    for (let i = current_page - 2; i <= current_page + 2; i++) {
        if (i >= 1 && i <= max_pages) {
            new_pages.push({
                number: i,
                is_active: i === current_page ? "active" : "",
            });
        }
    }
    pages.value = new_pages;
}


const pull_page = async page => {
    current_page = page;
    await getList(page).then(resp => {
        console.log(resp.data)
        users.value = resp.data.users;
        user_total = resp.data.user_count;
        update_pages();
    }).catch(resp => {
        console.log(resp.data);
    });
}

onMounted(() => {
    pull_page(current_page);
})

</script>

<style scoped>
img.record-user-photo {
    width: 5vh;
    border-radius: 50%;
}
</style>