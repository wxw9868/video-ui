<template>
    <v-card variant="flat" height="100%">
        <v-layout>
            <v-main>
                <v-tabs v-model="tab" align-tabs="center" :mandatory=true>
                    <v-tab value="one" @click="getData('va.CreatedAt','desc')">{{ $t('RecentUpdate') }}</v-tab>
                    <v-tab value="two" @click="getData('a.actress','desc')">{{ $t('Alphabetically') }}</v-tab>
                    <v-tab value="three" @click="getData('count','desc')">{{ $t('MostVideos') }}</v-tab>
                </v-tabs>
                <v-list lines="two">
                    <v-lazy :min-height="200" :options="{ 'threshold': 0.5 }" transition="fade-transition">
                        <v-data-iterator :items="items" :items-per-page="itemsPerPage" :page="page" :loading="loading">
                            <template v-slot:default="{ items }">
                                <v-container class="d-flex" fluid>
                                    <v-row justify="start" dense>
                                        <v-col v-for="(file, i) in items" :key="i" cols="6" sm="3" order="1">
                                            <v-skeleton-loader type="list-item-avatar-two-line" :loading="loading">
                                                <v-list-item 
                                                    :title="file.raw.actress"
                                                    :subtitle="file.raw.count + subtitle"  
                                                    :href="path + file.raw.id"
                                                >
                                                    <template v-slot:prepend>
                                                        <v-avatar>
                                                            <v-img :src="host + file.raw.avatar"></v-img>
                                                        </v-avatar>
                                                    </template>
                                                </v-list-item>
                                            </v-skeleton-loader>
                                        </v-col>
                                    </v-row>
                                </v-container>
                                <v-pagination v-model="page" :length="length" @click="pagination()"></v-pagination>
                            </template>
                        </v-data-iterator>
                    </v-lazy>
                </v-list>
            </v-main>
        </v-layout>
    </v-card>
</template>

<script>
import { ref, inject } from 'vue';
import { useGoTo } from 'vuetify';

export default {
    setup() {
        const host = inject('serverHost');
        const goTo = useGoTo()
        return { host, goTo }
    },
    data: () => ({
        tab: '',
        path: '/video/list?id=',
        subtitle: ' 部影片',
        itemsPerPage: 40,
        page: ref(1),
        items: [],
        loading: true,
        length: 0,
    }),
    methods: {
        pagination() {
            localStorage.setItem('actress-currentPage',this.page);
            this.goTo(0, { container: '#goto-container' });
        },
        loadPage() {
            let currentPage = parseInt(localStorage.getItem('actress-currentPage'));
            this.page = currentPage || this.page;
        },
        loadTab(tab,action,sort) {
            if (action !='' && sort != '') localStorage.setItem('actress-currentPage',1)

            this.tab = tab || localStorage.getItem('actress-tab');
            action = action || localStorage.getItem('actress-action');
            sort = sort || localStorage.getItem('actress-sort');

            localStorage.setItem('actress-tab',this.tab);
            localStorage.setItem('actress-action',action);
            localStorage.setItem('actress-sort',sort);
            return {action,sort}
        },
        getData(action, sort) {
            const obj = this.loadTab(this.tab,action,sort);

            this.$http.get('/video/getActress', { params: { action: obj.action, sort: obj.sort } })
                .then(response => {
                    // console.log(response);
                    const data = response.data.data.list;
                    this.items = data;
                    this.length = Math.ceil(data.length / this.itemsPerPage);
                    this.loading = false;
                    this.loadPage(); 
                }).catch(function (error) {
                    if (error.response) {
                        // 请求成功发出且服务器也响应了状态码，但状态代码超出了 2xx 的范围
                        console.log(error.response.data);
                        console.log(error.response.status);
                        console.log(error.response.headers);
                    } else if (error.request) {
                        // 请求已经成功发起，但没有收到响应
                        // `error.request` 在浏览器中是 XMLHttpRequest 的实例，
                        // 而在node.js中是 http.ClientRequest 的实例
                        console.log(error.request);
                    } else {
                        // 发送请求时出了点问题
                        console.log('Error', error.message);
                    }
                    console.log(error.config);
                    console.log(error);
                });
        }
    },
    mounted() {
        this.getData('', '');
    },
}
</script>