<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { FilterMatchMode } from 'primevue/api';

const products = ref([]);
const layout = ref('grid');
const searchTerm = ref('');
const sortOrder = ref(null);
const sortField = ref(null);
// onBeforeMount(() => {
//     initFilters();
// });

onMounted(async () => {
    await Data();
});

async function Data() {
    let users = null;
    const allP = []
    await axios.get(`https://simple-assemble-af150-default-rtdb.firebaseio.com/users/users.json`).then((data) => {
        users = data.data;
    });
    for (const key in users) {
        console.log("key", users[key].uid)
        await axios.get(`https://simple-assemble-af150-default-rtdb.firebaseio.com/products/${users[key].uid}.json`).then((data) => {
            const res = data.data
            for (const key in res) {
                allP.push(
                    {
                        id: key,
                        name: res[key].name,
                        description: res[key].description,
                        images: res[key].images,
                        videoLink: res[key].videoLink,
                        category: res[key].category
                    }
                )
            }


        });
    }

    products.value = allP
    console.log("products", products.value)


}
function filterByValue(array, string) {
    return array.filter(o =>
        Object.keys(o).some(k => o[k].includes(string.toLowerCase())));
}

const filteredProducts = computed(() => {
    if (searchTerm.value == '') {
        return products.value
    }

    else {
 
        let filter = filterByValue(products.value,searchTerm.value);        console.log(filter)

        return filter;
    }
})
</script>

<template>
    <div class="grid">
        <div class="col-12">
            <div class="">
                <h1 class="text-7xl m-2 text-center mb-5">Products</h1>
                <DataView :value="filteredProducts" :layout="layout" :paginator="true" :rows="9">
                    <template #header>
                        <div class="grid grid-nogutter">
                            <div class="col-6 text-left">
                                <span class="block mt-2 md:mt-0 p-input-icon-left">
                                    <i class="pi pi-search" />
                                    <InputText v-model="searchTerm" placeholder="Search..." />
                                </span>
                            </div>
                            <div class="col-6 text-right">
                                <DataViewLayoutOptions v-model="layout" />
                            </div>
                        </div>
                    </template>
                    <template #list="slotProps">
                        <div class="col-12">
                            <div class="flex flex-column md:flex-row align-items-center p-3 w-full">
                                <img :src="slotProps.data.images[0]" :alt="slotProps.data.name"
                                    class="my-4 md:my-0 w-9 md:w-10rem shadow-2 mr-5" style="max-height: 140px;" />
                                <div class="flex-1 text-center md:text-left">
                                    <div class="font-bold text-2xl">{{ slotProps.data.name }}</div>
                                    <div class="mb-3">{{ slotProps.data.description }}</div>

                                    <div class="flex align-items-center">
                                        <i class="pi pi-tag mr-2"></i>
                                        <span class="font-semibold">{{ slotProps.data.category.toString() }}</span>
                                    </div>
                                </div>

                            </div>
                        </div>
                    </template>

                    <template #grid="slotProps">
                        <div class="col-12 md:col-4">
                            <div class="card m-3 border-1 surface-border">

                                <div class="text-center ">
                                    <img :src="slotProps.data.images[0]" :alt="slotProps.data.name"
                                        class="w-9 shadow-2 my-3 mx-0" style="height: 240px;" />
                                    <div class="text-2xl font-bold"> {{ slotProps.data.name }}</div>
                                    <div class="mb-3"> {{ slotProps.data.description }}</div>
                                    <div class="flex align-items-center justify-content-center mb-2">
                                        <i class="pi pi-tag mr-2"></i>
                                        <span class="font-semibold">{{ slotProps.data.category.toString() }}</span>
                                    </div>
                                    <routerLink :to="'/products/' + slotProps.data.id">                                      
                                    <Button label="view"></Button></routerLink>  

                                </div>

                            </div>
                        </div>
                    </template>
                </DataView>
            </div>
        </div>


    </div>
</template>

<style scoped lang="scss">
@import '@/assets/demo/styles/badges.scss';
</style>
