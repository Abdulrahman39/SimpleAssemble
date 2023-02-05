<script setup>
import ProductService from '@/service/ProductService';
import PhotoService from '@/service/PhotoService';
import { ref, onMounted, onBeforeMount } from 'vue';
import { useLayout } from '@/layout/composables/layout';
import { useRoute } from 'vue-router';
import axios from 'axios';

const { contextPath } = useLayout();
const router = useRoute();

const products = ref({
    name:'',
    description:'',
    category: [],
    videoLink:''
});
const images = ref([]);
const id = router.params.id
const galleriaResponsiveOptions = ref([
    {
        breakpoint: '1024px',
        numVisible: 5
    },
    {
        breakpoint: '960px',
        numVisible: 4
    },
    {
        breakpoint: '768px',
        numVisible: 3
    },
    {
        breakpoint: '560px',
        numVisible: 1
    }
]);
const carouselResponsiveOptions = ref([
    {
        breakpoint: '1024px',
        numVisible: 3,
        numScroll: 3
    },
    {
        breakpoint: '768px',
        numVisible: 2,
        numScroll: 2
    },
    {
        breakpoint: '560px',
        numVisible: 1,
        numScroll: 1
    }
]);


async function Data() {
    await axios.get(`https://simple-assemble-af150-default-rtdb.firebaseio.com/AllProducts/${id}.json`).then((data) => {
        console.log(data.data)
        const allP = []
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

        products.value.name = allP[0].name;
        products.value.category = allP[0].category;
        products.value.description = allP[0].description;
        products.value.videoLink = allP[0].videoLink;

        images.value = allP[0].images

    })
}




onBeforeMount(async () => {

    await Data();
});
</script>

<template>
    <div class="grid p-fluid">
        <div class="col-12">
            <div class="card">
                <h1 class="text-center">Product Info</h1>
                
            </div>
        </div>

        <div class="col-12 ">
            <div class="card">
               
                <Galleria :value="images" :responsiveOptions="galleriaResponsiveOptions" :numVisible="7"
                    :circular="true" containerStyle=" justify-content-center margin: auto max-width: 400px">
                    <template #item="slotProps">
                        <Image :src=" slotProps.item" width="550" preview />
                    </template>
                    <template #thumbnail="slotProps" containerStyle=" justify-content-center margin: auto">
                        <div>
                            <Image :src="slotProps.item" :alt="slotProps.item.alt" style="width: fit-content ;display: block;"
                                height="50" />
                        </div>
                    </template>
                </Galleria>
            </div>
        </div>

        <div class="col-12">
            <div class="card">
                <h1>{{ products.name }} INFO: </h1>
                <div class="col ">
                    <TabView>
                    <!-- <TabPanel header="Name">
                        <p class="line-height-3 m-0">
                            {{products[0].name}}
                        </p></TabPanel
                    > -->
                    <TabPanel header="Description">
                        <p class="line-height-3 m-0">
                        {{ products.description }}   
                        </p>
                    </TabPanel>
                    <TabPanel header="Category">
                    <Chip v-for="c in products.category" :label="c" class="mr-2 mb-2"></Chip>  
                        
                    </TabPanel>
                    <TabPanel header="Vedio Link">
                        <iframe  width="560" height="315" :src="products.videoLink" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
                        <br><a class="text-primary" :href="products.videoLink">if video not working</a>
                    </TabPanel>
                    </TabView>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
@import '@/assets/demo/styles/badges.scss';
@import '@/assets/demo/styles/items.scss';
</style>
